---
title: Yet Another Haskell and (Neo)Vim Guide
layout: post
date: 2019-12-16
categories: setup
tags: haskell vim memory leak neovim nvim computer science setup
---

As the first semester of my second year as a Comp Sci undergrad comes to a close, I thought I'd share how I (stubbornly) continue to use Vim for everything, including Haskell development.
When tackling my assignments, I wanted a toolchain which was feature-full and fast.
There are plenty of posts out there on just this already, like [this one](http://marco-lopes.com/articles/Vim-and-Haskell-in-2019/), [this one](https://blog.jez.io/haskell-development-with-neovim/) [and this one](https://monicalent.com/blog/2017/11/19/haskell-in-vim/).

Pointing out here that I've browsed pretty much all of them while creating my setup, I'd like to throw mine into the mix here as well because:
<!-- more -->

- I've used a lot of the completion and syntax tools mentioned in those posts and found better stuff,
- I have a little bit more to add (see memory leak issues addressed further down).

## "Get to the point mate"

**NOTE**: Please install any Vim plugins with your favourite plugin manager.
I use [vim-plug](https://github.com/junegunn/vim-plug).

Click the root-level bullets to go view what I have to say about each, along with the config I use to set up the tools.

- [Syntax Highlighting](#syntax-highlighting)
    - [haskell-vim](https://github.com/neovimhaskell/haskell-vim)
- [Formatting](#formatting)
    - [brittany](https://hackage.haskell.org/package/brittany)
    - [neoformat](https://github.com/sbdchd/neoformat)
- [Auto-Completion](#auto-completion)
    - [coc.vim](https://github.com/neoclide/coc.nvim)
    - [haskell-ide-engine (HIE)](https://github.com/haskell/haskell-ide-engine)
- [Snippets](#snippets)
    - Run `:CocInstall coc-snippets`
    - [UltiSnips](https://github.com/SirVer/ultisnips)
    - [vim-snippets (provides some default snippets)](https://github.com/honza/vim-snippets)



## General Setup with Neovim

I won't go into detail here - some guides explain their whole Vim setup from scratch, but I want to stick mainly to the Haskell-related tools I use to make life easier.
If you'd like to see **my vimrc/init.vim**, look [here](https://gitlab.com/JontySR/dots/blob/master/nvim/.config/nvim/init.vim).
One thing to mention though is that I use Neovim now, having used Vim for the longest time.
This is because Neovim is pretty much always ahead of Vim in terms of innovation and speed (and clean codebase), and Vim often follows suit later.

When it comes to creating new projects, I'll be making a post soon on scripts I'm making to streamline setting up new projects across different languages, so stay tuned 😉.

## Syntax Highlighting

For Haskell **syntax highlighting**, Vim's defaults are a little lacklustre, and we can do better.
[haskell-vim](https://github.com/neovimhaskell/haskell-vim) is a plugin which provides much more reliable contextual highlighting, with support for more keywords and new features.
It requires minimal config - see the link for configuration options.

## Formatting

My *favourite* part of this whole setup is the **formatting** methods I use.
Haskell is already beautiful, but I don't trust myself to get all the spacing and indentation correct or consistent.
To remedy this, I employ the services of [brittany](https://hackage.haskell.org/package/brittany) and [neoformat](https://github.com/sbdchd/neoformat).
brittany is the smartest Haskell formatter out there, and I'm still in awe of it.

After installing brittany and neoformat, you'll need to enable brittany and, if you want to, enable format-on-save for ease of use:

<details>
<summary style="font-weight:700">Click to expand config</summary>

```vim
let g:neoformat_enabled_haskell = ['brittany']

augroup fmt
  autocmd!
  autocmd BufWritePre * undojoin | Neoformat
augroup END
```

</details>
<br />

## Auto-Completion

For auto-completion, I recently stumbled across [coc.vim](https://github.com/neoclide/coc.nvim), an Intellisense language server protocol engine for Vim.
I've previously used deoplete, neocomplete and YouCompleteMe (YCM), settling on YouCompleteMe for the most reliable and extensive completion.
However, I found YCM to be really heavy and it took quite a long time to compile on my machine.

Coc is modular (without re-compilation), fast and easy to set up in comparison.
Oh, and it does **asynchronous linting** too (if the language server you use supports it), so scrap ALE, Syntastic and others!
The overarching configuration I used for Coc adheres fairly strongly to the recommended config in the [readme (coc.vim)](https://github.com/neoclide/coc.nvim):

<details>
<summary style="font-weight:700">Click to expand config</summary>

```vim
" coc config
" Fill the global extensions list after installing
" extensions (with :CocInstall)
let g:coc_global_extensions = [
  \ 'coc-snippets', 
  \ 'coc-pairs',
  \ 'coc-json', 
  \ 'coc-ccls', 
  \ ]
" from readme
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
let g:UltiSnipsExpandTrigger = "<nop>"
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
" Or use `complete_info` if your vim support it, like:
" inoremap <expr> <cr> complete_info()["selected"] != "-1" ? "\<C-y>" : "\<C-g>u\<CR>"

" Use `[g` and `]g` to navigate diagnostics
nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)

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
nmap <F2> <Plug>(coc-rename)

" Remap for format selected region
xmap <leader>f  <Plug>(coc-format-selected)
nmap <leader>f  <Plug>(coc-format-selected)

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

" Create mappings for function text object, requires document symbols feature of languageserver.
xmap if <Plug>(coc-funcobj-i)
xmap af <Plug>(coc-funcobj-a)
omap if <Plug>(coc-funcobj-i)
omap af <Plug>(coc-funcobj-a)

" Use <C-d> for select selections ranges, needs server support, like: coc-tsserver, coc-python
"nmap <silent> <C-d> <Plug>(coc-range-select)
"xmap <silent> <C-d> <Plug>(coc-range-select)

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

</details>
<br />

The language server I used to interface with Coc is:

### haskell-ide-engine

Here's the only clunky part of the setup, and I wish it wasn't - our language server.
You'll want to install the [haskell-ide-engine (HIE)](https://github.com/haskell/haskell-ide-engine) on your system to proceed.
It's super powerful, and in my experience easier than ghcmod and ghcide to set up.

Here's a brief history of why it's a bit borked:

- [There was a **memory leak** HIE hadn't mitigated, but this guy had a workaround (thanks!)](https://dimjasevic.net/marko/2018/08/15/haskell-ide-the-memory-hog-engine/);
- [It got fixed](https://github.com/haskell/haskell-ide-engine/pull/1292);
- [Another big leak](https://github.com/haskell/haskell-ide-engine/issues/1318).

Thanks for the workaround [Marko](https://dimjasevic.net/marko/) (type `:CocConfig` in Vim to link HIE, adding to the `languageserver` key if you have other servers set up):

<details>
<summary style="font-weight:700">Click to expand config</summary>

```json
  "languageserver": {
    "haskell": {
      "command": "hie",
      "args": ["+RTS", "-c", "-M1500M", "-K1G", "-A16M", "-RTS", "--lsp"],
      "rootPatterns": [
        "stack.yaml",
        "cabal.config",
        "package.yaml"
      ],
      "filetypes": [
        "hs",
        "lhs",
        "haskell"
      ],
      "initializationOptions": {
        "languageServerHaskell": {
          "hlintOn": true
        }
      }
    }
  }
```

</details>
<br />

Sadly hie will kill itself in the background (only noticeable effect is that it temporarily "stops working" for ~20 secs) when it hits around 2GiB of memory, but it'll restart without lagging anything else.


## Snippets

Coc also provides powerful **snippet** functionality in its auto-complete drop-downs when used in conjunction with [UltiSnips](https://github.com/SirVer/ultisnips) and [vim-snippets (provides some default snippets)](https://github.com/honza/vim-snippets)!

After installing those two Vim plugins using the plugin manager of your choice, simply run `:CocInstall coc-snippets` to install the functionality within coc, and add `coc-snippets` to `g:coc_global_extensions` (see the top of the previous config block 👆). 

Congratulations!
From now on you can create snippets with `:UltiSnipsEdit` (in the filetype you'd like the snippet to exist in), and see your own snippets in auto-complete!
