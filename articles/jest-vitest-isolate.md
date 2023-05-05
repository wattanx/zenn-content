---
title: "JestとVitestのisolateについて"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vitest", "jest"]
published: true
---

## はじめに

:::message
Vitest のバージョンは `0.31.0`時点での話です。
:::

現状 Vitest が Jest など他のテスティングフレームワークに比べて遅くなる場合があることがわかっています。
(確実に遅くなるとはいえない。が、私自身もテストの速度が遅くなったことを経験しています。)

https://github.com/vitest-dev/vitest/issues/579

:::message
`vitest run`で実行した場合の話であり、`watch`モードは高速です。
:::

また Vitest を実行する場合、`--single-thread`オプションをつけると速くなるということもわかっています。
(`0.29.0`以前は `--no-threads`)

https://twitter.com/youyuxi/status/1621299180261244928

公式 Docs にも最大３倍速くなることが記載されています。

https://vitest.dev/config/#threads

> WARNING
>
> This option is different from Jest's --runInBand. Vitest uses workers not only for running tests in parallel, but also to provide isolation. By disabling this option, your tests will run sequentially, but in the same global context, so you must provide isolation yourself.
>
> This might cause all sorts of issues, if you are relying on global state (frontend frameworks usually do) or your code relies on environment to be defined separately for each test. But can be a speed boost for your tests (up to 3 times faster), that don't necessarily rely on global state or can easily bypass that.

`--single-thread`オプションをつけるとシングルスレッドで実行されるだけでなく、`isolate`もされなくなることが記載されています。

`isolate`されなくなるとはどういうことなのでしょうか。

## isolate について

Jest や Vitest における`isolate`は**環境の分離**です。

Jest や Vitest ではテストの並列化に [Worker Threads](https://nodejs.org/api/worker_threads.html)を使用しています。
(Jest では`jest-worker`というライブラリを通して利用している。)

テストを並列化しただけだと、下記のようなテストは状態を共有し他のテストに影響を与えてしまいます。
(test1.spec.ts の`globalThis.foo`への代入が他のテストにも影響する)

```ts:test1.spec.ts
describe('test 1', () => {
  it('foo', () => {
    const g = globalThis;
    g.foo = 'foo';
    expect(g.foo).toBe('foo');
  });
});
```

```ts:test2.spec.ts
describe('test 2', () => {
  it('not foo', () => {
    const g = globalThis;
    expect(g.foo).toBeUndefined();
  });
});
```

他のテストに影響するのを避けるには、それぞれのテストが実行される環境を分離する必要があります。

### Jest の isolate

Jest では環境の分離に Node.js の [vm](https://nodejs.org/api/vm.html)を使用しています。
vm を使うことで別々の環境でテストを実行することが可能になります。

### Vitest の isolate

Vitest では **isolate にも Worker Threads** を使っています。
(Jest 同様、並列でテストを走らせるために Worker Threads を使っている)

Worker Threads を `isolate` にも使っているので、`--single-thread`オプションをつけると、`isolate` もされなくなります。

`--single-thread`で下記のテストを実行すると、`test1.spec.ts`が先に実行されるとテストが落ちることが確認できます。

```ts:test1.spec.ts
describe('test 1', () => {
  it('foo', () => {
    const g = globalThis;
    g.foo = 'foo';
    expect(g.foo).toBe('foo');
  });
});
```

```ts:test2.spec.ts
describe('test 2', () => {
  it('not foo', () => {
    const g = globalThis;
    expect(g.foo).toBeUndefined();
  });
});
```

(↓ の stackblitz でも確認できます)
https://stackblitz.com/edit/vitest-dev-vitest-sksonx

なぜ`vm`で `isolate` をしていないのか、詳しい理由は理解できていませんが、`vm が ESM に完全に対応していないため使えない`といったコメントを Issue や Discord で見たことがあります。

また、Vitest では tinypool の`isokateWorker`オプションを使っているので、Worker を作り直して環境を分離していると思われます。
https://github.com/tinylibs/tinypool

## Vitest でテストが遅くなる場合どうすればよいか

Vitest に移行してテストが遅くなってしまった場合、これまで下記のようなアプローチをとってきました。

### `--single-thread`で実行する

まずは `--single-thread`で実行できるか確認します。
他のテストに影響を与えるようなテストが書かれていなければ、`--single-thread`で高速化できます。
(`isolate`されなくなるため、注意が必要)

### `shared`オプションをつけてテストを並列実行する

GitHub Actions や Circle CI で実行している場合は、`shared`オプションと matrix を使ってテストを並列実行することができます。

https://vitest.dev/guide/cli.html#shard

下記は GitHub Actions で並列実行する場合の例です。

```yml:.github/workflows/vitest.yml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1/4, 2/4, 3/4, 4/4]

    steps:
      - name: run vitest
        run: |
          npx vitest run --shard=${{ matrix.shard }}
```

ただし、GitHub Actions の場合、並列実行したとしてもそれぞれの job の実行時間の合計で課金されるので注意が必要です。
(4 並列でそれぞれ 3 分かかる場合、12 分ぶん課金される。)

## 最後に

公式ドキュメントを見ただけでは、`isolate`が何をしているのか全然理解できませんでしたが、
小さいテスティングフレームワークを実際に作ってみることで、なんとなくわかってきました。

小さいテスティングフレームワークを作成する際に、下記の記事を参考にしました。

https://cpojer.net/posts/building-a-javascript-testing-framework

Vitest に移行しても速くならない場合もありますが、watch モードは高速やし ESM 周りのストレスがないので、Vitest に移行してみる価値はあると思っています。
