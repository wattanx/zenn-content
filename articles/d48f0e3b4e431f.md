---
title: "ポートフォリオサイトをNuxt.jsからNext.jsに置き換える" # 記事のタイトル
emoji: "🐺" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["nuxtjs", "vercel", "nextjs"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

社会人 3 年目に入り、何かしらアウトプットを作ろうかなということでポートフォリオサイトを作ることにしました。
最初は趣味で勉強していて Nuxt.js で作成し Netlify にデプロイしましたが、いろいろあって結局 Next.js に置き換えました。
モンハンライズを買ってしまったので、なかなか進まなかったですが無事置き換えることができたのでいろいろ詰まったところとか書こうと思います。

https://wattanx.vercel.app/

# 環境

- 以前
  Nuxt.js 2.15.3
  TypeScript 4.1.5
  vuex-module-decorators 1.0.1
  Vuetify

- 移行後
  Next.js 10.1.2
  TypeScript 4.2.3

# データ管理

API は micro CMS を利用しています。
職務経歴とかスキルは micro CMS 経由で取得しています。
Zenn の記事を表示しているのですが、こちらの記事を参考にさせていただきました。
https://zenn.dev/catnose99/articles/cb72a73368a547756862

# なぜ移行したか

- Netlify にデプロイした場合、画像表示が遅い
- TypeScript をより活かしたいから React にしたい
- React 使うなら Next.js 使って Vercel にデプロイしたい

# 苦労したこと

## スタイル(CSS)

Vuetify にゴリゴリに依存していたため、自前で実装する必要があった。
　 → 　 Vuetify のデザインが気に入ってたため、他の CSS フレームワークを選択しなかった。

## 環境変数の使い方

なんか「.env」とか「.env.local」とか種類多すぎてわからん。

以下 Next.js v9.4 以上の場合

| ファイル名               | 定義するもの                                                       |
| ------------------------ | ------------------------------------------------------------------ |
| .env                     | すべての環境に対するデフォルト設定。                               |
| .env.local               | すべての環境に対する公開したくない設定。                           |
| .env.[environment]       | 特定の環境に対するデフォルト設定。 `.env.development`など          |
| .env.[environment].local | 特定の環境に対する公開したくない設定。`.env.development.local`など |

クライアント側で使う環境変数は`NEXT_PUBLIC_`をつけないといけない。

```env:.env
NEXT_PUBLIC_SITENAME="Test"
```

ドキュメントを見る感じだと、
`.env`とか`.env.[environment]`はレポジトリに含めてデフォルト設定として利用する。

`.env.local`とか`.env.[environment].local`は公開したくない API キーとか変数を設定して、`.gitignore`に含める。って感じでした。

https://nextjs.org/docs/basic-features/environment-variables

## Nuxt.js の asyncData 関数は Next.js では何なのか

SSG で作成する場合は`getStaticProps`でした。
`getInitialProps`じゃないのね。

実装は以下を参考にすればできました。
https://nextjs.org/docs/basic-features/data-fetching

## 百竜夜行はどう攻略するか

- 里守バリスタとか自動型の設備をちゃんと設置する
- 反撃の狼煙が出てるときは、武器で攻撃する。
- 青いマークが出てる関門を攻撃する敵を優先して倒す
- 気合
  友達の少ない私は一人でイブシマキヒコに挑みました。

# 移行前のパフォーマンス

![](https://storage.googleapis.com/zenn-user-upload/2qee2tio51d2inpesoikzwrfpcor)

ブラウザリフレッシュをしています
画像表示が遅い...

![](https://storage.googleapis.com/zenn-user-upload/4xmpmz68q3k4w0wsrtj80mmp7czu)

# 移行後のパフォーマンス

![](https://storage.googleapis.com/zenn-user-upload/b500pmroymmmlz1k2f81fty040ly)

ブラウザリフレッシュをしています
爆速。（もはやブラウザリフレッシュしてるのかもわからないくらい）

![](https://storage.googleapis.com/zenn-user-upload/o3aikzx18qoiij5qc157yfmh2tt5)

# まとめ

Netlify 無料プランの CDN は日本リージョン未対応だからっていうのが原因なんですかね。
パフォーマンスが気になるなら Vercel がいいかなーって感じました。
あと React を一から勉強する必要がありましたが、チュートリアルやってあとはググればなんとかなりました。
個人的に使い分けがまだわからないと感じるのは、クラスコンポーネントと関数コンポーネントの使い分け。
感覚的には、関数コンポーネントが使いやすいのだがどう使い分けるのか。

まあなにより
一狩りいこうぜ！

https://wattanx.vercel.app/

Vercel と Netlify の比較は以下のスクラップがわかりやすいと思いました。
https://zenn.dev/catnose99/scraps/6780379210136f
