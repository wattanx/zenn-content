---
title: "microCMS SDKのC#版を作成した"
emoji: "👻"
type: "tech"
topics: ["csharp", "microcms"]
published: false
---

## はじめに

microCMS 様が公式に出している SDK には私もお世話になっています。

https://github.com/microcmsio/microcms-js-sdk

様々な言語の SDK が用意されていますが、C#はまだ提供されていないようです。
C#でも microCMS を用いたアプリケーションの開発がスムーズにできるように、上記の microCMS SDK を参考にしながら C#版の microCMS SDK を作成しました。

https://github.com/wattanx/microcms-csharp-sdk

## 使い方

さくっと確認するために UnitTest のプロジェクトを作成しコンテンツを取得してみます。

### microCMS で API の作成

microCMS 側で API を作成します。基本的な操作や作成方法は下記のドキュメントを参照ください。

https://document.microcms.io/manual/getting-started

スキーマ設定は下記の記事と同じ設定にします。
https://zenn.dev/wattanx/articles/d45d5627ffef54#microcms-%E3%81%AE-api-%E3%82%B9%E3%82%AD%E3%83%BC%E3%83%9E%E8%A8%AD%E5%AE%9A

### Install

- Package Manager

```
Install-Package microCMS.SDK -Version 1.1.0
```

- .NET CLI

```
dotnet add package microCMS.SDK --version 1.1.0
```

### microCMS のコンテンツのクラスの作成

microCMS からの Response をタイプセーフに扱うために、コンテンツ用のクラスを作成します。

microcms-js-sdk と同様に List 形式とオブジェクト形式に対応するベースクラスを提供しています。(`MicroCMSListContent`, `MicroCMSObjectContent`)

microCMS からの Response はプロパティ名が camelCase ですが、C#のコーディング規則ではプロパティ名は PascalCase です。
ちょっと手間ですが、コーディング規則に合わせて PascalCase にします。

https://docs.microsoft.com/ja-jp/dotnet/csharp/fundamentals/coding-style/coding-conventions

ただし、Response をデシリアライズするときに PascalCase のままではできないので、`JsonProperty`属性を付けて camelCase を指定します。

```csharp:Model.cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using Newtonsoft.Json;

using microCMS.SDK.Client;
using microCMS.SDK.Query;

namespace Sample
{
    public class Category : MicroCMSListContent
    {
        [JsonProperty("name")]
        public string Name { get; set; }
    }

    public class Tag : MicroCMSListContent
    {
        [JsonProperty("name")]
        public string Name { get; set; }
    }

    public class Author : MicroCMSListContent
    {
        [JsonProperty("name")]
        public string Name { get; set; }

        [JsonProperty("text")]
        public string Text { get; set; }
    }

    public class PopularArticles : MicroCMSObjectContent
    {
        [JsonProperty("articles")]
        public IEnumerable<Blog> Articles { get; set; }
    }

    public class Blog : MicroCMSListContent
    {
        [JsonProperty("title")]
        public string Title { get; set; }

        [JsonProperty("category")]
        public Category Category { get; set; }

        [JsonProperty("tag")]
        public IEnumerable<Tag> Tag { get; set; }

        [JsonProperty("toc_visible")]
        public bool TocVisible { get; set; }

        [JsonProperty("body")]
        public string Body { get; set; }

        [JsonProperty("description")]
        public string Description { get; set; }

        [JsonProperty("ogimage")]
        public MicroCMSImage OgImage { get; set; }

        [JsonProperty("writer")]
        public Author Writer { get; set; }

        [JsonProperty("partner")]
        public string Partner { get; set; }

        [JsonProperty("related_blogs")]
        public IEnumerable<Blog> RelatedBlog { get; set; }
    }
}

```

### MicroCMSClient を利用してコンテンツを取得

次に microCMS からデータを取得します。
`MicroCMSClient`を利用すると様々な形式のデータが取得可能です。

| Method                                                                                         | Description                                           |
| :--------------------------------------------------------------------------------------------- | :---------------------------------------------------- |
| Task<T> Get<T>(GetRequest request)                                                             | microCMS からコンテンツを取得し、T 型で返す           |
| Task<MicroCMSListResponse<T>> GetList<T>(GetListRequest request) where T : MicroCMSListContent | microCMS から List データを取得し、T 型で返す         |
| Task<T> GetListDetail<T>(GetListDetailRequest request) where T : MicroCMSListContent           | microCMS から List の詳細データを取得し、T 型で返す   |
| Task<T> GetObject<T>(GetObjectRequest request) where T : MicroCMSObjectContent                 | microCMS から Object 形式のデータを取得し、Ｔ型で返す |

