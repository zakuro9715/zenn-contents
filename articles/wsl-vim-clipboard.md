---
title     : WSL + vim で Windows とクリップボードを共有する
type      : tech
emoji     : 🐢
topics    : [wsl, vim, tmux]
published : true
---

# 概要

WSL 上の vim では、デフォルトでは Windows とクリップボードを共有できません。

この記事では、WSL 上の vim で Windows とクリップボードを共有する方法を紹介します。

# 方法

## 方法 1

WSL 上の vim は X の clipboard を使用します。そのため、VcXsrv などを使用して X を使えるようにすれば、クリップボードを共有できます。

## 方法 2

私が採用しているのはこちらです。

clip.exe を使用して Windows 側のクリップボードを操作します。

```.vimrc
vnoremap <RightMouse> :w !clip.exe<CR><ESC>
```

キーバインドではなく、Yank 時に自動で同期させたい場合は以下のようにします。

```.vimrc
augroup Yank
  au!
  autocmd TextYankPost * :call system('clip.exe', @")
augroup END
```

tmux との兼ね合いもあって、貼り付けには Windows Terminal の Paste (Shift + Right Click) を利用しています。
