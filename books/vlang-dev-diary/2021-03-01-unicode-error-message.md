---
title: '2021/03/01 - エラーメッセージのアンダーラインがずれる問題を解決した'
---

<!-- textlint-disable ja-technical-writing/no-doubled-joshi -->
[昨日書いた](https://zenn.dev/zakuro9715/books/vlang-dev-diary/viewer/2021-02-28-east-asian-width)とおり、マルチバイト文字を使うとアンダーラインがずれる問題があります。
<!-- textlint-enbale -->

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
    7 |     mut n := 0
    8 |     n = '漢字'
vlib/v/checker/tests/error_with_unicode.vv:8:6: error: cannot assign to `n`: expected `int`, not `string`
    6 |     f2(0, '🐟🐧')
    7 |     mut n := 0
    8 |     n = '漢字'
      |         ~~~~~~~~
    9 |     n = 'ひらがな'
   10 |     n = '简体字'
vlib/v/checker/tests/error_with_unicode.vv:9:6: error: cannot assign to `n`: expected `int`, not `string`
    7 |     mut n := 0
    8 |     n = '漢字'
    9 |     n = 'ひらがな'
      |         ~~~~~~~~~~~~~~
   10 |     n = '简体字'
   11 |     n = '繁體字'
```

`encoding.utf8.east_asian` の実装は仕様準拠しているので、どうしてもテーブルが大きくなり、V言語はコンパイラのバイナリサイズを重視するために、これをコンパイラに使うことはできませんでした。

実際には表示幅を正しくするためには完全に仕様準拠する必要はなく、2文字幅の文字を検出するだけで事足ります。ということで、データとにらめっこしながらひたすら条件分岐を書きました。これによって、エラーのアンダーラインが概ね正しく表示されるようになりました。

[builtin/v.util: correct error underline for unicode wide chars #9010](https://github.com/vlang/v/pull/9010)

```
vlib/v/checker/tests/error_with_unicode.vv:5:17: error: cannot use `int literal` as `string` in argument 2 to `f1`
    3 |
    4 | fn main() {
    5 |     f1('🐀🐈', 0)
      |                ^
    6 |     f2(0, '🐟🐧')
    7 |     mut n := 0
vlib/v/checker/tests/error_with_unicode.vv:6:8: error: cannot use `string` as `int` in argument 2 to `f2`
    4 | fn main() {
    5 |     f1('🐀🐈', 0)
    6 |     f2(0, '🐟🐧')
      |           ~~~~~~
    7 |     mut n := 0
    8 |     n = '漢字'
vlib/v/checker/tests/error_with_unicode.vv:8:6: error: cannot assign to `n`: expected `int`, not `string`
    6 |     f2(0, '🐟🐧')
    7 |     mut n := 0
    8 |     n = '漢字'
      |         ~~~~~~
    9 |     n = 'ひらがな'
   10 |     n = '简体字'
vlib/v/checker/tests/error_with_unicode.vv:9:6: error: cannot assign to `n`: expected `int`, not `string`
    7 |     mut n := 0
    8 |     n = '漢字'
    9 |     n = 'ひらがな'
      |         ~~~~~~~~~~
   10 |     n = '简体字'
   11 |     n = '繁體字'
```

ちなみにこの実装を書いたところ、CI で `realloc` が落ちるというエラーが発生し、当初これが真剣にわかりませんでしたが、どうやら match が全ケースを map で持っているらしく、range で指定してもその範囲の値を全部持っているので、メモリを食い尽くしていたようです。つまり、`match c { 0...100 {} }` とかくと、0 と 100 だけではなく 0-100 のすべての数値を値として保持しているわけです。これは将来改善されると思いますが、現状では広い範囲を書くときは `match` を使うとコンパイル時にメモリを使うので避けたほうが良いのかもしれません。

ところで、Unicode の面白さにテストケースを考える楽しみというのがあります。今回のテストケースでは 🐀🐈 と 🐟🐧 という文字列を使用していますが、これは猫がネズミを食べ、ペンギンが魚を食べている様子を表しています。 East_Asian_Width のテストケースには 📛(Tofu on fire) を入れてみたりと、Emoji のテストは遊ぶ余地があり楽しいです。

