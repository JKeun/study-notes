### Vimrc 에 적절한 플러그인과 환경변수를 추천해줘. Vimrc 파일을 작성해줘

Vim을 보다 강력하게 만들기 위해 여러 가지 플러그인과 환경 변수를 설정할 수 있습니다. 여기서는 기본적인 플러그인과 유용한 환경 변수를 포함한 .vimrc 예제를 제공합니다. 이 예제에서는 코드 편집, 파일 탐색, Git 통합, 자동 완성, 그리고 테마를 위한 플러그인을 포함하고 있습니다.

```vim
" vim-plug 설치 (이미 설치한 경우 건너뛸 수 있음)
if empty(glob('~/.vim/autoload/plug.vim'))
  silent !curl -fLo ~/.vim/autoload/plug.vim --create-dirs
    \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  autocmd VimEnter * PlugInstall
endif

call plug#begin('~/.vim/plugged')

" 파일 탐색기 플러그인
Plug 'preservim/nerdtree'
Plug 'ryanoasis/vim-devicons' " NERDTree 아이콘 지원

" 상태 표시줄 플러그인
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'

" Git 통합 플러그인
Plug 'tpope/vim-fugitive'

" 자동 완성 플러그인
Plug 'neoclide/coc.nvim', {'branch': 'release'}

" 구문 강조 및 코드 인식 플러그인
Plug 'sheerun/vim-polyglot'

" 테마 플러그인
Plug 'morhetz/gruvbox'

" 주석 처리 플러그인
Plug 'tpope/vim-commentary'

call plug#end()

" 기본 설정
syntax on
set number " 줄 번호 표시
set relativenumber " 상대적 줄 번호 표시
set tabstop=4 " 탭 간격 설정
set shiftwidth=4 " 들여쓰기 간격 설정
set expandtab " 탭을 공백으로 변환
set smartindent " 스마트 인덴트 사용
set autoindent " 자동 인덴트 사용
set cursorline " 커서 라인 강조
set background=dark " 배경 색상 설정
colorscheme gruvbox " 테마 설정

" NERDTree 설정
autocmd vimenter * NERDTree
autocmd bufenter * if (winnr('$') == 1 && exists('b:NERDTree') && b:NERDTree.isTabTree()) | q | endif
nnoremap <C-n> :NERDTreeToggle<CR>

" vim-airline 설정
let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#tabline#buffer_nr_show = 1

" coc.nvim 설정
" 기본적인 설정 파일 생성
if !filereadable(expand('~/.vim/coc-settings.json'))
  call writefile(['{}'], expand('~/.vim/coc-settings.json'))
endif
" coc.nvim을 위한 기본 매핑
inoremap <silent><expr> <Tab> pumvisible() ? "\<C-n>" : "\<Tab>"
inoremap <silent><expr> <S-Tab> pumvisible() ? "\<C-p>" : "\<S-Tab>"
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" vim-commentary 설정
" 편리한 주석 토글 매핑
nmap <leader>c :Commentary<CR>
vmap <leader>c :Commentary<CR>

" 기타 유용한 매핑
nnoremap <C-p> :Files<CR> " FZF 파일 찾기
nnoremap <C-b> :Buffers<CR> " FZF 버퍼 목록
```


### 실제 .vimrc 설정 내용
```vim
set nocompatible              " Vi와 호환 불가 설정, 필수
filetype off                  " 필수

" Vundle을 포함시키기 위해 runtime 경로를 설정하고 초기화
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" Vundle이 스스로를 관리하도록 설정, 필수
Plugin 'VundleVim/Vundle.vim'

" 파일 탐색기 플러그인
Plugin 'preservim/nerdtree'
Plugin 'ryanoasis/vim-devicons' " NERDTree 아이콘 지원

" 상태 표시줄 플러그인
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'

" Git 통합 플러그인
Plugin 'tpope/vim-fugitive'

" 구문 강조 및 코드 인식 플러그인
Plugin 'sheerun/vim-polyglot'

" 테마 플러그인
Plugin 'morhetz/gruvbox'

" 주석 처리 플러그인
Plugin 'tpope/vim-commentary'


" 당신의 모든 플러그인은 다음 명령어 이전에 추가되어야 합니다
call vundle#end()            " 필수
filetype plugin indent on    " 필수
" 플러그인의 들여쓰기 변화를 무시하려면, 대신 이 명령어를 사용하십시오:
"filetype plugin on
"
" 간단한 도움말
" :PluginList       - 설정된 플러그인의 리스트
" :PluginInstall    - 플러그인 설치; 업데이트를 하려면 `!`를 덧붙이거나 :PluginUpdate 명령을 사용하십시오
" :PluginSearch foo - foo에 대해 검색; `!`를 덧붙여 로컬 캐시를 새로고침하십시오
" :PluginClean      - 사용하지 않는 플러그인의 삭제를 확인; `!`를 붙여 자동 삭제를 승인하십시오
"
" 더 자세한 내용은 :h vundle 문서나 wiki의 FAQ를 확인하십시오
" 다음 줄부터 플러그인이 아닌 내용을 넣으십시오



" 기본 설정
syntax on
set number " 줄 번호 표시
set relativenumber " 상대적 줄 번호 표시
set tabstop=4 " 탭 간격 설정
set shiftwidth=4 " 들여쓰기 간격 설정
set expandtab " 탭을 공백으로 변환
set smartindent " 스마트 인덴트 사용
set autoindent " 자동 인덴트 사용
set cursorline " 커서 라인 강조
set background=dark " 배경 색상 설정
colorscheme gruvbox " 테마 설정

" NERDTree 설정
" Start NERDTree and put the cursor back in the other window.
" autocmd VimEnter * NERDTree | wincmd p
" Start NERDTree when Vim is started without file arguments.
" autocmd StdinReadPre * let s:std_in=1
" autocmd VimEnter * if argc() == 0 && !exists('s:std_in') | NERDTree | endif
" Start NERDTree. If a file is specified, move the cursor to its window.
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * NERDTree | if argc() > 0 || exists("s:std_in") | wincmd p | endif
nnoremap <C-n> :NERDTreeToggle<CR>

" vim-airline 설정
let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#tabline#buffer_nr_show = 1

" vim-commentary 설정
" 편리한 주석 토글 매핑
nmap <leader>c :Commentary<CR>
vmap <leader>c :Commentary<CR>

" 기타 유용한 매핑
nnoremap <C-p> :Files<CR> " FZF 파일 찾기
nnoremap <C-b> :Buffers<CR> " FZF 버퍼 목록
```
