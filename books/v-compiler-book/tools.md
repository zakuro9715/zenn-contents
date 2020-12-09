---
title: "Tools"
---

# Tools

vコマンドの実装は [cmd/v](https://github.com/vlang/v/tree/weekly.2020.49.5/cmd/vhttps://github.com/vlang/v/tree/weekly.2020.49.5/cmd/v) です。

また、サブコマンドは [cmd/tools](https://github.com/vlang/v/tree/weekly.2020.49.5/cmd/tools) に実装されています。

ここでは、コンパイルに関係のあるコマンドのみを取り上げます

## コンパイル

```
if command in ['run', 'build', 'build-module'] || command.ends_with('.v') || os.exists(command) {
	// println('command')
	// println(prefs.path)
	builder.compile(command, prefs)
	return
}
```

となっており、単純に `v.builder.compile` を呼び出しています

`buider.compile` では、

```
mut b := new_builder(pref)
match pref.backend {
	.c { b.compile_c() }
	.js { b.compile_js() }
	.x64 { b.compile_x64() }
}
```

としてバックエンドごとにコンパイルメソッドを呼び出し、

```
if pref.is_test || pref.is_run {
	b.run_compiled_executable_and_exit()
}
```

v test あるいは v run で実行されている場合、コンパイル結果を実行します。

Cのコンパイルは単純で、パーサを呼び出し、チェッカを呼び出し、cgen で C のコードを生成したら、CCを呼び出すだけです。

パーサなどそれぞれの要素については後の章で解説します。

# repl

V言語には repl がついています。この実装はどうなっているのでしょうか。

repl の実装は [vrepl.v](https://github.com/vlang/v/blob/weekly.2020.49.5/cmd/tools/vrepl.v#L98-L301https://github.com/vlang/v/blob/weekly.2020.49.5/cmd/tools/vrepl.v#L98-L301) です。

読む限り、入力からソースコードを組み立て、`v -repl run` で実行しています。

`-repl` で `pref.is_repl` が true になります。これによって、unused variable のエラーなどを抑制しているようです。

この実装では、都度コンパイルが走っています。v repl は若干実行が遅いのですが、それが原因でしょう。
