---
title: '2021/03/19 - `f(opt() or { value })` ができるようになっていた。'
---

V言語では最近まで引数に optional を使うことができず、一時変数を作る必要がありました。これは大変不便だった。

```
fn opt() ?int {
    return 0
}

fn f() ? {
    // これはだめ
    println(opt() or { 1 })
    println(opt() or { return })
    println(opt() ?)

    // このように一時変数に入れる
    a := opt() or { 1 }
    println(a)
    b := opt() or { return }
    println(b)
    c := opt() ?
    println(c)
}
```

ところが、私が気づかないうちにこの問題は解決されていたようだ。

- [checker/cgen: support return from nested or #8430](https://github.com/vlang/v/pull/8430)
- [checker: support nested propagation cases f(g() ?) #8447](https://github.com/vlang/v/pull/8447)

しばしばこのような便利な変更がアナウンス無しで発生するので、変更をしっかりと追いかけないと見逃してしまう。
