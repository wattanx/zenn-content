---
title: "Nuxt 2025 リリースまとめ"
emoji: "⛰️"
type: "tech"
topics: ["nuxt", "vue"]
published: false
publication_name: "comm_vue_nuxt"
---

この記事では 2025 年にリリースされた Nuxt の主要なアップデートについてまとめています。

## Nuxt 3.16

https://nuxt.com/blog/v3-16

- `npm create nuxt` で新しい Nuxt プロジェクトを作成可能に
  - 今までは `npx nuxi init` を使う必要があった
- `unhead` v2 へのアップグレード
- Nuxt DevTools v2 へのアップグレード
  - 今後の開発をよりスムーズに進めるために必要な破壊的変更を含んだアップデートを含む
- 様々なパフォーマンス改善
  - [roe.dev](https://github.com/danielroe/roe.dev) は v3.16 で 32% 高速化し、[nuxt.com](https://github.com/nuxt/nuxt.com) は 28% 高速化した
- delayed/lazy hydration のサポート
  - Vue に組み込まれている Hydration 戦略を使っている。
    - https://ja.vuejs.org/guide/components/async#lazy-hydration
- ページとしてスキャンするファイルを細かく制御できるようになった
  - `nuxt.config.ts` の `pages.pattern` で設定可能
- `debug` オプションがより柔軟になった
- decorator のサポート (experimental)
- Layer の namespace が自動で作成されるようになった
- Error Handling の改善
- `addTypeTemplate` に nitro の型情報の拡張かどうかを指定するオプションが追加された
- Nitro v2.11 へのアップグレード
- 依存している unjs ecosystem のメジャーアップデート

## Nuxt 3.17

- Data Fetching Improvements
  - `useFetch` と `useAsyncData` の改善
  - 同じ key の場合、`data` や `status` を共有するようになった
  - Reactive な key のサポート
  - Data Refeching の改善
- `NuxtTime` コンポーネントの追加
- `NuxtErrorBoundary` コンポーネントの改善
- `NuxtLink` に `trailingSlash` prop が追加された
- `NuxtLoadingIndicator` コンポーネントの props に `hideDelay` と `resetDelay` が追加された
- ドキュメントが `@nuxt/docs` としてパッケージ化された
- 開発者体験向上
- モジュール開発の強化
- パフォーマンス向上

## Nuxt 3.18

- Nuxt 4 に含まれる機能の backport
- Lazy Hydration Macros の追加
  - `defineLazyHydrationComponent` マクロによってより書きやすくなった。
- `NuxtRouteAnnouncer` がデフォルトで追加されるようになった。(`app.vue` を利用してない場合)
- Chrome DevTools workspace integration が追加された
  - Chrome DevTools で Nuxt のソースコードを直接編集できるようになった
- Component の型安全性の向上
  - `ClientOnly`, `DevOnly`, `NuxtTime`, の型定義が改善された
- `onWatcherCleanup` が auto import できるようになった
- Observability の強化
- パフォーマンス向上
- いくつかのバグ修正

## Nuxt 3.19
