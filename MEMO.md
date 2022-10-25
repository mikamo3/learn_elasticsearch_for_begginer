# ElasticsearchとRDBの用語の紐付け

概ね以下の理解で良さそう

|ES|RDB|
|----|----|
|Index|データベース|
|Type|テーブル|
|Document|レコード|

ただしTypeの指定は非推奨で_docを指定することになっている


# 主なオペレーション

```
GET _search #検索

PUT /library/_doc/1 #追加、更新
{
  "title":"hoge",
  "name":{
    "first":"foo",
    "last":"bar"
  },
  "publish_date":"1999-01-02T00:00:00+0900",
  "price": 20.15
}

GET /library/_doc/1 #ドキュメント取得


POST /library/_doc/ #ドキュメントIDを指定せずにドキュメントを作成、レスポンスでID確認
{
  "title":"hoge",
  "name":{
    "first":"foo",
    "last":"bar"
  },
  "publish_date":"1999-01-02T00:00:00+0900",
  "price": 20.15
}

POST /インデックス/_update/ドキュメントID #指定した箇所のみ更新
{
  "doc": {
    "price": 10
  }
}

DELETE /library/_doc/1 #ドキュメントの削除
DELETE /library #index削除

GET /library/_search #index内のすべてのドキュメントを検索
```


クエリは以下のような感じ
```
{"query":
  "match":{
    "title":"fox"
  }
}

```

スペース区切りでor
スペース含む場合はmatch_phraseを使う


AND検索
boolを使う
```
"query":{
  "bool":{
    "must":[
      {"match":{"title":"quick"}},
      {"match_phrase":{"title":"lazy dog"}},
    ]
  }
}
```

フィルタリング
filterクエリを指定

```
{
  "query":{
    "bool":{
      "filter":{
        "range":{
          "price":{
            "gte":5,
            "lte":10
          }
        }
      }
    }
  }
}
```

# マッピング

```
GET /library/_mapping #確認
```

追加
```
PUT /library/_mapping
{
  "properties":{
    "my_new_field":{
      "type": "text",
      "analyzer":"english"#任意でanalyzerを追加できる
    }
  }
}
```

一度追加したマッピングは変更不可

# Analysis

文字列の分析結果がわかる

```
GET /library/_analyze
```

```
{
  "tokens" : [
    {
      "token" : "Brown",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "fox",
      "start_offset" : 6,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "brown",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "dog",
      "start_offset" : 16,
      "end_offset" : 19,
      "type" : "<ALPHANUM>",
      "position" : 3
    }
  ]
}

```

tokenaizerをインストールして適切にドキュメントの分析が可能

日本語だとkuromojiとか

