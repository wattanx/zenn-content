---
title: "@nuxt/test-utils のススメ"
emoji: "🦔"
type: "tech"
topics: ["nuxt"]
published: false
---

## はじめに

Nuxt 3 がリリースされてから 1 年以上経ちましたが、みなさん Nuxt 3 でテストどうしてますか？

Auto Imports を使っているため テストの設定を追加したり、Nuxt が提供する Composables を使って `[nuxt] instance unavailable` というエラーが出て困っている人はいないですか？

こういった問題を解決するために、`@nuxt/test-utils` というライブラリを導入することをおすすめします。

## @nuxt/test-utils とは

Nuxt から提供されている Nuxt アプリケーションの Unit Test・E2E Test 用ライブラリです。

`@nuxt/test-utils` を使うことで Nuxt アプリケーションのテストを書きやすくなります。

以前は`nuxt-vitest`というライブラリとして開発されていて、`@nuxt/test-utils`に統合された機能も存在します。

## Unit Test

`useNuxtApp`などを使用したコンポーネントをテストすると、`[nuxt] instance unavailable`というエラーが出ます。

`useNuxtApp`が Nuxt の runtime に依存しているためです。

`@nuxt/test-utils`は Nuxt の runtime を必要とするコードをテストするための環境を提供しているので、このエラーを解消することができます。

:::message
現在は Vitest のみサポートしてます。
:::

## セットアップ

1. nuxt.config に `@nuxt/test-utils/module` を追加します。

これはオプショナルですが、module に設定することで Nuxt Dev Tools に統合され、開発中のテスト実行をサポートしてくれます。

```ts:nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@nuxt/test-utils/module"],
});
```

2. 以下のように`vitest.config`を修正します。

```ts:vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  // vitestのconfig
})
```

## Nuxt Runtime 環境でテストを実行する

`@nuxt/test-utils`をインストールしただけでは何も変わりません。
オプトインで Nuxt Runtime 環境を設定できるようになっています。

`.nuxt.`というファイル名をつけるか、`@vitest-environment nuxt`とコメントを書くことで Nuxt Runtime 環境でテストを実行できます。（`my-file.nuxt.test.ts` or `my-file.nuxt.spec.ts`）

```ts
// @vitest-environment nuxt
import { test } from "vitest";

test("my test", () => {
  // ... test with Nuxt environment!
});
```

すべてのファイルを Nuxt Runtime 環境で実行したい場合は vitest.config.ts を以下のように設定します。

```ts:vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
  }
})
```

- jsdom または happy-dom 環境で実行される
- テスト実行前に global な Nuxt アプリケーションがセットアップされる
  - なので、テスト中にグローバルな状態を変更しないように注意する必要がある。
  - 必要な場合はテスト後に初期化する必要がある。（他のテストに影響する可能性がある）

## Builin Mocks

以下のモックがビルトインで提供されています。

- `IntersectionObserver API`
- `IndexedDB API`

設定方法はこんな感じ

```ts:vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environmentOptions: {
      nuxt: {
        mock: {
          // default: true
          intersectionObserver: true,
          // default: false
          indexedDb: true,
        }
      }
    }
  }
})
```

## Helpers

テストを簡単に書けるようにするためのヘルパー関数が提供されています。

- `mountSuspended`

Nuxt アプリケーション内で任意のコンポーネントをマウントし、async setup や plugin の injection の使用を可能にします。

```ts:plugins/hello.ts
export default defineNuxtPlugin(() => {
  return {
    provide: {
      hello: (msg: string) => `Hello, ${msg}!`,
    },
  };
});
```

```vue:components/SomeComponent.vue
<script setup lang="ts">
defineProps<{ message: string }>();

const { $hello } = useNuxtApp();
</script>

<template>
  <div>
    <h1>{{ $hello(message) }}</h1>
  </div>
</template>
```

```vue:pages/index.vue
<template>
  <SomeComponent message="This is an auto-imported component" />
</template>
```

```ts:pages/index.nuxt.test.ts
import { test, expect } from "vitest";
import { mountSuspended } from "@nuxt/test-utils/runtime";
import IndexPage from "~/pages/index.vue";

test("should render SomeComponent", async () => {
  const wrapper = await mountSuspended(IndexPage);
  expect(wrapper.text()).toContain(
    "Hello, This is an auto-imported component!"
  );
});
```

- `mockNuxtImport`

Nuxt が提供する Composables や Auto Imports の関数をモックするためのヘルパー関数です。

使わない場合

```ts
vi.mock("#app/nuxt", async (importOriginal) => {
  const mod = await importOriginal()

  return {
    ...mod,
    useNuxtApp: () => ({
      ...
    })
  }
})
```

使う場合

```ts
import { mockNuxtImport } from "@nuxt/test-utils/runtime";

mockNuxtImport('useNuxtApp', () => {
  // factory 関数を返す
  return () => ({
    ...
  })
})
```

`mockNuxtImport`はマクロです。実際は、使わない場合のような形に変換されて実行されています。

- `registerEndpoint`

モックデータを返す Nitro エンドポイントを作成できます。API にリクエストしてデータを表示するコンポーネントをテストしたい場合に便利です。

外部の API もモック可能です。

ここまで紹介した `@nuxt/test-utils/runtime`の helper たちは、E2E テスト(`@nuxt/test-utils/e2e`)と同じファイルで使うことができません。

## E2E Test

`@nuxt/test-utils`は E2E テストもサポートしています。

## セットアップ

各`describe`のブロックに `@nuxt/test-utils/e2e` が提供するヘルパーを使ってテストコンテキストを作成する必要があります。

```ts:my-test.e2e.ts
import { describe, test } from "vitest";
import { setup, $fetch } from "@nuxt/test-utils/e2e";

describe("My test", async () => {
  await setup({
    // test context options
  });

  test("my test", () => {
    // ...
  });
});
```

`setup`は`beforeAll`, `beforeEach`, `afterEach`, `afterAll`でテストに必要な処理を実行します。
