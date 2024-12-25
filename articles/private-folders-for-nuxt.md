---
title: 'Nuxt で Next.js の Private folders のような機能を実現する方法'
emoji: '⛰️'
type: 'tech'
topics: ['nuxt', 'vue']
published: true
publication_name: 'comm_vue_nuxt'
---

## はじめに

[Next.js](https://nextjs.org/) には [Private folders](https://nextjs.org/docs/app/getting-started/project-structure#private-folders) という機能があります。
Next.js の App Router では、以下のように `_` で始まる名前にすることでルーティングから除外されます。
(基本的に `page.js` や `route.js` でないとルートとして認識されないようにはなっているので、Private folders を使わなくても以下の例では除外されている)

```sh
pages/
├─ dashboard/
│  └─ _components
│     └─ button.tsx # ルーティングから除外される
└─ page.tsx # /dashboard はルートになる
```

[Nuxt](https://nuxt.com/) には同じような機能はありませんが、とある機能を使うことで同じようなことが実現できます。

## Nuxt のルーティングについて

Nuxt は File-based routing をサポートしており、`pages/` にファイルを置くことでルートが自動で生成されます。

`pages` には `.vue` ファイル以外も置くことができるので、そのままでは全てのファイルがルートになります。

```sh
pages/
├─ _utils
│  └─ format.ts # /_utils/format はルートになる
└─ index.vue # / はルートになる
```

Nuxt のルーティングの仕組みを以下の環境を使って簡単に説明します。

https://stackblitz.com/edit/github-kmppwajf

`nuxt dev` を実行した状態で `/_vfs/%23build%2Froutes.mjs` にアクセスすると以下のようなソースコードが表示されます。

```ts
import { default as formatQK6KF9EiXFMeta } from '/home/projects/github-kmppwajf/pages/_utils/format.ts?macro=true';
import { default as indexk4zboVR8EKMeta } from '/home/projects/github-kmppwajf/pages/index.vue?macro=true';
export default [
  {
    name: '_utils-format',
    path: '/_utils/format',
    component: () => import('/home/projects/github-kmppwajf/pages/_utils/format.ts'),
  },
  {
    name: 'index',
    path: '/',
    component: () => import('/home/projects/github-kmppwajf/pages/index.vue'),
  },
];
```

Nuxt はこのようなルーティングの設定を自動で生成しており、この設定をベースにルーティングを行っています。

実際にどのようにこの設定が使われているのかについては [Nuxt はどうやって file-based routing を実現しているか](https://zenn.dev/comm_vue_nuxt/articles/62b8dea3a018f9) を見てください。

## Lifecycle Hooks を使って Private folders のような機能を実現する

Private folders を実現するために使う`とある機能`とは [Lifecycle Hooks](https://nuxt.com/docs/api/advanced/hooks) です。
Nuxt ではこの Lifecycle Hooks を使ってビルド時や実行時に処理を追加することができます。
(余談: Lifecycle Hooks は https://github.com/unjs/hookable というライブラリをベースに作られています。)

今回使用する Lifecycle Hooks は `pages:extend` です。
`pages:extend` を使うことで、自動生成されるルーティングの設定を変更することができます。

`/_` で始まるパスをルーティングから除外するために以下のようなコードを追加します。
(Lifecycle hooks は以下のように `nuxt.config` に追加することができます。)

```ts:nuxt.config.ts
import { defineNuxtConfig } from 'nuxt/config';
import type { NuxtPage } from 'nuxt/schema';

export default defineNuxtConfig({
  hooks: {
    'pages:extend': (pages) => {
      const pagesToRemove: NuxtPage[] = [];
      pages.forEach((page) => {
        if (/\/_[^/]+/.test(page.path)) {
          pagesToRemove.push(page);
        }
      });
      pagesToRemove.forEach((page) => {
        pages.splice(pages.indexOf(page), 1);
      });
    },
  },
});
```

:::message
元の配列を変更しているのは意図的です。
:::

これだけのコードで Private folders のような機能を実現することができます。
処理の内容としては `/_` で始まるパスを配列から削除しているだけです。

`pages:extend` の callback 引数である配列がルーティングの設定を生成する際に使われるので、この配列から削除することでルーティングから除外することができます。

実際にルーティングの設定がどう変わっているのか以下の環境で見てましょう。

https://stackblitz.com/edit/github-kmppwajf-hjiqtys2

`nuxt dev` を実行した状態で `/_vfs/%23build%2Froutes.mjs` にアクセスすると以下のようなソースコードが表示されます。

```ts
import { default as indexVhp30nypx0Meta } from '/home/projects/github-kmppwajf-hjiqtys2/pages/index.vue?macro=true';
export default [
  {
    name: 'index',
    path: '/',
    component: () => import('/home/projects/github-kmppwajf-hjiqtys2/pages/index.vue'),
  },
];
```

`pages:extend` を使わない場合は`/_utils/format` がルートとして認識されているので `/_utils/format` に遷移すると 500 エラーになります。

`pages:extend` を使うことで`/_utils/format` がルートとして認識されないので 404 エラーになります。

:::details .nuxtignore は使えないのか
Nuxt には特定のファイルをビルド時に無視する仕組みとして `.nuxtignore` があります。
https://nuxt.com/docs/guide/directory-structure/nuxtignore

ルーティングの設定はビルド時に生成されるため、`.nuxtignore` を使ってビルド時に無視することでルーティングの設定から除外することができると考えていました。

`.nuxtignore` を使うことでルーティングの設定から除外することはできましたが、hmr が動かなくなるという問題があったので、Lifecycle Hooks を使うこと方法を紹介しています。
(`.nuxtignore` や `.gitignore` に設定したファイルは watcher から除外されているため hmr が動かなくなります)

:::

## さいごに

`pages:extend` を使うことで Nuxt でも Next.js の Private folders のような機能を実現することができることを紹介しました。

便利な一方、 `components` などの auto imports が使えなくなるので auto imports を使っている場合は注意が必要です。
(設定することで Private folders でも auto imports には対応できます)