```csharp:Sample.cs
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.Collections.Generic;
using System.Linq;
using Newtonsoft.Json;

using microCMS.SDK.Client;
using microCMS.SDK.Query;

[TestClass]
public class Sample
{
    [TestMethod]
    public void Get()
    {
        var apiKey = "自分のApiKey";
        var serviceDomain = "自分のServiceDomain";

        var queries = new MicroCMSQueries()
        {
            Limit = 10
        };
        var client = new MicroCMSClient(serviceDomain, apiKey);

        // ResponseはT型にデシリアライズされます。
        var response = client.GetList<Category>(new GetListRequest() { Endpoint = "categories", Queries = queries }).Result;

        // JSON化してResponseを確認する
        Console.WriteLine(JsonConvert.SerializeObject(response));
    }
}
```

上記のリクエストでは下記のようなデータが返ってきます。

```json
{
  "contents": [
    {
      "name": "チュートリアル",
      "id": "f3e0lszs7x9o",
      "createdAt": "2021-04-29T08:00:04.541Z",
      "updatedAt": "2021-10-07T12:56:11.597Z",
      "publishedAt": "2021-04-29T08:00:04.541Z",
      "revisedAt": "2021-10-07T12:56:11.597Z"
    },
    {
      "name": "更新情報",
      "id": "3comasi7os1f",
      "createdAt": "2021-04-29T07:59:49.765Z",
      "updatedAt": "2021-10-07T12:56:32.224Z",
      "publishedAt": "2021-04-29T07:59:49.765Z",
      "revisedAt": "2021-10-07T12:56:32.224Z"
    },
    {
      "name": "テスト",
      "id": "goul34gb1qa",
      "createdAt": "2021-04-29T07:59:28.655Z",
      "updatedAt": "2021-10-07T12:56:41.037Z",
      "publishedAt": "2021-04-29T07:59:28.655Z",
      "revisedAt": "2021-10-07T12:56:41.037Z"
    }
  ],
  "totalCount": 3,
  "limit": 10,
  "offset": 0
}
```

queries で指定できるパラメータはドキュメントを参照してください。

https://document.microcms.io/content-api/get-list-contents

queries の利用例として、`filters`を用いて id 指定したい場合の一例です。

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.Collections.Generic;
using System.Linq;
using Newtonsoft.Json;

using microCMS.SDK.Client;
using microCMS.SDK.Query;

[TestClass]
public class Sample
{
    [TestMethod]
    public void Get()
    {
        var apiKey = "自分のApiKey";
        var serviceDomain = "自分のServiceDomain";

        var queries = new MicroCMSQueries()
        {
            Filters = "id[equals]f3e0lszs7x9o"
        };
        var client = new MicroCMSClient(serviceDomain, apiKey);

        // ResponseはT型にデシリアライズされます。
        var response = client.GetList<Category>(new GetListRequest() { Endpoint = "categories", Queries = queries }).Result;

        // JSON化してResponseを確認する
        Console.WriteLine(JsonConvert.SerializeObject(response));
    }
}

```

上記のリクエストでは下記のようなデータが返ってきます。

```json
{
  "contents": [
    {
      "name": "チュートリアル",
      "id": "f3e0lszs7x9o",
      "createdAt": "2021-04-29T08:00:04.541Z",
      "updatedAt": "2021-10-07T12:56:11.597Z",
      "publishedAt": "2021-04-29T08:00:04.541Z",
      "revisedAt": "2021-10-07T12:56:11.597Z"
    }
  ],
  "totalCount": 1,
  "limit": 10,
  "offset": 0
}
```

## 今後改善したいこと

C#の命名規則に従って作成しているので、毎回`JsonProperty`を付ける必要がありプロパティの数が多いと大変です。

```csharp
public class Category : MicroCMSListContent
{
    [JsonProperty("name")]
    public string Name { get; set; }
}
```

なので、`T4 Template`を使って microCMS のスキーマから自動生成したいと思ってます。

もし C#で microCMS を利用する機会があればぜひご利用ください。
