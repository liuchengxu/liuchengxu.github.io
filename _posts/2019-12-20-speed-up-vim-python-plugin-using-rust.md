---
bg: "tools.jpg"
layout: post
title:  "Make Vim Python plugin 10x faster using Rust"
crawlertitle: "Speed up your vim python plugin by using Rust"
summary: "Using Rust to speed up the built-in fuzzy filter of vim-clap written in Python"
categories: posts
tags: ['vim']
author: Liu-Cheng Xu
---

Several months ago, I released my favorite Vim plugin by the far: [**vim-clap**](https://github.com/liuchengxu/vim-clap), which is a generic fuzzy finder and dispatcher, using the modern `popup`(Vim) and `floating_win`(NeoVim) feature, written in Pure VimScript. Of course, it's asynchoronous. At the beginning, I'm very proud of that clap is in pure vimscript in that it will have the minimal pain to setup, neither extra compiled feature nor external dependency is required, just works well on all the platform for both Vim and NeoVim.

![](https://user-images.githubusercontent.com/8850248/66710372-7df56180-eda9-11e9-8c41-ffbc462a5340.gif)

Assuming you are using [vim-plug](https://github.com/junegunn/vim-plug) and the newer Vim/NeoVim:

- Vim: `:echo has('patch-8.1.2114')`
- NeoVim: `:echo has('nvim-0.4')`

put this line in your vimrc is all you need to install vim-clap:

```vim

Plug 'liuchengxu/vim-clap'

```

## Introduce optional Python dependency

It's in pure vimscript that brings vim-clap the feature of minimal installtion requirements. However, that feature becomes to a subtle _bug_ over time. You know, VimScript is never known as a performant script language, instead it's much slower than the popular Python and Lua. You can never apply these complex logic in vimscript and expect your plugin to run very well.

No one wants to use a Vim plugin that is slow, so one principle of [vim-clap](https://github.com/liuchengxu/vim-clap) is always to run asynchoronously so that the UI will never be blocked. I thought using the async job would be the ultimate solution, but it's not unfornately. The asynchoronous job does help mostly, but not completely. The first thing is that job of Vim/NeoVim causes a `redraw` problem, you can see the flicker obviously. What's more, using the async job only can not let Vim remain reponsive when dispatching these CPU-intensive tasks, [see this issue](https://github.com/liuchengxu/vim-clap/issues/75).

![](https://user-images.githubusercontent.com/8850248/67620599-3b1c9a80-f83b-11e9-8d8c-72bfae9d9177.gif)

(Note the flicker happens in `:Clap grep`, that doesn't happen in `:Clap blines` )

As I have said, vim-clap can be served as a fuzzy finder. With vimscript only, you can only handle a small few thousand items and have the limited regex based substring fuzzy filtering at that moment. There is no advanced fuzzy filtering algorithm written in vimscript.

One solution is to leverage the existing fuzzy finders, e.g., [fzy](https://github.com/jhawthorn/fzy), [fzf](https://github.com/junegunn/fzf), [skim](https://github.com/lotabout/skim), using the async `job` feature which already exists in both Vim and NeoVim for a while . And quickly the external fuzzy filter support is added, now you could use `:Clap! blines ++ef=fzf` to filtering the buffer lines using fzf. But I'm actually still not satisfied as the external fuzzy filter has to rely on the async job, the `redraw` issue of which has been settled though.

Then I began to resort Python or lua to achive faster and better built-in fuzzy filtering in order to avoid the `redraw` issue of async job. Fortunately, I found [aslpavel/sweep.py](https://github.com/aslpavel/sweep.py/blob/master/sweep.py) which has implemented the [fzy fuzzy finder having better results than other fuzzy finders](https://github.com/jhawthorn/fzy/blob/master/ALGORITHM.md). Then I shamelessly borrowed its internal fzy implementation into [vim-clap](https://github.com/liuchengxu/vim-clap), [see the initial PR]( https://github.com/liuchengxu/vim-clap/pull/92).

With Python, now we can handle 10,000 items and have the best fuzzy finder embedded in Vim.

## Make Python filter 10x faster using Rust

The introduced Python fzy implementation in vim-clap can handle 10,000 items decently, but that's far from enough from my perspective. And with some profile, I found the bottleneck is this fuzzy filtering function. Once I read about this post [Speed up your Python using Rust](https://developers.redhat.com/blog/2017/11/16/speed-python-using-rust/), I immediately realized that that's a fairly promising optimization.

Thanks to [PyO3](https://github.com/PyO3/pyo3), now writting a Python dynamic module in Rust is trivial and elegant. What you need to learn is the exmple on the README of PyO3 and rewritting the related function in Python to Rust. As a matter of fact, I firstly used [rust-cpython](https://github.com/dgrunwald/rust-cpython), but I ran into some issues on macOS, hence I switched to PyO3 and it works smoothly.

Thanks to [stewart/rff](https://github.com/stewart/rff), the fzy Rust implementation is already good to go.

What I have to do is to wrap the Rust version fzy and replace the pure Python fzy in [vim-clap](https://github.com/liuchengxu/vim-clap) with the generated Python dynamic module.

I did some tests laster, and found that the optimization result is very satisfying.

From pytest, Rust is **30x** faster:

```
--------------------------------------------------------------------------------------------- benchmark: 6 tests --------------------------------------------------------------------------------------------
Name (time in ms)                   Min                    Max                   Mean             StdDev                 Median                IQR            Outliers      OPS            Rounds  Iterations
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
test_rust_10000                 10.0804 (1.0)          35.7376 (1.0)          11.9004 (1.0)       4.4393 (1.04)         10.6368 (1.0)       0.5522 (1.0)          4;16  84.0305 (1.0)          91           1
test_pure_python_10000         317.4718 (31.49)       328.1628 (9.18)        324.2129 (27.24)     4.2679 (1.0)         323.9821 (30.46)     5.5182 (9.99)          1;0   3.0844 (0.04)          5           1

test_rust_100000               157.7825 (15.65)       193.6999 (5.42)        182.2141 (15.31)    14.0614 (3.29)        187.0058 (17.58)    11.5470 (20.91)         1;1   5.4880 (0.07)          5           1
test_pure_python_100000      5,077.7557 (503.73)    5,199.0074 (145.48)    5,143.5521 (432.22)   45.2209 (10.60)     5,157.5873 (484.88)   55.6680 (100.82)        2;0   0.1944 (0.00)          5           1

test_rust_200000               427.0450 (42.36)       462.5228 (12.94)       450.2531 (37.84)    13.5722 (3.18)        453.5792 (42.64)    11.7978 (21.37)         1;1   2.2210 (0.03)          5           1
test_pure_python_200000     12,122.4214 (>1000.0)  12,300.0331 (344.18)   12,238.9739 (>1000.0)  72.1199 (16.90)    12,256.4216 (>1000.0)  97.0552 (175.77)        1;0   0.0817 (0.00)          5           1
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

From the profile of Vim/Neovim(`:h profile`), we can consider this as the true enhancement we can feel practially:

```
stats of pure Python fuzzy filter performance:

total items: 100257
[once]
====== vim ======
FUNCTION  <SNR>37_ext_filter()
1   5.951908             <SNR>37_ext_filter()

====== nvim ======
FUNCTION  <SNR>43_ext_filter()
1   6.592639             <SNR>43_ext_filter()

total items: 100257
total items: 100257
[multi]
====== vim ======
FUNCTION  <SNR>37_ext_filter()
3  11.148641             <SNR>37_ext_filter()

====== nvim ======
FUNCTION  <SNR>43_ext_filter()
3  12.655566             <SNR>43_ext_filter()

[bench100000]
====== vim ======
FUNCTION  <SNR>23_ext_filter()
1   5.400793             <SNR>23_ext_filter()

====== nvim ======
FUNCTION  <SNR>27_ext_filter()
1   5.921336             <SNR>27_ext_filter()
```

```
stats of Rust fuzzy filter performance:

total items: 100257
total items: 100257
[once]
====== vim ======
FUNCTION  <SNR>37_ext_filter()
1   0.352795             <SNR>37_ext_filter()

====== nvim ======
FUNCTION  <SNR>43_ext_filter()
1   0.940487             <SNR>43_ext_filter()

total items: 100257
total items: 100257
[multi]
====== vim ======
FUNCTION  <SNR>37_ext_filter()
3   0.843340             <SNR>37_ext_filter()

====== nvim ======
FUNCTION  <SNR>43_ext_filter()
3   2.221188             <SNR>43_ext_filter()

[bench100000]
====== vim ======
FUNCTION  <SNR>23_ext_filter()
1   0.306407             <SNR>23_ext_filter()

====== nvim ======
FUNCTION  <SNR>27_ext_filter()
1   0.865437             <SNR>27_ext_filter()
```

Due the overhead of calling Python from Vim/Neovim, rewritting in Rust only makes Vim **16x** faster and NeoVim **7x** faster.

Have a look at [vim-clap/pythonx/clap](https://github.com/liuchengxu/vim-clap/tree/master/pythonx/clap) and [vim-clap/test](https://github.com/liuchengxu/vim-clap/tree/master/test) if you want to run the test yourself.

## Conclusion

- By rewritting the pure Python function in Rust, filtering 100,000 items takes **5.4s** to **0.3s** for Vim.

- The `pynvim` of NeoVim is slower than `python` of Vim, especially for the Python dynamic module(3x slower). Maybe the NeoVim dev guys should check it out why it's much slower(_or let me know if the test is unfair_).

- Writting Python dynamic module in Rust is pretty easy. If you wrote some Vim plugin in Python, suffering some performance problems, have a try with Rust!

Now you can easily install [**vim-clap**](https://github.com/liuchengxu/vim-clap) with the optional compiled Rust extension via the plugin manager:

```vim

" The Rust extension will only be installed if cargo is available.
Plug 'liuchengxu/vim-clap', { 'do': function('clap#helper#build_all') }

```

Note that even using Rust makes the built-in filtering 10x faster, that does not totally tackle the performance issue of filtering/searching at scale inside Vim/NeoVim. Refer to [this performance tracking issue](https://github.com/liuchengxu/vim-clap/issues/140) if you are interested. And any help would be appreciated.

Furthermore, I'm honestly not a Rust expert and the current Rust implementation is definitely not the most swift. I believe there are more talented Rustceans that are vimmers too. Feel free to [make it faster again!](https://github.com/liuchengxu/vim-clap/blob/master/pythonx/clap/fuzzymatch-rs/src/lib.rs).
