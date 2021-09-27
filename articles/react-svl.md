---
title: "React用の簡単なバリデーションライブラリを作りました"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "validation"]
published: false
---

# はじめに

React を用いた Web アプリケーション開発において、Form のバリデーションをどうするかという問題に遭遇しました。
`React Hook Form`や`Formik`などいろんなライブラリを比較・検討してきましたが、
そこまで大きな機能は必要なく簡単なバリデーションができればいいという考えからライブラリの自作に至りました。

https://github.com/wattanx/react-svl

## 欲しかった機能

バリデーションライブラリの機能として欲しかったのは

- バリデーションのルールを指定できる
- Controlled Components を利用する
- 必須チェック、最小・最大値チェック、最大・最小文字長チェック

## 使い方
