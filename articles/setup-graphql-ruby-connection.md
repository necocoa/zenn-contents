---
title: ''
emoji: '🤖'
type: 'tech'
topics: ['graphql', 'rails', 'ruby']
published: false
---

## 設定

バージョンいくつからかわかりませんが、最新ではデフォルトで有効になったようです。
`use GraphQL::Pagination::Connections`

`default_max_page_size` を設定するとその件数が LIMIT の初期値になります。

```diff ruby:app_schema.rb
class RailsGraphqlApiSchema < GraphQL::Schema
  mutation(Types::MutationType)
  query(Types::QueryType)

  use GraphQL::Batch
+ default_max_page_size 20

  # 略
end
```

## Connection に変更

```diff ruby
module Resolvers
  class PostsResolver < Resolvers::BaseResolver
-   description 'Find all posts'
-   type [Types::PostType], null: false
+   description 'Find connection posts'
+   type Types::PostType.connection_type, null: false

    def resolve
      Post.all.recent
    end
  end
end
```

引数に after, before, first, last が追加され、返り値が PostConnection 型になります。
![](https://storage.googleapis.com/zenn-user-upload/c1e866c22c38fc2e4cf37d69.jpg)

```graphql
query ($first: Int, $after: String) {
  posts(first: $first, after: $after) {
    nodes {
      id
      title
    }
    pageInfo {
      endCursor
      hasNextPage
    }
  }
}
```

```json
{
  "first": 2,
  "after": ""
}
```

## 総件数項目を追加

```diff ruby
module Types
  class BaseConnection < Types::BaseObject
    # add `nodes` and `pageInfo` fields, as well as `edge_type(...)` and `node_nullable(...)` overrides
    include GraphQL::Types::Relay::ConnectionBehaviors

+   field :total_count, Integer, '総件数', null: false

+   def total_count
+     object.items.size
+   end
  end
end
```

### posts

```graphql
query ($first: Int, $after: String) {
  posts(first: $first, after: $after) {
    totalCount
    nodes {
      id
      title
    }
    pageInfo {
      endCursor
      hasNextPage
    }
  }
}
```

```json
{
  "first": 4,
  "after": ""
}
```

![](https://storage.googleapis.com/zenn-user-upload/cd3a49844748e918e34584d7.jpg)

```sql
Started POST "/graphql"
   (1.7ms)  SELECT "posts".* FROM "posts" ORDER BY "posts"."created_at" DESC
  Post Load (2.2ms)  SELECT "posts".* FROM "posts" LIMIT $1  [["LIMIT", 4]]
  Post Exists? (2.2ms)  SELECT 1 AS one FROM "posts" LIMIT $1 OFFSET $2  [["LIMIT", 1], ["OFFSET", 4]]
   (4.9ms)  SELECT COUNT(*) AS count_all, "comments"."post_id" AS comments_post_id FROM "comments" WHERE "comments"."post_id" IN ($1, $2, $3, $4) GROUP BY "comments"."post_id"  [[nil, 2], [nil, 3], [nil, 4], [nil, 5]]
Completed 200 OK
```

https://graphql-ruby.org/pagination/using_connections

ページ情報を付与する方法
https://note.com/tana_d/n/n39e2b8dcd4ee
