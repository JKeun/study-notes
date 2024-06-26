# Google Cloud Run 배포 과정
FastAPI 서비스를 Cloud Run 에 배포하기 위한 시행 착오들


### Dockerfile

```
# Dockerfile
FROM python:3.11-slim

# 작업 디렉토리 설정
WORKDIR /code

# 의존성 파일 복사 및 설치
COPY ./requirements.txt /code/requirements.txt
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

# 애플리케이션 코드 복사
COPY ./app /code/app

# FastAPI 서버 실행
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
```
- --no-cache-dir: 캐시 사용 없이 변경된 requiremenets.txt 를 그대로 반영


### deploy.sh

```
#!/bin/bash

service_account_id=<service_id>
project=<project_id>
region=<region>
secret_name=<secret_name>
secret_id=<secret_id>
docker_image="gcr.io/$project/$service_account_id"

# Docker build and push
docker buildx build --platform linux/amd64 --no-cache -t $docker_image .
docker push $docker_image

# Create a service account if it doesn't exist
if ! gcloud iam service-accounts list --filter="email ~ ^$service_account_id@$project.iam.gserviceaccount.com$" --format="value(email)"; then
  gcloud iam service-accounts create $service_account_id \
      --display-name="$service_account_id" \
      --project=$project
fi

gcloud projects add-iam-policy-binding $project \
  --member="serviceAccount:$service_account_id@$project.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

gcloud run deploy $service_account_id \
  --image $docker_image \
  --platform managed \
  --region $region \
  --service-account $service_account_id@$project.iam.gserviceaccount.com \
  --set-secrets=OPENAI_API_KEY=projects/$secret_id/secrets/$secret_name:latest \
  --allow-unauthenticated
```

- docker build `--platform linux/amd64`: Mac M1 Silicon 제품의 경우 해당 플래그를 꼭 추가해야함
- docker build `--no-cache`: 여러차례 동일 이름의 도커 이미지를 만들었을 경우, 캐시 사용하지 않고 수정된 도커 이미지를 빌드
- cloud `--set-secrets`: Google Secret Manager 를 사용하는 경우 해당 키값을 환경 변수로 추가


### get_openai_api_key

```
def get_openai_api_key(version: str="latest") -> str:
    # deploy env
    openai_api_key = os.getenv("OPENAI_API_KEY")
    if openai_api_key:
        return openai_api_key

    client = secretmanager.SecretManagerServiceClient()
    name = f"projects/{PROJECT_ID}/secrets/{SECRET_MANAGER}/versions/{version}"
    response = client.access_secret_version(request={"name": name})
    return response.payload.data.decode("UTF-8")
```

- deploy 환경일 경우 `OPENAI_API_KEY` 환경변수를 사용하도록 코드 수정


### 그 외 docker 관련 문법들

컨테이너 조회

- `$docker ps -a`

이미지 조회

- `$docker images`

이미지 삭제

- `$docker rmi <image-id>`
  - `-f`: 강제 삭제

이미지 빌드

- `$docker build --no-cache -t  <image-url or image-id or image-name> .`

이미지 run

- `$docker run -e OPENAI_API_KEY=$OPENAI_API_KEY -p 8080:8080 <image>`
  - `-e` 환경변수 입력: `export OPENAI_API_KEY='your_openai_api_key_here'` 선 입력 후 사용

이미지 CLI 접근

- `$docker run -it <image> bash`
  - 만약 `python:3.11-slim` 과 같은 경량화된 베이스 이미지를 사용하는 경우, `sh` 사용 가능
    - `$docker run -it <image> sh`

### Cloud Run 토막상식

Cloud Run 자동 Shutdown

- 서비스에 대한 요청이 없으면, 인스턴스는 일정 시간이 지나면 자동으로 종료됨
- 기본적으로 15분
- 최소 및 최대 인스턴스 설정으로 서비스의 자동 스케일링을 제어 할 수 있음
  - `gcloud run services update [SERVICE_NAME] --min-instances=1 --max-instances=5`

최소 인스턴스 수에 따른 자동 shutdown 동작 흐름

- 최소 인스턴스 수가 0인 경우
  - 동작: 최소 인스턴스 수가 0으로 설정되어 있으면, 서비스에 대한 요청이 없을 때 인스턴스가 자동으로 축소됨 즉, 서비스가 유휴 상태일 때는 인스턴스가 전혀 실행되지 않음
  - 자동 종료: 15분 동안 요청이 없으면 인스턴스는 자동으로 종료.
- 최소 인스턴스 수가 1인 경우
  - 동작: 최소 인스턴스 수가 1로 설정되어 있으면, 서비스가 유휴 상태일 때도 최소한 하나의 인스턴스는 항상 실행. 서비스가 요청을 더 빠르게 처리할 수 있도록 하기 위함
  - 자동 종료: 최소 인스턴스 수가 1로 설정되어 있는 경우, 인스턴스가 완전히 종료되지 않음. 유휴 상태에서도 최소 하나의 인스턴스는 항상 실행되기 때문에, 15분 동안 요청이 없더라도 서비스가 자동으로 완전히 shutdown 되지 않음.
