---
bg: "tools.jpg"
layout: post
title:  "Tree-Sitter Highlighting in Vim"
crawlertitle: "Tree-Sitter highlight for both Vim and NeoVim"
summary: "tree-sitter highlight for both Vim and NeoVim"
categories: posts
tags: ['vim']
author: Liu-Cheng Xu
---

Vim-Clap has now integrated the tree-sitter highlighting for both Vim and Neovim. Install the latest [vim-clap](https://github.com/liuchengxu/vim-clap), [enable the experimental plugin](https://liuchengxu.github.io/vim-clap/plugins/intro.html) and invoke `:ClapAction syntax/tree-sitter-highlight` to try it out. Feedback is highly appreciated.

-----

I encountered some performance issues with Vim highlighting in the past. With some available bandwidth for side projects recently, I decided to dig into this problem and have done some experiments with Vim highlighting alternatives in vim-clap. If you haven’t heard of vim-clap, it initially started as a Vim plugin focusing on fuzzy finder using the recent popup feature, but later introduced the Rust dependency for performance enhancement. Now, it goes beyond a fuzzy finder, offering features like linting and syntax highlighting through its plugin system. A key principle in vim-clap is offloading heavy work to the Rust backend while keeping Vim a lightweight UI/UX layer. Some people may criticize the downsides with the external dependency, but for me, you can implement various features using beloved Rust for Vim instead of writing more and more VimScripts!

Returning to the topic of syntax highlighting, the results of my experiments with the tree-sitter highlight engine are promising from my perspective. I find it almost usable, and I already made it my daily driver.

### Sublime Syntax

Before delving into tree-sitter, I experimented with another highlight engine [syntect](https://github.com/trishume/syntect), a syntax highlighting library for Rust that uses [Sublime Text syntax definitions](http://www.sublimetext.com/docs/3/syntax.html#include-syntax), which has been widely used in many open-source projects ([bat](https://github.com/sharkdp/bat), [delta](https://github.com/dandavison/delta), etc). Thanks to using Rust, the integration was smooth. What I need to do is to pass the lines that should be highlighted into syntax engine and convert the highlights into a form that can be easily consumed by Vim/Neovim’s highlight API, specifically, Vim uses `prop_add()` and Neovim uses `nvim_buf_add_highlight()`. 

![Sublime Syntax Highlighting - Preview](https://github.com/liuchengxu/vim-clap/assets/8850248/eefce803-7cdd-4ddd-b0e5-3d65c1600080)

I implemented the sublime syntax highlighting for the provider preview, which only parses and renders the lines in the preview window, rather than the entire buffer. For the regular buffer highlighting that requires parsing the entire buffer, additional work is needed (e.g., [this TODO](https://github.com/liuchengxu/vim-clap/blob/9fa56f9177/crates/maple_core/src/stdio_server/plugin/syntax.rs#L119)) to make it usable.

![Sublime Syntax Highlighting](https://github.com/liuchengxu/vim-clap/assets/8850248/fa042154-45bc-48b6-b695-58ced65b0594)

Based on my limited experience, the Sublime syntax highlighting lacks refinement; compared to the native syntax highlighting in Vim, there are many missing highlights. It may be good enough for CLI applications, where you quickly glance at files and exit, but inadequate for prolonged use in an editor. When it comes to implementing a new highlight mechanism, we do expect more precise and granular highlighting, at least comparable to Vim highlighting, in many aspects. The ecosystem of Sublime syntax may not be thriving, possibly due to the decline of Sublime Text. In my opinion, this alreadys renders sublime syntax highlighting hardly an option for syntax highlighting in Vim editor.

### Tree-Sitter

Tree-sitter is a trending tool for syntax highlighting, and Neovim already has built-in support for it.  There is a heated discussion of this topic in [this GitHub issue](https://github.com/vim/vim/issues/9087), you can find a lot of fruitful information there. I have always wanted to try tree-sitter, plus there was [a feature request](https://github.com/liuchengxu/vim-clap/issues/532) in vim-clap for three years. Therefore, my primary focus of this experiment is on tree-sitter, and I’ll share my progress in vim-clap.

The general idea is to rerun the tree-sitter parser whenever the buffer changes. If the buffer has been written to the disk (by listening to the event `BufWritePost` ), load the file directly. Otherwise, use the Vim API `getbufline()` to fetch all the lines in the buffer in realtime. Rendering is not updated on every text change which is too frequent, but performed on `CursorMoved`.

Although the parsing happens on the entire buffer level, we only render the highlights for visual lines, not the whole buffer. This means we only do dozens of lines of highlighting each time. While I didn’t measure it precisely, this lazy rendering is supposedly super fast, and I haven’t run into any performance issues. Furthermore, we calculate the highlight difference between the existing ones and the latest version, updating only the changed highlights, which is efficient and reduces a lot of Vim API calls. 

The implementation is not optimal, and there is room for improvement, but it’s sufficient for now from my side.

![Vim Highlighting and Tree-Sitter](https://github.com/liuchengxu/vim-clap/assets/8850248/96dd2083-be5c-4522-a961-b86411e0cef8)

All screenshots in this post were taken based on this minimal vimrc.

```jsx
set nocompatible
set runtimepath^=/Users/xuliucheng/.vim/plugged/vim-clap
let g:clap_plugin_experimental = v:true
filetype plugin indent on
```

As you can see, vim and tree-sitter highlighting look pretty similar, as the colors in tree-sitter highlighting are linked to the highlight groups (`Function`, `Type`, etc) in your existing vim color scheme. (syntect uses its own color scheme, extra work is needed if we want to convert it to match Vim’s color scheme.) However, there are subtle differences if you take a closer look.  For instance, doc-comment in Rust `///` uses the same highlight as the normal comment, and the macro `tracing::debug!` is not highlighted, while Vim does this well.

The tree-sitter highlighting is overall decent but not perfect in many aspects. As indicated in [this GitHub issue](https://github.com/vim/vim/issues/9087), more efforts are needed to make tree-sitter a better and complete replacement for Vim highlighting. The tree-sitter ecosystem is still young but is increasingly growing, and even GitHub has begun to use it, before another solution is proposed, I believe tree-sitter has the potential as an option alongside Vim’s built-in regex-based highlighting.

As the ending words, I'd like to highlight that the tree-sitter highlighting support in vim-clap is at an super early stage, changes and bugs are expected. The tree-sitter highlighting may work pretty well for certain filetypes, but quite a number of language supports are still missing, especially there are no alternatives for many of non-language Vim syntax files at all. You can always fallback to the Vim highlighting whenever the tree-sitter support is missing or poor by running .`:ClapAction syntax/tree-sitter-highlight-disable | syntax on`.
