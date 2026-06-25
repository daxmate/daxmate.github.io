---
hidden: true
layout: single
title: "VIM设置CPP开发环境"
date: 2022-03-21
author: 'dax'
categories: "IT"
---
# Vim设置CPP开发环境

本来我是想用vscode的，但是那个里面用vim的一些设定不正常，下定决心还是以后就只用Vim来写代码。

- 首先，要设置一下在Vim里面编译和运行代码，将如下代码加入到`.vimrc`中。
```vim
" 设置<leader>b 编译程序
au FileType cpp map <buffer> <leader>b :w<CR>:exec '!g++ -std=c++20' shellescape(expand("%"), 1) '-o' shellescape(expand("%:r"), 1)<CR>
" <leader>r运行
au FileType cpp map <buffer> <leader>r :!./%:r<cr>
" 自动显示行号
au FileType cpp :set nu
```

- 然后再设置一下CPP的模板。将如下代码加入到`.vimrc`中。当然需要新建一个模板文件中相应的位置。
```vim
" 新建cpp文件的时候自动从~/.vim/templates/skeleton.cpp中读取模板
autocmd BufNewFile *.cpp 0r ~/.vim/templates/skeleton.cpp
```

- 安装coc插件，用`plug`的话会很简单。用来实现代码自动补足。
	- 安装完之后，运行`:CoCConfig`, 这里我们使用`ccls`语言服务器

		```json
		{"languageserver": {
		  "ccls": {
		    "command": "ccls",
		    "filetypes": ["c", "cc", "cpp", "c++", "objc", "objcpp"],
		    "rootPatterns": [".ccls", "compile_commands.json", ".git/", ".hg/"],
		    "initializationOptions": {
		        "cache": {
		          "directory": "/tmp/ccls"
		        }
		      }
		  }
		}}
		```
	- 下面自然是要安装ccls语言服务器
		`brew install ccls`
		这个会安装`llvm`，很大，时间也很长
	- 在项目的文件夹里面新建一个`.ccls`文件，写入：
		```bash
		clang++
		%h %cpp -std=c++17
		```
	- 下面需要把coc的一些设置写到`.vimrc`文件中, 注意这里把<leader>f的设置给注释掉了，coc的代码格式化我不太喜欢，又不会自己找到设置来修改，所以这个就不用了，后面用neoformat插件来实现。
		```vim
		"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
		" settings of coc
		" if hidden is not set, TextEdit might fail.
		set hidden
		
		" Some servers have issues with backup files, see #649
		set nobackup
		set nowritebackup
		
		" Better display for messages
		set cmdheight=2
		
		" You will have bad experience for diagnostic messages when it's default 4000.
		set updatetime=300
		
		" don't give |ins-completion-menu| messages.
		set shortmess+=c
		
		" always show signcolumns
		set signcolumn=yes
		
		" Use tab for trigger completion with characters ahead and navigate.
		" Use command ':verbose imap <tab>' to make sure tab is not mapped by other plugin.
		inoremap <silent><expr> <TAB>
		      \ pumvisible() ? "\<C-n>" :
		      \ <SID>check_back_space() ? "\<TAB>" :
		      \ coc#refresh()
		inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"
		
		function! s:check_back_space() abort
		  let col = col('.') - 1
		  return !col || getline('.')[col - 1]  =~# '\s'
		endfunction
		
		" Use <c-space> to trigger completion.
		inoremap <silent><expr> <c-space> coc#refresh()
		
		" Use <cr> to confirm completion, `<C-g>u` means break undo chain at current position.
		" Coc only does snippet and additional edit on confirm.
		inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"
		
		" Use `[c` and `]c` to navigate diagnostics
		nmap <silent> [c <Plug>(coc-diagnostic-prev)
		nmap <silent> ]c <Plug>(coc-diagnostic-next)
		
		" Remap keys for gotos
		nmap <silent> gd <Plug>(coc-definition)
		nmap <silent> gy <Plug>(coc-type-definition)
		nmap <silent> gi <Plug>(coc-implementation)
		nmap <silent> gr <Plug>(coc-references)
		
		" Use K to show documentation in preview window
		nnoremap <silent> K :call <SID>show_documentation()<CR>
		
		function! s:show_documentation()
		  if (index(['vim','help'], &filetype) >= 0)
		    execute 'h '.expand('<cword>')
		  else
		    call CocAction('doHover')
		  endif
		endfunction
		
		" Highlight symbol under cursor on CursorHold
		autocmd CursorHold * silent call CocActionAsync('highlight')
		
		" Remap for rename current word
		nmap <leader>rn <Plug>(coc-rename)
		
		" Remap for format selected region
		" xmap <leader>f  <Plug>(coc-format-selected)
		" nmap <leader>f  <Plug>(coc-format-selected)
		
		augroup mygroup
		  autocmd!
		  " Setup formatexpr specified filetype(s).
		  autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
		  " Update signature help on jump placeholder
		  autocmd User CocJumpPlaceholder call CocActionAsync('showSignatureHelp')
		augroup end
		
		" Remap for do codeAction of selected region, ex: `<leader>aap` for current paragraph
		xmap <leader>a  <Plug>(coc-codeaction-selected)
		nmap <leader>a  <Plug>(coc-codeaction-selected)
		
		" Remap for do codeAction of current line
		nmap <leader>ac  <Plug>(coc-codeaction)
		" Fix autofix problem of current line
		nmap <leader>qf  <Plug>(coc-fix-current)
		
		" Use <tab> for select selections ranges, needs server support, like: coc-tsserver, coc-python
		nmap <silent> <TAB> <Plug>(coc-range-select)
		xmap <silent> <TAB> <Plug>(coc-range-select)
		xmap <silent> <S-TAB> <Plug>(coc-range-select-backword)
		
		" Use `:Format` to format current buffer
		command! -nargs=0 Format :call CocAction('format')
		
		" Use `:Fold` to fold current buffer
		command! -nargs=? Fold :call     CocAction('fold', <f-args>)
		
		" use `:OR` for organize import of current buffer
		command! -nargs=0 OR   :call     CocAction('runCommand', 'editor.action.organizeImport')
		
		" Add status line support, for integration with other plugin, checkout `:h coc-status`
		set statusline^=%{coc#status()}%{get(b:,'coc_current_function','')}
		
		" Using CocList
		" Show all diagnostics
		nnoremap <silent> <space>a  :<C-u>CocList diagnostics<cr>
		" Manage extensions
		nnoremap <silent> <space>e  :<C-u>CocList extensions<cr>
		" Show commands
		nnoremap <silent> <space>c  :<C-u>CocList commands<cr>
		" Find symbol of current document
		nnoremap <silent> <space>o  :<C-u>CocList outline<cr>
		" Search workspace symbols
		nnoremap <silent> <space>s  :<C-u>CocList -I symbols<cr>
		" Do default action for next item.
		nnoremap <silent> <space>j  :<C-u>CocNext<CR>
		" Do default action for previous item.
		nnoremap <silent> <space>k  :<C-u>CocPrev<CR>
		" Resume latest coc list
		nnoremap <silent> <space>p  :<C-u>CocListResume<CR>
		```
- 安装`neoformat`插件
	- 安装完成后，在~/.vim/plugged/neoformat/autoload/neoformat/formatters/里找到cpp.vim, 可以看到CPP的astyle是调用的c的格式函数来实现的。修改如下：
		```vim
		" function! neoformat#formatters#cpp#astyle() abort
		"     return neoformat#formatters#c#astyle()
		" endfunction


		function! neoformat#formatters#cpp#astyle() abort
			return {
						\ 'exe': 'astyle',
						\ 'args': ['--style='ansi', '--indent=spaces=4', '--pad-oper', '--pad-header'],
						\ 'stdin': 1,
				}
		endfunction
		```
		将原来的函数备注掉，然后设置一下需要的格式参数，具体的astyle的参数列表可以到网上找到参考。
	- 当然需要安装一下astyle，用brew安装即可。
	- 在vimrc中加入键映射。
		`autocmd FileType cpp noremap <buffer> <leader>f :Neoformat astyle<CR>`



这样一来就大功告成啦!!!
		
