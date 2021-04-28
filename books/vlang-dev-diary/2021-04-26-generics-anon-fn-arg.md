---
title: '2021/04-26 - 型引数を使った関数型引数が使えるようになった'
---

タイトルはちょっとわかりにくいが、具体的にはこういうやつが動くようになった。

```
fn reduce<T>(arr []T, f fn (a T, b T) T, init T) T {
	mut acc := init
	for v in arr {
		acc = f(acc, v)
	}
	return acc
}

println(reduce([0, 1, 2], fn (a int, b int) int {
	return a + b
}, 0))
```

もちろんメソッドでも使える

```
struct MyStruct<T> {
	arr []T
}

fn (mut s MyStruct<T>) get_data(pos int) T {
	return s.arr[pos]
}

fn (mut s MyStruct<T>) iterate(handler fn (T) int) int {
	mut sum := 0
	mut i := 0
	for {
		k := s.get_data<T>(i)
		sum += handler(k)
		i++
		if i > 4 {
			break
		}
	}
	return sum
}

pub fn consume(data int) int {
	return data
}

fn test_generics_with_anon_generics_fn() {
	mut s := MyStruct<int>{
		arr: [1, 2, 3, 4, 5]
	}
	y := s.iterate<int>(consume)
	println(y)
	assert y == 15
}
```

- [checker: fix generics with anon generics fn argument (fix #9859)](https://github.com/vlang/v/pull/9897)

私が V言語の開発に参加し始めてから半年も立っていないが、こうしてどんどんと便利になっている。特に generics 周りは当初全く使い物にならないレベルだったのが、それなりに便利に使える機能になりつつある。

この開発スピードで便利になるなら素直に将来が楽しみだ。

未実装の generics で欲しいものだと、Alias と Sum Type の generics がある。これもそのうち実装されると思いたい。

```
type Array<T> = T[]
type Data<T> = Foo<T> | Bar<T>
```
