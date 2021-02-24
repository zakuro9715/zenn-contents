---
title: '2021/02/23 - strings.Builder が io.Writer を実装していない'
---

V言語において `io.Writer` はかなり忘れられた存在になっている。標準ライブラリの中で実装されていたのは `os.File` くらいだった。

そのかわりに、`write(string)` が定義されている構造体は多い。`io.Writer` のことを忘れていると、そうなるのは理解できる。

`strings.Builder` もこのパターンで、`write(string)` として定義されている。たださすがに `strings.Builder` が `io.Writer` を実装しないのはまずいというか、その方針で行くならばもはや `io.Writer` が存在している意味がない。ということで修正した。

`strings.Builder` の定義を修正するのは簡単だが、これはいろいろな場所で使われているため、そちらの修正が面倒だった。加えて、`io.Writer` は自動生成される C コードの中で使われてしまっているため、`wirte` のシグネチャをいきなり変えるとセルフコンパイルが通らなくなる。なので、`write_string` を追加し、`write` の使用箇所をこれに置き換える PR のあとに、`write` のシグネチャを変更する PR を作った。プログラミング言語を開発していると、セルフコンパイルを通すために一工夫必要な場面がときおり発生する。

- [all: add strings.Builder.write_string and use write_string instead of write #8892](https://github.com/vlang/v/pull/8892)
- [builder: implements io.Writer](https://github.com/vlang/v/pull/8914)

この例に限らず、V言語の標準ライブラリには色々と微妙な設計が多い。`gg` とか `gx` とかが vlib 直下にあるのだが、これが何なのか初見で分かる人はいるのだろうか。
