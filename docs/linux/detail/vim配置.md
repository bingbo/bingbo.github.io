
## 下面是linux下vim的相关配置
> 直接cp以下代码到.vimrc文件即可

```bash
"""""""""""""""""""""""""""""""""
" => General Options "{{{ 
set guifont=DejaVu\ Sans\ YuanTi\ Mono\ 10

" Session 
set sessionoptions-=curdir
set sessionoptions+=sesdir
" ¿¿¿¿¿¿¿
set nocompatible
"autoclose html/xml tag
autocmd BufNewFile,BufRead *.html,*.htm,*.xml,*.tpl inoremap </ </<c-x><c-o>

set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936

set termencoding=utf-8

set encoding=utf-8

set fileencoding=utf-8
" ¿¿¿¿¿¿¿
set hls
set incsearch   
"set ic smartcase 

" ¿¿¿¿¿¿¿¿¿
set showcmd

" ¿¿¿¿¿¿
set history=200

" ¿¿¿¿¿¿¿¿¿¿¿¿¿¿¿
set autoread 

" ¿¿¿¿¿¿
set mouse=a

" ¿¿mapleader
let mapleader = ";"
let g:mapleader = ";"

" ¿¿¿¿¿¿¿¿
map <leader>s :source ~/.vimrc<cr> 
map <leader>e :e! ~/.vimrc<cr>

" ¿.vimrc¿¿¿¿¿¿¿¿
autocmd! bufwritepost vimrc source ~/.vimrc 

" ¿¿¿¿¿¿¿¿¿¿¿¿¿
autocmd BufReadPost *
			\ if line("'\"") > 0 && line ("'\"") <= line("$") |
			\ exe "normal! g'\"" |
			\ endif 


"}}} 
" => Colors and Fonts "{{{ 
set t_Co=256
" ¿¿vim¿¿¿¿¿
"set background=dark
"colorscheme wombat
colorscheme desert

" ¿¿¿¿
syntax on

" ¿¿
"set gfn=Vera\ Sans\ YuanTi\ Mono:h10
"set gfn=Droid\ Sans\ Fallback:h10

" GUI
if has("gui_running")
	set guioptions-=T
	let psc_style='cool'
endif 

" ¿¿¿¿
set foldmethod=marker
"}}}
" => other UI options"{{{
" Tab¿¿
set smarttab
set tabstop=4 
set expandtab 

" ¿¿¿¿ 
set smartindent 
set shiftwidth=4
set autoindent 
set cindent 

" ¿¿¿¿
set number 

" ¿¿¿¿¿¿ 
set ruler 

" wild¿¿ 
set wildmenu 

" ¿¿¿¿¿¿¿3¿
set so=3

" set backspace
set backspace=eol,start,indent

" Backspace and cursor keys wrap to
set whichwrap+=<,>,h,l

" set magic on 
set magic 

" No sound on errors
set noerrorbells
set novisualbell
set t_vb=

" ¿¿¿¿
set showmatch 

" How many tenths of a second to blink
set mat=2

" ¿¿¿
set laststatus=2
function! CurDir()
	let curdir = substitute(getcwd(), '/home/peter', "~/", "g")
	return curdir
endfunction
set statusline=\ %f%m%r%h\ %w\ %<CWD:\ %{CurDir()}\ %=Pos:\ %l/%L:%c\ %p%%\ 


" Smart way to move btw. windows
map <C-j> <C-W>j
map <C-k> <C-W>k
map <C-h> <C-W>h
map <C-l> <C-W>l

" ¿¿¿¿¿¿
set cmdheight=1
"}}}
" => Files "{{{
" ¿¿¿¿ 
set fileencodings=ucs-bom,utf-8,gb2312,gbk,gb18030,big-5,ucs,latin1

" ¿¿¿¿¿¿¿¿
filetype on 
filetype plugin on 
filetype indent on 

" ¿¿¿¿
set ffs=unix,dos
nmap <leader>fd :se ff=dos<cr>
nmap <leader>fu :se ff=unix<cr>

" ¿¿¿¿¿
set nobackup
set nowritebackup
```
