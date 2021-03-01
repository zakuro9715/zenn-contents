---
title: '2021/02/28 - Unicode の East_Asian_Width を標準ライブラリに実装した'
---

よくある話ですが、V言語はマルチバイト文字の扱いが適当です。

私は確認していませんが、discrod の日本語チャンネルでは、v-ui で日本語を使ったらバグったという報告もありました。

ところで、V言語ではある程度ユーザーフレンドリーなエラーメッセージを表示します。

```
main.v:1:1: error: expected 1 arguments, but got 0
    1 | println()
      | ~~~~~~~~~
```

このような感じですね。

さて、先程マルチバイト文字の扱いが適当だということを書きました。ということは、嫌な予感がしますね。試してみましょう。

```
vlib/v/checker/tests/error_with_unicode.vv:5:17: error: cannot use `int literal` as `string` in argument 2 to `f1`
    3 |
    4 | fn main() {
    5 |     f1('🐀🐈', 0)
      |                    ^
    6 |     f2(0, '🐟🐧')
    7 |     mut n := 0
vlib/v/checker/tests/error_with_unicode.vv:6:8: error: cannot use `string` as `int` in argument 2 to `f2`
    4 | fn main() {
    5 |     f1('🐀🐈', 0)
    6 |     f2(0, '🐟🐧')
      |           ~~~~~~~~~~
```

はい。見事にずれています。アンダーラインの表示はどうやら1バイトが1文字という表示になっているようです。

文字の表示幅については、unicode に East_Asian_Width の定義があり、これを実装する必要がありそうです。この実装については他の言語にもいろいろなライブラリがあるため、それらを参考に、適当にテーブルを作り二分探索で判定する実装を行いました。

[utf8: support East_Asian_Width](https://github.com/vlang/v/pull/8978)

実装自体はマージされたのですが、下記のようなコメントがついています

> This new addition will add between 36KB and 85KB (because of all the constants), to every program that imports encoding.utf8, including the V compiler itself (it imports it through v.scanner), even if the functions are not used (-skip-unused is not good enough for now).
>
> Can it be moved in vlib/encoding/utf8/east_asian

コンパイラ本体では使いたくないという雰囲気が濃厚です。

ということで、エラーメッセージのアンダーラインがずれる問題については別途対処する必要がありそうです（実は執筆時点でこの問題は修正済みですが、その話はまた後日）。

ところで、Unicode には得てして政治的な問題がつきまといます。テストする場合、何も考えずに日本語だけのテストを書くようでは問題があります。テストケースとしても問題ですし、得てして政治的な問題もつきまといます。歴史的経緯から、日本人が日本語のテストケースだけ書き、韓国語や中国を無視した場合、（単なる考慮不足でも）あまり嬉しくない問題が発生する可能性があります。このあたりは日本にいると軽く考えがちですが、基本的には気を使えるだけ使っておくのが無難でしょう。

しかし、日中韓くらいなら文字のことを知っているので問題ないのですが、文字によってはどう実装したりテストすればいいのかさっぱりわからないものもあります。加えて、しばしば vim が壊れます。私の環境の vim では、アラビア語の前にテストを挿入したりすると表示がおかしくなります。以前は絵文字でも壊れていた気がしますが、今は流石に大丈夫みたいです。

Unicode 関連の面白話は、[Turing Complete FM: 12. Unicode、絵文字、Androidのテキスト関連のハンドリング、無数の文字トリビア (のな)](https://turingcomplete.fm/12) がおすすめです。East_Asian_Width のような属性値の取得については比較的単純ですが、レンダリング周りをやる人は大変だなあと思います。
