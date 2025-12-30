---
title: "Nuxt 2025 リリースまとめ"
emoji: "⛰️"
type: "tech"
topics: ["nuxt", "vue"]
published: true
publication_name: "comm_vue_nuxt"
---

この記事では 2025 年にリリースされた Nuxt の主要なアップデートについてまとめています。

## Nuxt 3.16

https://nuxt.com/blog/v3-16

### A New New Nuxt

`npm create nuxt`で新しい Nuxt プロジェクトを作成可能になりました。
今まで使っていた`npx nuxi init`の軽量版です。
Nuxt が提供する Module なども簡単に追加できるようになっています。

### Unhead v2

Nuxt では`<head>`の管理に [unhead](https://unhead.unjs.io/) を使っていて v2 にアップデートされました。(Opt-in で利用可能に)

v2 では、非推奨の機能が削除されていますが、Nuxt 3 では後方互換性が維持されているためほとんどのユーザーは何も変更する必要はありません。

ただし、`@unhead/vue`から直接 import してる場合は、auto import または`#app/composables/head`(`#imports`)からインポートするように変更する必要があります。
`compatibilityVersion: 4`を設定している場合、追加の変更については[アップグレードガイド](https://nuxt.com/docs/4.x/getting-started/upgrade#unhead-v2)を参照してください。

### Delayed Hydration Support

Vue に組み込まれている Hydration 戦略を使って delayed/lazy hydration のサポートが追加されました。
https://ja.vuejs.org/guide/components/async#lazy-hydration

以下の例は Nuxt のドキュメントより引用しています。

```vue
<template>
  <!-- Hydrate when component becomes visible in viewport -->
  <LazyExpensiveComponent hydrate-on-visible />

  <!-- Hydrate when browser is idle -->
  <LazyHeavyComponent hydrate-on-idle />

  <!-- Hydrate on interaction (mouseover in this case) -->
  <LazyDropdown hydrate-on-interaction="mouseover" />

  <!-- Hydrate when media query matches -->
  <LazyMobileMenu hydrate-on-media-query="(max-width: 768px)" />

  <!-- Hydrate after a specific delay in milliseconds -->
  <LazyFooter :hydrate-after="2000" />
</template>
```

### その他の変更点

- DevTools v2 へのアップグレード
- 様々なパフォーマンス改善
  - [roe.dev](https://github.com/danielroe/roe.dev) は v3.16 で 32% 高速化し、[nuxt.com](https://github.com/nuxt/nuxt.com) は 28% 高速化した
- ページとしてスキャンするファイルを細かく制御できるようになった
  - `nuxt.config.ts`の`pages.pattern`で設定可能
- `debug`オプションがより柔軟になった
- decorator のサポート (experimental)
- Layer の namespace が自動で作成されるようになった
- Error Handling の改善
- `addTypeTemplate`に nitro の型情報の拡張かどうかを指定するオプションが追加された
- Nitro v2.11 へのアップグレード
- 依存している unjs ecosystem のメジャーアップデート

## Nuxt 3.17

https://nuxt.com/blog/v3-17

### Data Fetching Improvements

#### Consistent Data Across Components

同じ key の場合、`data`や`status`を共有するようになりました。

```vue:ComponentA.vue
<script setup>
const { data: users, pending } = useAsyncData("users", fetchUsers);
</script>
```

```vue:ComponentB.vue
<script setup>
// ComponentA.vue と同じ data を共有する
const { data: users, status } = useAsyncData("users", fetchUsers);
</script>
```

同じ key であれば`useAsyncData`の内部で使われているインスタンスが同じになるため、以下のような問題が発生するような場合があります。

https://zenn.dev/wattanx/scraps/ef9a3cf2b0d706

#### Reactive Keys

`useAsyncData`の key に Reactive な key を使えるようになりました。

```ts
const userId = ref("123");
const { data: user } = useAsyncData(
  computed(() => `user-${userId.value}`),
  () => fetchUser(userId.value)
);

// userId を変更すると自動的に新しいデータフェッチがトリガーされる
userId.value = "456";
```

#### Optimized Data Refetching

複数のコンポーネントが同じデータソースを監視している場合、依存関係が変更されると一度だけデータフェッチがトリガーされるようになりました。

```ts
// `route.query.page` が変更されると、一度だけデータフェッチが行われ、すべてのコンポーネントが同時に更新されるようになった
const { data } = useAsyncData(
  "users",
  () => $fetch(`/api/users?page=${route.query.page}`),
  { watch: [() => route.query.page] }
);
```

### Built-In Nuxt Components

- `NuxtTime`コンポーネントの追加
- `NuxtErrorBoundary`コンポーネントの改善
- `NuxtLink`に`trailingSlash`prop が追加
- `NuxtLoadingIndicator`コンポーネントの props に`hideDelay`と`resetDelay`が追加

### その他の変更点

- ドキュメントが`@nuxt/docs`としてパッケージ化された
- 開発者体験向上
- モジュール開発の強化
- パフォーマンス向上

## Nuxt 3.18

https://nuxt.com/blog/v3-18

### Lazy Hydration Macros

Nuxt 3.16 で delayed/lazy hydration のサポートが追加されましたが、Auto Import のコンポーネントのみで利用可能でした。
`defineLazyHydrationComponent`マクロによって明示的に import したコンポーネントでも利用できるようになりました。

### Accessibility Improvements

`NuxtRouteAnnouncer`コンポーネントがデフォルトで追加されるようになりました。
この変更は`app.vue`を利用していない場合にのみ適用されます。`app.vue`を利用してる場合は引き続き手動で追加する必要があります。

### その他の変更点

- Nuxt 4 に含まれる機能の backport
- Chrome DevTools workspace integration が追加された
  - Chrome DevTools で Nuxt のソースコードを直接編集できるようになった
- コンポーネントの型安全性の向上
  - `ClientOnly`,`DevOnly`,`NuxtTime`の型定義が改善された
- `onWatcherCleanup`が Auto Import できるようになった
- Observability の強化
- パフォーマンス向上
- いくつかのバグ修正

## Nuxt 4.0

https://nuxt.com/blog/v4

### New Project Structure

`app/`ディレクトリが導入されました。

これによって file watcher の動作が高速に（特に Windows や Linux で顕著）。
また、IDE がクライアントコードとサーバーコードのどちらを扱っているのか、より的確に把握できるようになりました。

```
my-nuxt-app/
├─ app/
│  ├─ assets/
│  ├─ components/
│  ├─ composables/
│  ├─ layouts/
│  ├─ middleware/
│  ├─ pages/
│  ├─ plugins/
│  ├─ utils/
│  ├─ app.vue
│  ├─ app.config.ts
│  └─ error.vue
├─ content/
├─ public/
├─ shared/
├─ server/
└─ nuxt.config.ts
```

### Smarter Data Fetching

`useFetch`と`useAsyncData`に以下のような改善が加えられました。

- 同じキーを使用する複数のコンポーネントが自動的にデータを共有するようになった。(Nuxt 3.17 で導入された機能)
- コンポーネントのアンマウント時に自動的にクリーンアップが行われ、必要に応じてリアクティブキーを使用してデータを再取得できるようになった。(Nuxt 3.17 で導入された機能)
- さらに、キャッシュされたデータの使用タイミングをより細かく制御できるようになった

### Better TypeScript experience

Nuxt は、アプリコード、サーバーコード、shared/ フォルダー、ビルダーコードのために別々の TypeScript プロジェクトを作成するようになりました

これにより、型推論がより正確になり、異なるコンテキストで作業している際の混乱を招くエラーが減少します。

## Nuxt 4.1

https://nuxt.com/blog/v4-1

### Enhanced Chunk Stability

デフォルトでは、Vite ビルドで出力される JS チャンクはハッシュ化されており、効率よくキャッシュできます。
しかし、単一のコンポーネントの変更がすべてのハッシュを無効にし、404 エラーの可能性を大幅に高める可能性があります。

例えば https://github.com/nuxt/nuxt/issues/26565 のような問題があります。

以下のように import maps を導入することで、小さな変更が加えられた場合にビルドの大部分が無効になることを防ぎます。

<!-- prettier-ignore -->
```html
<!-- Automatically injected import map -->
<script type="importmap">{"imports":{"#entry":"/_nuxt/DC5HVSK5.js"}}</script>
```

この機能を使わない場合は以下のようになります。

```ts
// chunk-A.js (ハッシュ: ABC123)
import "./entry-XYZ789.js";

// エントリーが変更されると：
// chunk-A.js (ハッシュ: DEF456) ← 内容は変わっていないのにハッシュが変わる
import "./entry-NEW111.js";
```

この機能を使うと以下のようになります。

```ts
// After（機能あり）:
// chunk-A.js (ハッシュ: ABC123)
import "#entry";

// エントリーが変更されても：
// chunk-A.js (ハッシュ: ABC123) ← ハッシュは変わらない！
import "#entry";
```

ネイティブのインポートマップサポートが必要ですが、`vite.build.target`をインポートマップをサポートしていないブラウザに含めるように設定している場合、Nuxt は自動的に無効にします。

`experimental.entryImportMap`オプションを使用して手動で有効または無効にすることもできます。

### Experimental Rolldown Support

`rolldown-vite`を使ってビルドできるようになりました。(まだ experimental)

`vite`を`rolldown-vite`に override することで利用できます。

```json:npm の場合
{
  "overrides": {
    "vite": "npm:rolldown-vite@latest"
  }
}
```

### その他の変更点

4.1 の変更は 3.19 にも backport されています。

- Improved Lazy Hydration
  - `defineLazyHydrationComponent`マクロが auto import が OFF の場合でも利用できるようになった
- Enhanced Page Rules
  - `experimental.inlineRouteRules`が有効な場合、`NuxtPage`オブジェクトに`rules`プロパティが追加されるようになった
  - この変更により module からアクセスしやすくなった
- Module Dependencies and Integration
  - `moduleDependencies`オプションによって、他のモジュールのオプションを上書きしたりできるようになった
- Module Lifecycle Hooks
  - `onInstall`,`onUpgrade`が利用できるようになった
- Enhanced File Resolution
  - `resolveFiles`に`ignore`オプションが追加された
- Simplified Kit Utilities
  - `addServerImports`が配列以外も受け入れられるようになった
- いくつかのバグ修正とパフォーマンス改善

## Nuxt 4.2

https://nuxt.com/blog/v4-2

### Abort Control for Data Fetching

`useAsyncData`の中で`AbortController`signal を使えるようになりました。
この変更により、きめ細かなデータフェッチの中止制御が可能になります。

以下の例は Nuxt のドキュメントより引用しています。

<!-- prettier-ignore -->
```vue
<script setup lang="ts">
const { data, error, clear, refresh } = await useAsyncData('users', (_nuxtApp, { signal }) => $fetch('/api/users', {
  signal
}))

refresh() // will actually cancel the $fetch request (if dedupe: cancel)

clear() // will cancel the latest pending handler
</script>
```

### Better Error Pages in Development

Stack Trace とカスタムエラーページも error overlay に表示されるようになりました。

### Opt-in Vite Environment API

`experimental.viteEnvironmentApi`を有効にすることで Vite Environment API をオプトインできるようになりました。
dev server が単一になり、パフォーマンスの改善などが期待できます。

### Async Data Handler Extraction

`useAsyncData`,`useLazyAsyncData`の中に書いた処理を自動で別の chunk に分離できるようになりました。
特に Static Generation のときに効果的で、Static Generation の場合ビルドのときにしか必要ないので client bundle から除外されます。

この機能は`experimental.extractAsyncDataHandlers`を有効にすることで利用できます。

### Experimental TypeScript Plugin Support

`@dxup/nuxt`module を使って TypeScript の開発体験を向上できるようになりました。
このモジュールは、Nuxt 固有の機能を使用する際の体験を向上させることを目的とした多数の TypeScript プラグインを追加します。

例えば以下のような機能があります。

- Auto Import されたコンポーネントファイルの名前を変更すると、すべての参照が自動的に更新されます
- import(`~/assets/${name}.webp`)のようなグロブパターンを使用する際に、ファイルに直接移動できます
- データフェッチ関数（`$fetch`、`useFetch`、`useLazyFetch`）からサーバールートハンドラーにジャンプします
- ランタイム構成プロパティで定義への移動がシームレスに機能します
- 自動インポートされたコンポーザブルとユーティリティのナビゲーションを改善する`@dxup/unimport`プラグインが含まれています
