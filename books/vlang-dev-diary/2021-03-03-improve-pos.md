---
title: '2021/03/02 - pos を修正する地道で大事な仕事'
---

私が V言語で積極的に行っている活動の1つに、pos を正しくするというものがある。

最近の言語では当たり前になりつつあるが、V言語でもエラー表示の際にエラー箇所をアンダーラインで示す。

```
vlib/v/checker/tests/is_type_invalid.vv:18:5: error: `Cat` doesn't implement method `speak` of interface `Animal`
   16 |     }
   17 |     a := Animal(Dog{})
   18 |     if a is Cat {
      |        ~~~~~~~~
   19 |         println('not cool either')
   20 |     }
```

ところが、このアンダーラインが分かりづらかったり、間違っていたりすることが意外と多い。パーサが pos を正しく設定していなかったり、エラーを報告する際に適切ではない pos を渡していたりする。例えば、式のエラーのはずなのに右オペランドの pos を渡していたりするケースだ。

これによってコンパイラが動作しなくなったり、バグが発生したりすることはないため、見過ごされがちである。しかし、私はこれを直すことをかなり重視しており、見つけたものについては基本的にすべて修正している。

エラー表示のアンダーラインというのは、些細なように見えて、大事なことだ。確かに、修正したところでバグが直るわけではないし、ぶっちゃけだいたい表示されていればどうでもいいという考え方もある。たしかに、修正は簡単とはいえ別に楽しいバグ修正ではないし、直さなくてもコンパイラには影響はないし、いちいち調べて修正して PR を出すのも面倒ではあるだろう。

しかし、V言語が本当に実用言語を目指すのであれば、そういう部分を放置するべきではないと思う。エラー表示はユーザーに見える部分で、そこが明らかに間違えていると言語そのものの印象が悪くなる。

<!-- textlint-disable ja-technical-writing/no-doubled-joshi -->
また、エラー表示を間違えているということは、実装者がその部分を適当に書いたという可能性が浮上する。実際、pos が間違っていた部分の周辺に、明らかに適当に書いたと思わしき部分があり、別のバグを発見したこともある。何を実装していて、なぜそれをエラーにするのかを正しく理解して書いていれば、エラー表示の範囲を間違えるということはないだろう。
<!-- textlint-enable -->

- [parser: Improve position of mut receiver waning / error #8240](https://github.com/vlang/v/pull/8240)
- [checker: improve error position of infix expr #8828](https://github.com/vlang/v/pull/8828)
- [parser: improve anon fn pos #8210](https://github.com/vlang/v/pull/8210/files)
- [checker: correct pos for type error of `if v is interface` #9080](https://github.com/vlang/v/pull/9080)
- その他いろいろ
