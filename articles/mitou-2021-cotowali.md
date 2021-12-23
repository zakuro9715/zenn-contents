---
title     : 静的型付けスクリプト言語 Cotowali
type      : tech
emoji     : 🔤
topics    : [vlang]
published : false
---

この記事は[未踏2021採択者アドカレ！](https://qiita.com/advent-calendar/2021/mitou-2021) の 23 日目の記事です。

# Who are you

[zakuro](https://twitter.com/zakuro9715) です。もともと Web 周りの仕事をしていましたが、いろいろあった結果無職になり、未踏に採択されて今に至ります。

無職になったタイミングで [V言語](https://github.com/vlang/v)の開発に参加し始め、V Organization Mmeber でもあります。

# Cotowali について

[Cotowali](https://cotowali.org) は、2021年度未踏に「[シェルスクリプトへのコンパイルを行う静的型付けスクリプト言語の開発](https://www.ipa.go.jp/jinzai/mitou/2021/gaiyou_tk-3.html)」のテーマで採択され、現在開発中のスクリプト言語です。

https://cotowali.org

https://github.com/cotowali/cotowali

Cotowali コンパイラは V言語で記述されています。Ｖコミュニティ公式ではないプロジェクトとしては規模が大きく、V言語コンパイラのバグ発見にも貢献しています。

## コンセプト

- POSIX 準拠シェルスクリプトへのトランスパイル
- シェルスクリプトの機能を取り入れながらも、一般的な言語に近い理解しやすいシンタックス
- シンプルな性的型付け

ちなみに、Cotowali はあくまでシェルスクリプトをバックエンドに使う新規のスクリプト言語であり、シェルスクリプトに型をつけるものではありません。

静的型付けではありますが、型システムはごく簡易なものです。強い型付けによる強力なサポートではなく、ケアレスミスの防止のような補助的な用途を目的としています。

## 想定用途

これまでシェルスクリプトを使用していたような場面で使える、書きやすい言語としての用途を想定しています。また、シェルスクリプトの書きづらさを嫌ってポータビリティを犠牲に Ruby や Python が選択される場合がありますが、そのような場面でポータビリティを犠牲にしなくてもいい選択肢として選べる言語になることも目的としています。

具体的なケースとして、以下のような例を想定しています。

- `curl -sSL http://... | sh` でインストールするタイプのツールのインストールスクリプト
- プロジェクトのセットアップスクリプト
- Docker の entrypoint スクリプト
- CI で利用するスクリプト

反対に、以下のようなケースは Cotowali のターゲット外です

- Web アプリ
- 複雑な CLI ツール
- ある程度パフォーマンスを気にする必要があるスクリプト

シンプルに言えば、小規模でポータビリティがあることが望ましいものが想定用途。大規模だったり、パフォーマンスを重視するものはターゲット外です。

# 言語仕様 / サンプル

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

```require.li
// require は単にファイルを読み込みます。module を作ることはありません。
// また、コンパイル時に処理されるため、source コマンドとは別物です。出力時には単一ファイルになります。
require 'os'                                           // std / COTOWALI_PATH
require './mymod'                                      // relative
require 'github:cotowali/cotowali@main/tests/hello.li' // github
require 'https://.../hello.li'                         // http
```

```module.li
module mod {
  fn hello() { println("hello mod") }
}

module mod::submod {
  fn hello() { println("hello mod::submod") }
}

mod::hello()
mod::submod::hello()
```

```type.li
// type alias
type Vec2 = (float, float)

// methods
fn (v: Vec2) x(): float { return v[0] }
fn (v: Vec2) y(): float { return v[1] }

// operator overload
fn (lhs: Vec2) + (rhs: Vec2): Vec2 {
  return (lhs.x() + rhs.x(), lhs.y() + rhs.y())
}
```

```array.li
// Array は参照ではなく値です。

fn sum(vals: []int): int {
  var n = 0
  for v in vals {
    n += v
  }
  return n
}
sum([0, 1, 2])
```

## シェルスクリプトの用途のための機能

シェルスクリプトは単に分かりにくいだけの言語ではありません。コマンドを呼び出しパイプで繋ぐといった用途では、他の言語よりもシンプルに記述できる優れた側面もあります。

Cotowali ではそのようなシェルスクリプトの機能を取り入れています。

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

パイプライン演算子自体は他の言語にも存在する機能ですが、Cotowali のパイプライン演算子はそのままシェルスクリプトのパイプラインに変換されます。これは、既存のコマンド呼び出しと組み合わせて使用できることを意味します。

```
assert(((1, 2) |> @awk('{print $1 + $2}')) == '3')
```

シェルスクリプトでは行ごとの操作を行うことが多いです。これを反映したものが `...int` のような sequence 型であり、シェルスクリプトと同様にパイプでつなげて行ごとに処理する書き方ができます。

```
println(
  cat(data.txt)
    |> filter('foo')
    |> replace('foo', 'bar')
    |> head(3)
    |> join('\n')
)
```

### リダイレクト

リダイレクトによるファイルへの書き込みをサポートします。パイプラインの右端が文字列値の場合リダイレクトとして扱います。

現状では分かりやすさを優先して標準出力をファイルに書き込むリダイレクトのみをサポートしています。現時点では `2>&1` 等に相当するものはサポートしていません。

```
10 |>  'data.txt' // echo 10 > data.txt
20 |>> 'data.txt' // echo 20 >> data.txt

fn filename(): string { return 'file.txt' }
cat('data.txt') |> first() |> filename()

do_something() |> null // redirect to '/dev/null'
```

### インラインシェルスクリプト

インラインでシェルスクリプトを記述できます。また、ただ埋め込むだけではなく、`%name` でCotowali で定義した変数を利用できます。

```
var n = 10
var text: string
sh { %text="n = $%n" }
assert(text == 'n = 10')
```

## その他の構文

詳細は[ドキュメント](https://cotowali.org/ja/docs/getting-started/)を参照してください、と言いたいところなのですが、そちらの方には手が回っておらず十分ではありません。一応ある程度までは記述してあるため、参考程度にはなると思います。

より詳細について知りたい場合は、[テスト](https://github.com/cotowali/cotowali/tree/main/tests)および[標準ライブラリ](https://github.com/cotowali/cotowali/tree/main/std)を参照してください。

# 利用方法

AdC に合わせて最初のバージョンをリリースしました。インストーラを使用してインストールでき、Linux と macOS をサポートします。
実用的な言語とは言い難いですが、遊んでいただければ幸いです。

## インストール (Konryu)

インストールにはバージョンマネージャである [Konryu](https://github.com/cotowali/konryu) を使用します。下記のコマンドを実行し、表示される指示に従うと、koryu コマンド、lic コマンド(コンパイラ)、lish コマンド(REPL) が利用できます。

```
curl -sSL https://konryu.cotowali.org | sh
# 以下を .bashrc 等に追加ます
# export PATH="$HOME/.konryu/bin:$PATH"
# eval "$(konryu init)"
```

正常にインストールが完了し、PATH を正しく通していれば、以下のコマンドで Hello World が実行できます。

```
echo 'println("Hello World")' | lic run
```

konryu の使い方は help を参照してください^[なお、コマンドを実行した際、単に http error と表示されることがあります。これは GitHub API のレートリミットの可能性が非常に高いです。Konryu では 1 コマンドで最大 1 回の API アクセスを行います。GitHub トークンの設定や結果のキャッシュは今のところ未実装です。]

```
Konryu - Cotowali installer and version manager

Usage: kornyu [options] [command] [version]

Options:
  -h --help - Print help message

Commands:
  help      - Print help message
  init      - Print shell code to configure environment
  install   - Install cotowali release
  uninstall - Uninstall specified version
  use       - Use specified version
  releases  - List available cotowali releases
  versions  - List installed cotowali versions

  update  - Update konryu
  destroy - Destroy konryu and all installed files
```

Konryu は Cotowali 自体で書かれています。現在はリリースが一つしかないため、実質的にインストーラとしての役割しかありませんが、新しいバージョンがリリースされた場合、Konryu からインストールし、またバージョンを切り替えることができます。

Konryu はまさに Cotowali のターゲットとするユースケースであり、最初のリリースのためのマイルストーンでした。このようなスクリプトを記述できる程度には、あ Cotowali が動作することを示せているはずです。

## Head を試す

現状では Konryu が対象とするのはリリースされたバージョンのみです。
Cotowali はまだまだ開発中であり、頻繁に変更されます。最新版である Head は、ソースコードからビルドするか、docker を通して使用します。

ソースコードからのビルドは [README](https://github.com/cotowali/cotowali#build-from-source) を参照してください。

docker イメージは `cotowali/cotowali-dev` です。これは Github Action から自動的に push されるため、常に最新の環境です。
ただし、開発用の環境を含んだイメージであるため、サイズが大きいことに注意してください。

## コンパイラの使用方法

`lic file.li` でコンパイル結果を標準出力に出力します。実行は `lic file.li | sh` あるいは `lic run file.li` です。

## Playground / オンラインコンパイラ

ちょっと試すために処理系をインストールするのは面倒、という方のために、Playground を用意しています。

https://cotowali.org/play

また、オンラインコンパイル API も用意しているため、処理系がなくても以下のように手元で実行できます

```
# 現時点では https 非対応 です
curl http://lic.cotowali.org -d 'println("Hello World")' | sh
```

Heroku を利用しているため、初回のリクエストには1分程度かかることがあります。また、手動デプロイのため、最新版とは限らないことに注意してください

# 標準ライブラリ

Cotowali はポータビリティを重視します。
しかし、`@command` による直接のコマンド呼び出しや、インラインシェルスクリプトの使用は容易にポータビリティを損ねます。

ポータビリティを重視して書かれた標準ライブラリが多くの範囲をカバーすることで、
直接のコマンド呼び出しやインラインシェルスクリプトをユーザーが利用することなく、
自然にポータビリティの高いスクリプトが書ける言語になることを目指しています。

標準ライブラリにはまだまだ手が回っていないのが現状ですが、現時点で存在する、いくつかの特徴的なライブラリを紹介します。

- input_tty
    入力を受け取る input() は存在しますが、シェルスクリプトに変換する特性上、標準入力からの入力ではユーザーの入力を適切に処理できないことがあります。

    たとえば、`curl url | sh` のような利用方法において、stdin は入力をとるために利用できません。

    これを解決するために、tty から入力を読み取る `input_tty` 関数が実装されています

    ```
    const name = input_tty('Who are you ?')
    println("Hello $name")
    ```

- tar.li
    tar コマンドのオプションは覚えていますか？ 熟練のプログラマにとっては常識かもしれませんが、
    少なくとも初心者には正直わかりづらいです。私も毎回調べていました。

    また、できるだけ標準ライブラリだけでスクリプトを書けるようにするという目的から考えても、tar は使用頻度が高く、標準ライブラリに必須です。

    Cotowali のでは、標準ライブラリ `tar` を require することで、
    `tar::create_to('file.tar', 'dist')` `tar::extract_file('file.tar')` のように使用できます。

    現状では、tar コマンドの x(Extract), c(create), f(file), C(dir), z(gzip) に対応する関数がサポートされています。

- platform.li
    OS やシェル等の判定や、コマンドの存在確認ができます。

- http.li
    `http::get(url)` が実装されています。curl や wget の複数のコマンドをラップすることでポータビリティのある http リクエストを行えます。
    また、busybox の wget もサポートします。

# 最後に

未踏で開発中の言語 Cotwali についての簡単な紹介でした。

まだまだ実用的な言語には遠く、うまく動かないかもしれませんが、気が向いたら遊んでいただければ幸いです。

ところで、GitHub の Star が増えると継続のモチベーションにもなりますし、単純に喜びます。

https://github.com/cotowali/cotowali
