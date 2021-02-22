---
title: "2020/02/22 - 可視性のバグ"
---

複数存在する同系統のバグです。タイトルの通りで、pub フィールドの設定を忘れていて pub がついてるのに外部からアクセスできないとか、逆に is_pub をチェックしていなくて pub じゃないのに外部からアクセスできるなど。

V言語コンパイラはいろいろな場所に可視性プロパティを持ち、可視性チェックを行っています。そのため、時々漏れが発生するわけです。

## [checker: check private constant #8501](https://github.com/vlang/v/pull/8501)

この修正前は、pub 指定していない const にアクセスできていました。原因はチェッカが可視性チェックをしていなかったことです。ついでにパーサが AST を生成するときに is_pub を設定していなかったので、const は pub をつけようがつけまいが内部的には private になっていました。

ちなみに、vlib はいろいろな箇所で pub をつけていない const にアクセスしていたため、どちらかというとそちらの修正が面倒でした。

## [parser: set is_public correctry when registering enum type symbol #8875](https://github.com/vlang/v/pull/8875)

これも単純に設定忘れです。EnumDecl にも is_pub はあり、そちらは設定されていましたが、TypeSymbol の is_public は忘れられていたようです。

