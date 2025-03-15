---
title     : "[CSS]スクロールバーをヘッダの下に表示する"
type      : tech
emoji     : 🛝
topics    : [css, html, scrollbar]
published : true
---

## 概要

普通にページを実装した場合、スクロールバーはページ全体に表示される。

しかし、固定ヘッダだと、全体ではなくヘッダの下のみにスクロールバーを表示したくなることがある。

これを実装する。といっても CSS の基本がわかっていればシンプル。

## 結論

tailwind で書いているが、特別なことはないので普通の css でも一緒。

@[codepen](https://codepen.io/zakuro/pen/GgRyNbw?default-tab=html,result)

### ポイント

CSS をある程度知っている人は見ればわかると思うので、簡単に

- 画面サイズのコンテナの中に、ヘッダやスクロールする要素を入れる 
	- サイズは `h-dvh` で `height: 100dvh` を指定する。
	- `h-screen` や `height: 100vh` は特定のケースで期待した動作をしない。
		https://studio.design/ja/whats-new/viewport
- スクロールする要素は、`overflow-y: auto` に加えて、高さは決定されるようにすること。要素が伸びてしまうと、overflow しなくなってしまう。
	サンプルの場合は親が高さ固定の flex なので、要素の高さが決まっている。
- ヘッダは [`position: sticky`](https://developer.mozilla.org/ja/docs/Web/CSS/position#sticky) を使うのが楽。
- スクロールする要素内にも `sticky` を使うことで、常時表示される要素を作れる。

## 元ネタ

https://chat-gpt.com

ChatGPT では h-full を内側まで引き入れてきて実装している。