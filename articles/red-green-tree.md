---
title: 新しい構文木、Red Green Trees / Lossless Syntax Tree 概観
type: tech
published: false
---

## 概要

本記事では、近年採用されるケースが増えた Red Green Tree あるいは Lossless Syntax Tree と呼ばれる構文木について解説します。

### 発明と浸透

このデータ構造は、Roslyn のチームによって発明されました。

後述するいくつかの特徴的で顕著なメリットを有しており、近年の言語処理系で採択されるケースが増えています。

具体的には以下の処理系で採用されています。

- Swift (libsyntax)
- Kotlin
- rust-analyzer
- oil-shell
- biome(旧rome)

## 名称

このデータ構造は、Red Green Tree あるいは Lossless Syntax Tree と呼ばれています。ただし、Red Green Tree という名称は浸透しておらず、Lossless Syntax Tree という名称は、名称というよりも特徴を表すタイトルとして使用されているように感じます。

よって、このデータ構造に統一的な名称はありません。手法自体は浸透していますが、それぞれが独自の名称を使用したり、あるいは特に名前を付けていなかったりします。

実装のみが引き継がれた結果、統一的な名前が付かなかったのかもしれません。

Red Green Trees という名前は、Roslyn でのアイデア発案時に使用されたようですが、前述のとおりあまり浸透していません。

Lossless Syntax Tree という名前が使用されるケースもあります。この名前は Lossless であることを表すため、厳密には Red Green Tree のみを指すものではないと考えられます。しかし、ほとんどの場合で Losless Syntax Tree は Red Green Tree を指しています。

Lossless Syntax Tree は必ずしも Red Green Tree を指すとは限らないこと、実装の解説で RedNode や GreenNode といった単語を使用することから、本記事においては原則として Red Green Tree の名前を使用します。

## 前提知識

### AST と CST

言語処理系において、もっともよく使用されるのが AST(Abstract Syntax Tree) です。

ところで、AST とはどういうものでしょうか。当たり前に使用されすぎて考えることもなかったかもしれませんが、一度振り返ってみましょう。

AST とは Abstrcat Syntax Tree であり、日本語では抽象構文木です。つまり、なにか抽象化された構文木ということです。

対照的なものとして、CST があります。CST は Concrete Syntax Tree であり、具象構文木です。

この二つをまとめたのが以下の図です。

```
図
```

抽象構文木の方が情報が減っているのがわかります。つまり、抽象化しているのです。

普段使われる AST では、抽象化のレベルはもう少し低くなります。たとえば、エラー位置を表示するために位置情報は必要です。

しかし、一定の抽象化は行われ、ソースコードの時点から失われる情報があります。つまり、AST からは元のソースコードが正確にはわかりません。

具象構文木は、抽象構文木よりも多くの情報を持っています。元のソースコードを復元できるかは実装次第です。

Lossless Syntax Tree の Lossless とは、元のソースコードから情報を失わないという意味です。


# Green Node

子のリストを持つ
Red Node を通して差し替える

# Red Node

GreenNode のラッパー的なもの
親への参照を持つ
GreenNode への参照を持つ
サイズが小さい

- 子のリスト取得
    - GreenNode は子のリストを持っている
    - GreenNode から子のリストの GreenNode を取得。
    - 自身を親に、取得した GreenNode を持つ RedNode を作成できる

- 差し替え
    - 子は毎回作成されるので、GreenNode を差し替えれば、子も自動的に差し替えられる
    - 親のGreenNodeの子を差し替えた新しいGreenNodeを作成する。この時点では木全体に変更が反映されない
    - RedNodeの親をたどり、子を差し替えたGreenNodeへと入れ替えていく
    - 親がなくなったら停止する。この時点でルートまですべてのノードが差し替えられる
    - その性質上、差し替えコストは深さに依存するため、全書き換えよりも効率が良い
