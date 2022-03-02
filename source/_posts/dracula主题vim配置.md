---
title: dracula主题vim配置
date: 2021-04-08 17:41:27
tags:

---

最近喜欢用Dracula主题，主题官网：https://draculatheme.com/



在mac上设置了Dracula全家桶，设置vim主题的时候，效果一直不是很好，找了很多资料，最后找到了一个比较满意的配置，记录一下

```bash
packadd! dracula
syntax enable
colorscheme dracula
set hlsearch
set nu
set termguicolors
highlight Normal ctermbg=None
let &t_ZH="\e[3m"
let &t_ZR="\e[23m"
```

主要是最下面四行配置，`highlight Normal ctermbg=None`能让vim的背景和iterm2的背景相同，最后两行能让一些奇怪的高亮恢复正常，还能显示斜体