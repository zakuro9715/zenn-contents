---
title     : 静的型付けスクリプト言語 Cotowali
type      : tech
emoji     : 🔤
topics    : [vlang]
published : false
---

この記事は[未踏2021採択者アドカレ！](https://qiita.com/advent-calendar/2021/mitou-2021) の 23 日目の記事です。

# Cotowali とは

TODO

# 背景

## シェルスクリプトのメリット

シェルスクリプトのメリットは、シェルとしての機能面と、ポータビリティの高さだと考えています。

機能面では、コマンドを呼び出し、パイプで繋ぎ、リダイレクトを行うようなことが簡単にできます。
もちろん Python や Ruby でも同様のことは可能ですが、シェルスクリプトのほうがシンプルに記述できます。

このメリットを JavaScript で享受できるようにしたのが、Google の [zx](https://github.com/google/zx) です。

シェルスクリプトのもう一つのメリットは、言うまでもなくポータビリティです。

Ruby や Python、zx を使用するには、処理系が必要です。処理系が存在したとしても、バージョンの違いによる問題が発生します。

対して、シェルスクリプトの処理系は、Linux や macOS なら必ず存在します
また、POSIX sh に準拠したシェルスクリプトであれば、処理系の違いやバージョンの違いによる問題は発生しません。

## シェルスクリプトのデメリット

シェルスクリプトのデメリットで大きいのは、書くのがつらいことでしょう。
ある程度複雑なことをする場合、ポータビリティを犠牲にしてでも Python や Ruby で書きたくなってしまいます。

また、矛盾するようですが、シェルスクリプトはポータビリティの面でもデメリットがあります。
シェルスクリプトでポータビリティの高いスクリプトを書くのは意外と難しいのです。

shellcheck 等を利用すれば、ある程度のポータビリティのチェックはできます。
しかし、コマンドの GNU 拡張など、ポータビリティを損ねる罠は多く存在します。

# Cotowali が目指すこと

# 言語仕様 / サンプル
:
Cotowali の言語としての特徴は、二つの部分から構成されます。

1. 普通のプログラミング言語としての機能
2. シェルスクリプトの用途のための機能

## 普通のプログラミング言語としての機能

シェルスクリプトのつらさの要因として、一般的なプログラミング言語には見られない特殊性が挙げられます。

それは `if` - `fi` のような表層的なものから、式を直感的に書けないことや、関数が return で値を返さないこと等です。

Cotowali ではそれらを排除し、できるだけ平凡な文法を採用しています。

サンプルコードをいくつか見てみましょう。このような、普通の言語がシェルスクリプトに変換されます。

```fizzbuzz.li
fn fizzbuzz(i: int): string {
  if i % 3 == 0 && i % 5 == 0 {
    return 'fizzbuzz'
  } else if i % 3 == 0 {
    return 'fizz'
  } else if i % 5 == 0 {
    return 'buzz'
  } else {
    return "$i"
  }
}

for i in range(0, 20) {
  println(fizzbuzz(i))
}
```

## シェルスクリプトの用途のための機能

前述のとおり、シェルスクリプトは、コマンドを呼び出しパイプで繋ぐといった用途では他の言語よりも書きやすい優位性があります。

Cotowali ではそれらのシェルスクリプトの機能をサポートします。

### コマンド呼び出し

`@command` で任意のコマンド呼び出しをサポートします。

```
require 'platform'
const url = 'https://example.com'
if platform::has_command('curl') {
  @curl(url)
} else if platform::has_command('wget') {
  @wget('-O', '-', url)
}
```

### パイプライン

```
fn (n: int) |> twice() |> int {
  return n * 2
}
fn ...int |> sum() |> int {
  var (v, ret): (int, int)
  while read(&n) {
    ret += v
  }
  return ret
}
assert((range(1, 4) |> sum() |> twice()) == 12)
```

パイプライン演算子自体は他の言語にも存在する機能ですが、Cotowali のパイプライン演算子はそのままシェルスクリプトのパイプラインに変換されます。
これは、既存のコマンド呼び出しと組み合わせて使用できることを意味します。

```
assert(((1, 2) |> @awk('{print $1 + $2}')) == '3')
```

### インラインシェルスクリプト

インラインでシェルスクリプトを記述できます。また、ただ埋め込むだけではなく、`%name` でCotowali で定義した変数を利用できます。

```
var n = 10
var text: string
sh { %text="n = $%n" }
assert(text == 'n = 10')
```

# 利用方法

AdC に合わせて最初のバージョンをリリースしました。インストーラを使用してインストールでき、Linux と macOS をサポートします。
実用的な言語とは言い難いですが、遊んでいただければ幸いです。

## インストール (Konryu)

インストールにはバージョンマネージャである Konryu を使用します。下記のコマンドを実行し、表示される指示に従うと、lic コマンド(lic)、lish コマンド(REPL) が利用できます。

```
curl -sSL https://konryu.cotowali.org | sh
# 以下を .bashrc 等に追加ます
# export PATH="$HOME/.konryu/bin:$PATH"
# eval "$(konryu init)"
```

Konryu は Cotowali 自体で書かれています。
Cotowali が実用に耐えると言い難いのは事実ですが、しかしバージョンマネージャである Konryu を記述できる程度には動作します。

## Head を試す

現状では Konryu が対象とするのはリリースされたバージョンのみです。
Cotowali はまだまだ開発中であり、頻繁に変更されます。最新版である Head は、ソースコードからビルドするか、docker を通して使用します。

ソースコードからのビルドは [README](https://github.com/cotowali/cotowali#build-from-source) を参照してください。

docker イメージは `cotowali/cotowali-dev` です。これは Github Action から自動的に push されるため、常に最新の環境です。
ただし、開発用の環境を含んだイメージであるため、サイズが大きいことに注意してください。

## コンパイラの使用方法

`lic file.li` でコンパイル結果を標準出力に出力します。実行は `lic file.li | sh` あるいは `lic run file.li` です。


# PoC 実装

## PowerShell バックエンド

## Universal バックエンド（星屑^[StarDust]スクリプト）

> 左手には花束、右手には約束を。似て非なる言葉たち、今はお揃いの装いで……

```
```

> 私は確かに楽園を見た。放たれた銀の弾丸は窓を覆う白い布を赤く染め上げる。
> だがここは酸素に満ちた偽りの楽園。確かに見える星屑に、伸ばしたその手は届かない。
> それは、誰もが知る 3つの M に記されたるが如し。

# 展望

# おまけ: Cotowali の未踏性について
