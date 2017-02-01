---
bg: "tools.jpg"
layout: post
title:  "Use Vim as a Python IDE"
crawlertitle: "how to use vim as a python ide"
summary: "Build a vim python IDE"
categories: posts
tags: ['tools']
author: Liu-Cheng Xu
---

I love vim and often use it to write Python. Here are some useful plugins and tools for building a delightful vim python environment, escpecially for Vim8:

![]({{ site.images }}/posts/vim-python-ide-screenshot.png)

As you can see, tmux is also one of my favourite tools in terminal.

### Syntax Checking

If you use Vim8, [w0rp/ale](https://github.com/w0rp/ale) is a better option than syntastic, for it utilizes the async feature in Vim8, you will never get stuck due to the syntax checking. It’s similar to flycheck in emacs, which allows you to lint while you type.

![](https://github.com/w0rp/ale/blob/master/img/example.gif?raw=true)
(taken from ale)

### Code Formatter

[google/yapf](https://github.com/google/yapf) can be used to format python code. Make a key mapping as bellow, then you can format your python code via `<LocalLeader> =`.

```vim
autocmd FileType python nnoremap <LocalLeader>= :0,$!yapf<CR>
```

You can also take a look at [Chiel92/vim-autoformat](https://github.com/Chiel92/vim-autoformat).

### Sort import

[timothycrosley/isort](https://github.com/timothycrosley/isort) helps you sort imports alphabetically, and automatically separated into sections.  For example, use `<LocalLeader>i` to run isort on your current python file:

```vim
autocmd FileType python nnoremap <LocalLeader>i :!isort %<CR><CR>
```

Or you can use its vim plugins: [fisadev/vim-isort](https://github.com/fisadev/vim-isort#installation).

### Auto Completion

[Valloric/YouCompleteMe](https://github.com/Valloric/YouCompleteMe) is a good way to provide code auto completion. If you think YCM is too huge to give a try, [jedi-vim](https://github.com/davidhalter/jedi-vim) is an alternative. They all use [jedi](https://github.com/davidhalter/jedi) as their backend.

![](https://github.com/davidhalter/jedi/raw/master/docs/_screenshots/screenshot_complete.png)
(taken from jedi-vim)

If use neovim, you can also try [Shougo/deoplete.nvim](https://github.com/Shougo/deoplete.nvim).

### Quick Run

If use Vim8, you can execute python file asynchronously by [skywind3000/asyncrun.vim](https://github.com/skywind3000/asyncrun.vim) and output automatically the result to the quickfix window like this:

```vim
“ Quick run via <F5>
nnoremap <F5> :call <SID>compile_and_run()<CR>

augroup SPACEVIM_ASYNCRUN
    autocmd!
    " Automatically open the quickfix window
    autocmd User AsyncRunStart call asyncrun#quickfix_toggle(15, 1)
augroup END

function! s:compile_and_run()
    exec 'w'
    if &filetype == 'c'
        exec "AsyncRun! gcc % -o %<; time ./%<"
    elseif &filetype == 'cpp'
       exec "AsyncRun! g++ -std=c++11 % -o %<; time ./%<"
    elseif &filetype == 'java'
       exec "AsyncRun! javac %; time java %<"
    elseif &filetype == 'sh'
       exec "AsyncRun! time bash %"
    elseif &filetype == 'python'
       exec "AsyncRun! time python %"
    endif
endfunction
```

For neovim, [neomake/neomake](https://github.com/neomake/neomake) is worthy of trying.

Another approach is to use **[TMUX](https://github.com/tmux/tmux)**. The idea is simple: it can split your terminal screen into two. Basically, you will have one side of your terminal using Vim and the other side will be where you run your scripts.

![]({{ site.images }}/posts/vim-python-ide-tmux.png)

### Enhance the default python syntax highlighting

[python-mode/python-mode](https://github.com/python-mode/python-mode) provides a more precise python syntax highlighting than the defaults. For example, you can add a highlighting for `pythonSelf` .

```vim
hi pythonSelf  ctermfg=68  guifg=#5f87d7 cterm=bold gui=bold
```

![](https://github.com/liuchengxu/space-vim-dark/blob/screenshots/screenshot2.png?raw=true)

For more customized python syntax highlightings, please see [space-vim: python Layer](https://github.com/liuchengxu/space-vim/blob/master/layers/%2Blang/python/config.vim#L52-L72) and *syntax/python.vim* in [python-mode/python-mode](https://github.com/python-mode/python-mode/blob/develop/syntax/python.vim) .

Actually, python-mode contains tons of stuff to develop python applications in Vim, e.g., static analysis, completion, documentation, and more. (But personally, I prefer to obtain the functionalities by some other better plugins.)

There are also some neccessary general programming plugins, e.g., [scrooloose/nerdcommenter](https://github.com/scrooloose/nerdcommenter) for convenient commenter, [Yggdroot/indentLine](https://github.com/Yggdroot/indentLine) or [nathanaelkane/vim-indent-guides](https://github.com/nathanaelkane/vim-indent-guides) for visually displaying indent levels in Vim, etc.

ALthough vim is great and many plugins are productive, IDE is still my first choice when it comes to refactoring code :).

For more details, please see [space-vim](https://github.com/liuchengxu/space-vim), enable ycmd, syntax-checking, python, programming Layer , then you could get a nice vim environment for python like the screenshot. Hope it helpful.
