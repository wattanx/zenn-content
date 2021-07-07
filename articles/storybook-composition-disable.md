---
title: "StorybookのPackage Compositionをdisableにしたい"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [storybook, chakraui]
published: true
---

# はじめに

CRA × Chakra UI × Storybook という環境で`npm run storybook`して画面を確認してみると、何か見慣れないもの出ていることに気が付きました。
現在の開発環境では不要なので消したいと思っていました。

![](https://storage.googleapis.com/zenn-user-upload/d2da93ff89d4bc79bd287c38.png)

# Storybook Composition

↑ の赤枠で出ている部分は、Storybook 6.0 から導入された`Storybook Composition`という機能です。
簡単に説明すると、ある Storybook の中から別の Storybook を参照することができる機能です。

https://storybook.js.org/docs/html/workflows/storybook-composition

上記 URL のドキュメントを見ると、`.storybook/main.js`に下記のように設定すると設定できると書いています。

```js:.storybook/main.js
module.exports={
  // your Storybook configuration
  refs: {
   'design-system': {
     title: "Storybook Design System",
     url: "https://5ccbc373887ca40020446347-yldsqjoxzb.chromatic.com"
   }
  }
}
```

そこで私は、自分のプロジェクトを確認したが上記のような設定はしていませんでした。
では、どこで Chakra UI の Composition の設定がされているのか。

# Package Composition

Composition にはもう一つ種類があり、`Package Composition`という機能があります。
こちらは参照先のパッケージの`package.json`に設定を書いておくことで、参照元の Storybook に自動的に読み込ませることができます。

https://storybook.js.org/docs/html/workflows/package-composition

今回のプロジェクトで参照している`@chakra-ui/react`の package.json を調べてみます。
調査すると、下記のソースを発見。これだ...

```json:package.json
"storybook": {
    "title": "Chakra UI",
    "url": "https://chakra-ui.netlify.app"
},
```

https://github.com/chakra-ui/chakra-ui/blob/main/packages/react/package.json

上記ドキュメントによると、`.storybook/main.js`に設定することで disable にすることができると書いているのでやってみます。

```js:.storybook/main.js
module.exports = {
  // 省略
  refs: {
    '@chakra-ui/react': { disable: true },
  },
```

そして実行！
![](https://storage.googleapis.com/zenn-user-upload/f3f5e31c30190936130da295.png)

無事 disable にできました！
