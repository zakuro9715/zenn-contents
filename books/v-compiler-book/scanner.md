---
title: '字句解析'
---

# Token

[vlib/v/token](https://github.com/vlang/v/blob/weekly.2020.49.5/vlib/v/token/token.v) にトークンの実装があります。

Token 構造体は kind や pos の入ったよくあるもので、特筆すべきことは特にありません。

その他、優先順位や is_xxx などのユーティリティも定義されています。

# [Scanner](https://github.com/vlang/v/tree/weekly.2020.49.5/vlib/v/scanner)

## [Scanner 構造体](https://github.com/vlang/v/blob/weekly.2020.49.5/vlib/v/scanner/scanner.v#L19-L51)

フィールドがたくさんあります。V のコンパイラ実装は一つの構造体にフィールドを大量に持たせていることが多いです。慣れましょう。

多くのものは名前を読めば役割がわかりますが、いくつかわかりにくい略語が使われているものがあります。V の内部実装では独特の略語が使われることが多いです（そして特にコメントはありません）。

- line_nr: 何行目かを表しています。line '\n' '\r' でしょうか。
- nr_lines: line_nr は減ることがりますが、こちらは最大行数です。ただ、コードを見る限り特に使われていません
- is_vh: 改行文字をスキップしなくなります。vh が何かはよくわかりませんでしたが、これも使われていないようです。
- is_inter_start/end: V コンパイラでは interpolation を inter と略す傾向にあります。
