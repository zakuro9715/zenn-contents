---
title: '2021/03/13 - V言語はエスケープシーケンスをそのまま C に渡している'
---

タイトルの通りで、V言語はエスケープシーケンスをチェックはするが、特に何もせずに C の文字列として吐き出している。

無効なエスケープシーケンスのチェックはしているので、基本的にはこの方針で問題ない。

ただし、1つだけ問題になるものがある。そう、Unicode だ。C言語の仕様上、`\uXXXX` は存在するものの、いくつか特殊な制約がついている。たとえば、ascii の範囲はいくつかを除いて禁止されている。

> 2 A universal character name shall not specify a character whose short identifier is less than
>   00A0 other than 0024 ($), 0040 (@), or 0060 ('), nor one in the range D800 through
>   DFFF inclusive.62)

また、いくつかのコンパイラは `\u00XX` を Unicode のコードポイントではなく `\xXX` と同じように扱う。

https://github.com/vlang/v/runs/2093700018#step:11:15

```
   > assert '\\u00c4'.bytes() == 'Ä'.bytes()
     Left value: [0xc4]
    Right value: [0xc3, 0x84]
```

このため、`\uXXXX` をそのまま Cコンパイラに渡してしまうと、Unicode リテラルとして機能しないケースが発生する。

[Unicode escaping seems not to work correctly #6317](https://github.com/vlang/v/issues/6317)

とりあえず ascii の範囲の `\u00XX` を `\xXX` に置き換える PR を作ってみた。

[scanner: replace ascii unicode(\u0020) with hex(\x20) #9259](https://github.com/vlang/v/pull/9259)

ただ、説明不足すぎたのか意図が全く伝わらなかった。現状で CI 全部通ってるじゃんという反応。V言語のコミュニティになれてきたのもあり、最近は説明をサボりがちだったのは少し反省。この反応はある程度当然のことで、普通 `\u0020` が使用できないなどとは思いもしないことだろう。なぜダメなのかはわからない。

確かに CI は通っているのだが、よく見るとそもそも文字列のエスケープに関する正常系のテストが全く存在しなかったりした。基本的なところのテストがないというのは結構よくあることで、とはいえ言語処理系の網羅的なテストは意外と難しい。他の言語はどうやっているのかと rust コンパイラを見てみたら、ひたすら大量のテストファイルが存在した。やっぱり見つけたものをテストにしていくしかないらしい。

ちなみにこれに限らず、V言語は規格的に怪しい C のコードを吐いている部分がある。例えばアンダースコア2つで始まったり、アンダースコアと大文字から始まったりする識別子を使用している。複数のコンパイラでチェックしているから致命的なものはないとは思うが、safety で no undefined behavior を謳うのであれば、出力する C のコードにはもう少し気をつけてほしいと思う。この雰囲気だと undefined behavior を踏んでいる可能性は十分にある。

