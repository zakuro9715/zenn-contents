---
title     : JavaScript で range 配列を作る方法は結局どれが一番いいのか
type      : tech
emoji     : 🗿
topics    : [javascript]
published : true
---

太古の昔から存在する議論。

連番配列の作成、つまり `n` から `[0, 1, 2, ...]` を作る方法はどれがベストか。

歴史上、数多のヘルパー関数が作られてきたであろうことは想像に難くない。

## 結論

素直に `for` で作る。

```javascript
const arr = [];
for (let i = 0; i < len; i++) {
  arr.push(i);
}
```

```javascript
const arr = Array(len);
for (let i = 0; i < len; i++) {
  arr[i] = i;
}
```

### 理由

- 単純
- 読みやすい
- 速い

ワンライナーで書くやり方はどれもやや癖があり、速くもない。


### `[...Array(length).keys()]` について

よく見かけるやり方。

```javascript
[...Array(len).keys()]
```

比較的浸透しているが、速くはない。読みやすくもないと思う。

## ベンチマーク

- サイズ指定で作成して `for` で代入するのが一番早い
- `fill` して `map` が意外と早い。
- `Array.from` は遅い。
- とはいえ 20x 程度で、大量に繰り返すものでもないので、全部誤差の範囲とも。

```
BENCH  Summary

for { arr[i]=i } - array.bench.ts > range
    2.66x faster than for { arr.push(i) }
    4.97x faster than Array.fill.map
    10.17x faster than [...Array.keys()]
    15.49x faster than Array.from(Array)
    20.98x faster than Array.from({ length })
```

:::details 詳細

```typescript
import { describe, bench } from 'vitest';
const len = 10000;

describe('range', () => {
  bench('Array.from(Array)', () => {
    Array.from(Array(len), (_, i) => i);
  });
  bench('Array.from({ length })', () => {
    Array.from({ length: len }, (_, i) => i);
  });
  bench('[...Array.keys()]', () => {
    [...Array(len).keys()];
  });
  bench('Array.fill.map', () => {
    Array(len).fill(0).map((_, i) => i)
  })
  bench('for { arr.push(i) }', () => {
    let arr = [];
    for (let i = 0; i < len; i++) {
      arr.push(i);
    }
  });
  bench('for { arr[i]=i }', () => {
    let arr = Array(len);
    for (let i = 0; i < len; i++) {
      arr[i] = i;
    }
  });
});
```

```
 RERUN  array.bench.ts x5


 ✓ array.bench.ts > range 3693ms
     name                           hz     min     max    mean     p75     p99    p995    p999     rme  samples
   · Array.from(Array)        3,099.94  0.2300  7.4550  0.3226  0.3450  1.0200  1.6050  2.3950  ±3.80%     1550
   · Array.from({ length })   2,288.35  0.3950  0.8950  0.4370  0.4300  0.7400  0.7850  0.8900  ±0.85%     1145   slowest
   · [...Array.keys()]        4,719.29  0.1750  4.7400  0.2119  0.2000  0.4800  0.5550  1.0300  ±2.13%     2360
   · Array.fill.map           9,654.65  0.0850  0.5750  0.1036  0.1050  0.2150  0.2600  0.3700  ±0.61%     4828
   · for { arr.push(i) }     18,078.73  0.0400  3.2700  0.0553  0.0550  0.1300  0.1500  0.3550  ±1.48%     9040
   · for { arr[i]=i }        48,016.63  0.0150  0.2150  0.0208  0.0200  0.0500  0.0600  0.0900  ±0.40%    24010   fastest

 BENCH  Summary

  for { arr[i]=i } - array.bench.ts > range
    2.66x faster than for { arr.push(i) }
    4.97x faster than Array.fill.map
    10.17x faster than [...Array.keys()]
    15.49x faster than Array.from(Array)
    20.98x faster than Array.from({ length })
```

:::

@[stackblitz](https://stackblitz.com/edit/array-bench?embed=1&file=array.bench.ts&view=editor)

## その他

[一応 range の proposal があり、stage2 になっている。](https://github.com/tc39/proposal-iterator.range)
