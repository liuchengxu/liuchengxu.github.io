---
layout: post
title:  "Vim Port of spacemacs-theme"
crawlertitle: "vim port of spacemacs-theme that supports both dark and light background"
summary: "A vim colorscheme that supports dark and light background inspired by spacemacs-theme"
categories: posts
tags: ['vim']
author: Liu-Cheng Xu
---

It has been two years since I started the project [space-vim](https://github.com/liuchengxu/space-vim). Ever since I tried spacemacs, I had became a big fan of its default theme: [spacemacs-theme](https://github.com/nashamri/spacemacs-theme), so I ported the dark version of it shortly, that is [space-vim-dark](https://github.com/liuchengxu/space-vim-dark), which is a dark only Vim colorscheme and has won some solid users in the past two years from my point of view.

![screenshot](https://raw.githubusercontent.com/liuchengxu/img/master/space-vim/space-vim-gui.png)

But I still prefer the light background when using vim in daylight, for in which case I can't see it very well on the dark background due the light reflection at times. Then I build a new one [space-vim-theme](https://github.com/liuchengxu/space-vim-theme) on the strength of [vim-colortemplate](https://github.com/lifepillar/vim-colortemplate) with the light background support added.

Finally we have the light version of spacemacs-theme :(.

dark                                                                                  | light
:----:                                                                                | :----:
![](https://raw.githubusercontent.com/liuchengxu/img/master/space-vim-theme/dark.png) | ![](https://raw.githubusercontent.com/liuchengxu/img/master/space-vim-theme/light.png)

The new colorscheme repo is [liuchengxu/space-vim-theme](https://github.com/liuchengxu/space-vim-theme). Install it using your preferred plugin manager, e.g., [vim-plug](https://github.com/junegunn/vim-plug):

```vim

Plug 'liuchengxu/space-vim-theme'

```

Try this new colorscheme by putting this line in your `.vimrc`:

```vim

color space_vim_theme

```

[space-vim-theme](https://github.com/liuchengxu/space-vim-theme) is usable at the present, but has not well polished yet, feel free to [tell me if you find something is not good!](https://github.com/liuchengxu/space-vim-theme/issues/new).
