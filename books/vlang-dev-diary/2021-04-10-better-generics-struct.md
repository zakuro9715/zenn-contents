---
title: '2021/04/10 - struct の generics が使えるようになってきた'
---

struct の generics がかなり改善されつつあり、とりあえず実装されていただけだったのが、ちゃんと便利に使える機能になってきた。

たとえば、ようやくレシーバで型引数が使えるようになった。できて当たり前に思えるかも知れないが、これまではレシーバには `Data<int>` のような具体的な型しか使えなかった。これでは実質的にメソッドが使えないようなもので、generics の struct が使い物にならない大きな原因だった。

ともあれ、この機能が実装されたことで、ようやく generics の struct を使う意味が出てきた。

- [generics: fix generics with generic struct receiver (fix #9473) #9658](https://github.com/vlang/v/pull/9658)

```v
struct Data<T> {
	v T
}

fn (v Data<T>) val<T>() T {
	return v.v
}

fn (v Data<T>) add<T>(n T) T {
	return v.v + n
}
```

また、戻り値にも型引数が使えるようになった。レシーバと同様、これまでは `Data<T>` をかえすようなことができなかった。

[generics: fix generics method return generics struct #9614](https://github.com/vlang/v/pull/9614)

```v
fn (v Data<T>) clone()<T>() Data<T> {
	return Data<T>{ v.n }
}
```

また、struct の省略記法も（知らないうちに）入っていたので、次のように簡単に書ける

```v
fn new_data<T>(n T) Data<T> {
	return { v.n }
}
```
