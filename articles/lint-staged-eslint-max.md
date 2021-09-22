---
title: "lint-staged × ESLintの--max-warnings=0をつけてコミットできなくなったときの対処法" # 記事のタイトル
emoji: "🐱"
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["lintstaged", "eslint", "husky"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

husky と lint-staged を使ってコミット前に ESLint を走らせるようにした。
しかし、ESLint のオプション`--max-warnings=0`がついているかつステージングのファイルが.`eslintignore`に含まれている場合にコミットできなくなったので、
対処法をメモ。

## 前提条件

ESLint, husky, lint-staged の設定済み

ESLint: v7.30.0
lint-staged: v11.1.2
husky: 7.0.2

```json:package.json
"scripts": {
  ...,
  "prepare": "husky install",
  "lint-staged": "lint-staged"
},
"lint-staged": {
    "*.{js,ts,jsx,tsx}": [
      "eslint --max-warnings=0 ."
    ]
  },
```

```ignore:.eslintignore
src/test.ts
```

```ts:src/test.ts
import { Hoge } from './hoge';
export const hogehoge = () => {}
```

## 前提条件の状態でコミットを試みる

前提条件にある`src/test.ts`をコミットしようとすると下記の Warning が表示されコミットできない。

```
0:0 warning File ignored because of a matching ignore pattern. Use "--no-ignore" to override
```

`.eslintignore`に設定されているファイルをコミットしようとすると、上記の Warning が発生しそれが`--max-warnings=0`にひっかかっているのか？
原因はわからないが、`.eslintignore`に設定されているファイルを`--max-warnings=0`で ESLint を実行するとダメなようだ。

## 対処法

lint-staged の README に書かれている内容を試すとコミットできるようになった。

https://github.com/okonet/lint-staged#how-can-i-ignore-files-from-eslintignore

具体的には`.lintstagedrc.js`に下記のソースを追加し、lint-staged に読み込ませることで対処が可能。

:::message
ESLint のバージョンには注意が必要
:::

ESLint のバージョン >= 7

```js:.lintstagedrc.js
const { ESLint } = require('eslint')

const removeIgnoredFiles = async (files) => {
  const eslint = new ESLint()
  const isIgnored = await Promise.all(
    files.map((file) => {
      return eslint.isPathIgnored(file)
    })
  )
  const filteredFiles = files.filter((_, i) => !isIgnored[i])
  return filteredFiles.join(' ')
}

module.exports = {
  '**/*.{ts,tsx,js,jsx}': async (files) => {
    const filesToLint = await removeIgnoredFiles(files)
    return [`eslint --max-warnings=0 ${filesToLint}`]
  },
}
```

ESLint のバージョン < 7

```js:.lintstagedrc.js
const { CLIEngine } = require('eslint')

const cli = new CLIEngine({})

module.exports = {
  '*.js': (files) =>
    'eslint --max-warnings=0 ' + files.filter((file) => !cli.isPathIgnored(file)).join(' '),
}
```

```json:package.json
"scripts": {
  ...,
  "lint-staged": "lint-staged --config .lintstagedrc.js"
},
```

原理はよくわかってないが、`.eslintignore`対象のファイルを除外して ESLint を実行してるってことなのかな。

これで`.eslintignore`に含まれているファイルをコミットすることができたら成功です 👍

## 参考

https://github.com/okonet/lint-staged#how-can-i-ignore-files-from-eslintignore
