---
title: "2021/02/24 - if 式の条件に void が使える（当然落ちる）"
---

if の条件に void を使うコードをチェッカがスルーしていた。具体的にはこういうやつ。もちろん C レベルで落ちる。

```v
if println('') {
}
```

コメントいわく、未定義の変数などは void_type になるため、その場合はスキップしたいらしい。

たしかに、未定義の変数は別の場所でエラーになるから、ユーザーがエラーを読みやすくするために if の条件を skip するのは良いのかもしれない。
ただ、void_type は普通に void な関数にも当てはまるのでこのようなバグになる。そもそも未定義の変数の型は void とは別に用意したほうが良いのではないか。とはいえ、すでに未定義が void_type になることを前提にしたコードも多く、直すのも大変そうなので放置。

とりあえず if の条件式のチェックだけは直した。それから、if の条件式の型に関するテストがなかったので、それも追加した。

ちなみに、V言語のコンパイラには基本的に統合テストのみであり、UnitTest は存在しない。そもそもコンパイラの設計上、意味のある UnitTest を書くことができない。

- [checker: reject void type condition](https://github.com/vlang/v/pull/8737)
