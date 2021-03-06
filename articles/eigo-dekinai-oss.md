---
title: '英語ができない人のための OSS コントリビューション入門'
---

誰にとっても最初はハードルが高い OSS へのコントリビューションですが、特に英語が苦手な人にとってはとてもとても高いハードルを感じるものです。そもそもなんて書けばいいのかわからない、PR は作ったが通じなくて返事もなく close されたらどうしようという不安。大変よくわかります。

私の英語力は壊滅的です。高専の英語の授業は寝ていた記憶しかなく、客観的に見ると、私の英語力は良くて中学卒業レベルといったところでしょう。少なくとも、私の PR を見れば英語が下手なのは明らかでしょう。

- [cgen: use overloaded eq op in auto eq method](https://github.com/vlang/v/pull/9475)
    > tests added this PR fails on current master because orverloaded eq method is not used by auto eq method.
- [cgen: auto eq method for sumtype](https://github.com/vlang/v/pull/9408)
    > and sum type declaration are very similar syntax.
    >
    > And alias has auto eq methods, but sum type don't have it. This is not intuitive behavior imho.

そんな私ですが、V言語に定期的にコントリビュートし、Organization メンバーになることができました。その経験から、英語ができなくても存外何とかなるということや、下手なりにどうすればいいのかをまとめてみました。

## 1. なんとかなるという強い気持ちを持つ

いきなりの精神論ですが、これが最も大切であり、これさえできれば問題の 70% は解決したといっていいでしょう。私を含め、英語ができない人はいわば英語に過剰な恐怖心を抱いていることがあります。しかし、それさえ乗り切れればたいていのことは何とかなります。

実際のところ、英語ができなかったとしても、英語で伝えるというのは可能なのです。単語は調べれば何とかなりますし、文法も前置詞もすべて忘れてとりあえず単語を並べておくだけでもある程度伝わります。

## 文章を短く、難しいことは考えない

複雑な説明をするのは難しいので、`this code has bug. expected A, but got B. I fixed this.` のような短い文の羅列にしてしまうのが良いです。難しい文法のことは忘れましょう。冠詞がなくても、複数形が単数形になっていても問題なく伝わります。

## 翻訳ツールを活用する

DeepL を使いましょう。リーディングは DeepL さえあれば困ることはほとんどありません。

## 2. バグフィックスをメインにする

機能提案ではどうしても議論が必要になります。さすがに単語を並べるだけで機能についての議論を行うのは無理があるでしょう。なので、バグフィックスをメインにコントリビューションするのがおすすめです。バグフィックスであれば、再現コードとテストを追加すれば、ほとんど説明がなくても理解してもらうことができます。

## 3. 一つの PR は可能な限り小さく

変更は意識して小さくしましょう。内容が大きくなるほど説明するべきことも増えます。
