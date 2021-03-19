---
title: '2021/03/05 - 謎の diff'
---

fmt に変更を入れたあと、PR を出そうと思ったら何故か cleancode の CI が落ちている。よく見ると謎の diff。

```
--- a/vlib/v/fmt/fmt.v
+++ b/vlib/v/fmt/fmt.v
@@ -1736,7 +1736,7 @@ pub fn (mut f Fmt) if_expr(node ast.IfExpr) {
        f.single_line_if = false
        if node.post_comments.len > 0 {
                f.writeln('')
-               f.comments(node.post_comments,
+               f.comments(node.post_comments,
                        has_nl: false
                        prev_line: node.branches.last().body_pos.last_line
                )

```


<!-- textlint-disable ja-technical-writing/no-doubled-joshi -->
まあ謎というほどでもなく、行末のスペースが削除されていた。行末のスペースは vim が保存時に削除する設定になっている。

問題は、なぜ行末にスペースがあったのかということだ。`fmt.v` はフォーマット対象なので、常にフォーマット済みである。ということは、v fmt がスペースを挿入しているということだ。

V言語にコントリビュートを始めた当時なら途方にくれたことだろうが、なれてきたものでだいたい察しはつく。引数のあとに `, ` を入れて、その後改行する場合はスペースを削除するが、struct の短縮記法ではこの処理が抜けているということだろう。

調べてみると、実際そのとおりであった。通常の引数で複数行に分割されるときは wrap_long_line が行末のスペースを削除するが、この記法の場合はそれとは別で改行を入れているので、スペースが削除されていない。

```
// https://github.com/vlang/v/blob/9ee518681170a8d2ce8d2135e98933a9d8cd4090/vlib/v/fmt/fmt.v#L140-L142
if f.out.buf[f.out.buf.len - 1] == ` ` {
	f.out.go_back(1)
}
```

ということで、些かアドホックではあるが、単純にスペースを削除する処理を入れた。もっとも、こういうアドホックな実装が許されているからこういうバグが発生するのだが……。

[fmt: remove tail space when using multiline short arg #9110](https://github.com/vlang/v/pull/9110)
