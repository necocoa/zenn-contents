---
title: ''
emoji: '🤖'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['graphql', 'rails', 'ruby']
published: false
---

```diff ruby:app_schema.rb
class RailsGraphqlApiSchema < GraphQL::Schema
  mutation(Types::MutationType)
  query(Types::QueryType)

  use GraphQL::Batch
+ use GraphQL::Pagination::Connections
+ default_max_page_size 20

  # 略
end
```

```diff ruby
module Resolvers
  class PostsResolver < Resolvers::BaseResolver
-   description 'Find all posts'
-   type [Types::PostType], null: false
+   description 'Find connection posts'
+   type Types::PostType.connection_type, null: false

    def resolve
      Post.all
    end
  end
end
```

```graphql
query ($after: String, $first: Int) {
  posts(after: $after, first: $first) {
    nodes {
      id
      title
      countComments
    }
    pageInfo {
      endCursor
      hasNextPage
      hasPreviousPage
      startCursor
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
query ($after: String, $first: Int) {
  posts(after: $after, first: $first) {
    totalCount
    nodes {
      id
      title
      totalComments
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
   (1.7ms)  SELECT COUNT(*) FROM "posts"
  Post Load (2.2ms)  SELECT "posts".* FROM "posts" LIMIT $1  [["LIMIT", 4]]
  Post Exists? (2.2ms)  SELECT 1 AS one FROM "posts" LIMIT $1 OFFSET $2  [["LIMIT", 1], ["OFFSET", 4]]
   (4.9ms)  SELECT COUNT(*) AS count_all, "comments"."post_id" AS comments_post_id FROM "comments" WHERE "comments"."post_id" IN ($1, $2, $3, $4) GROUP BY "comments"."post_id"  [[nil, 2], [nil, 3], [nil, 4], [nil, 5]]
Completed 200 OK
```

### コメントをページネーション

```diff ruby
module Types
  class PostType < Types::BaseObject
    field :body, String, null: false
-   field :comments, [Types::CommentType], null: false
+   field :comments, Types::CommentType.connection_type, null: false
    field :created_at, GraphQL::Types::ISO8601DateTime, null: false
    field :id, ID, null: false
    field :sum_comments, Integer, null: false
    field :title, String, null: false
    field :total_comments, Integer, null: false
    field :updated_at, GraphQL::Types::ISO8601DateTime, null: false

    def comments
-     Loaders::AssociationLoader.for(Post, :comments).load(object)
+     object.comments
    end

    def total_comments
      Loaders::AssociationCountLoader.for(Post, :comments).load(object)
    end

    def sum_comments
      Loaders::AssociationSumLoader.for(Post, :comments, :id).load(object)
    end
  end
end
```

```graphql
query ($id: ID!, $after: String, $first: Int) {
  post(id: $id) {
    id
    title
    body
    totalComments
    comments(first: $first, after: $after) {
      nodes {
        id
        body
      }
      pageInfo {
        endCursor
        hasNextPage
      }
    }
  }
}
```

```json
{
  "id": 4,
  "first": 4,
  "after": ""
}
```

![](https://storage.googleapis.com/zenn-user-upload/f3d459d305f484203c26d8d2.jpg)

```sql
Started POST "/graphql"
  Post Load (1.9ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 4], ["LIMIT", 1]]
  Comment Load (2.4ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = $1 LIMIT $2  [["post_id", 4], ["LIMIT", 4]]
  Comment Exists? (1.9ms)  SELECT 1 AS one FROM "comments" WHERE "comments"."post_id" = $1 LIMIT $2 OFFSET $3  [["post_id", 4], ["LIMIT", 1], ["OFFSET", 4]]
   (3.8ms)  SELECT COUNT(*) AS count_all, "comments"."post_id" AS comments_post_id FROM "comments" WHERE "comments"."post_id" = $1 GROUP BY "comments"."post_id"  [["post_id", 4]]
Completed 200 OK
```

### Post のレビュー

```
$ rails g model PostReview point:integer post:references
```

```diff ruby
# post_review.rb
+ class PostReview < ApplicationRecord
+   belongs_to :post
+ end

# post.rb
class Post < ApplicationRecord
  has_many :comments, dependent: :destroy
+ has_many :reviews, class_name: 'PostReview', dependent: :destroy
end

```

https://graphql-ruby.org/pagination/using_connections

ページ情報を付与する方法
https://note.com/tana_d/n/n39e2b8dcd4ee
