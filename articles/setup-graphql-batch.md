---
title: '[N+1解消]RailsのGraphQLにgraphql-batchを導入する[graphql-ruby]'
emoji: '💬'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['graphql', 'rails', 'ruby']
published: false
---

この記事では GraphQL 自体の説明や GraphQL 特有の用語の説明などは行いません。
Rails をつかって GraphQL の API をサクっと作ってみるという趣旨で作成しています。

2021 年 5 月 27 日時点での最新 Version です。

```
ruby: 3.0.1
rails: 6.1.3.2
graphql-ruby: 1.12.10
graphql-batch: 0.4.3
```

GraphQL をセットアップし Port を取得する API は以前の記事で作成しております。

https://zenn.dev/necocoa/articles/setup-graphql-ruby

現在の Model は以下のとおりです。

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  has_many :comments
end

# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :post
end
```

## N+1 となる Query を作成

### Posts Query を作成する

まずは query type に posts field を追加します。

```diff ruby:app/graphql/types/query_type.rb
module Types
  class QueryType < Types::BaseObject
    # Add `node(id: ID!) and `nodes(ids: [ID!]!)`
    include GraphQL::Types::Relay::HasNodeField
    include GraphQL::Types::Relay::HasNodesField

    field :post, resolver: Resolvers::PostResolver
+   field :posts, resolver: Resolvers::PostsResolver
  end
end
```

### Posts Resolver を作成する

PostsResolver を作成します。
返り値はわかりやすく Posts 全件にします。
ページネーションについては今回は触りません。

```ruby:app/graphql/resolvers/posts_resolver.rb
module Resolvers
  class PostsResolver < Resolvers::BaseResolver
    description 'Find all posts'
    type [Types::PostType], null: false

    def resolve
      Post.all
    end
  end
end
```

### N+1 の Query を確認する

posts と has-many の comments を取得してみます。

```graphql
query {
  posts {
    id
    title
    comments {
      id
      body
    }
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/149166195490036355e221cf.jpg)

Comment の SQL が N+1 になりました。
こちらを graphql-batch をつかって解消していきます。

```
Started POST "/graphql"
  Post Load (1.8ms)  SELECT "posts".* FROM "posts"1
  Comment Load (3.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = $1  [["post_id", 1]]1
  Comment Load (4.9ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = $1  [["post_id", 2]]1
  Comment Load (3.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = $1  [["post_id", 3]]1
```

## GraphQL Batch を導入

### graphql-batch gem をインストール

GraphQL では指定した field をグラフ構造で取得していくため includes などを使って preload できません。
そのため graphql-batch という GraphQL 向けの N+1 解消ライブラリを使ってみます。

https://github.com/Shopify/graphql-batch

```diff:Gemfile.rb
+ gem 'graphql-batch'
```

```
$ bundle install
```

### GraphQL::Batch を有効化

続いて GraphQL::Batch を有効にします。
graphql ディレクトリ配下に任意のアプリ名の schema.rb があるので、そちらに追記します。

```diff ruby:app/graphql/rails_graphql_api_schema.rb
class RailsGraphqlApiSchema < GraphQL::Schema
  mutation(Types::MutationType)
  query(Types::QueryType)

+ use GraphQL::Batch

  # 略
end
```

## Loader を実装

GraphQL::Batch::Loader を継承した Loader を作成していきます。

### AssociationLoader を実装する

Record に関連する Record をまとめて取得する Loader をつくります。

graphql 配下に loaders ディレクトリを作成します。

```
$ mkdir app/graphql/loaders
```

example の association_loader を実装します。
https://github.com/Shopify/graphql-batch/blob/master/examples/association_loader.rb

Loader 内でなにをやっているのか知りたい方はこちらの記事が参考になりましたので、参照ください。
https://blog.kymmt.com/entry/graphql-batch-examples

```
$ touch app/graphql/loaders/association_loader.rb
```

```ruby:app/graphql/loaders/association_loader.rb
module Loaders
  class AssociationLoader < GraphQL::Batch::Loader
    def self.validate(model, association_name)
      new(model, association_name)
      nil
    end

    def initialize(model, association_name)
      super()
      @model = model
      @association_name = association_name
      validate
    end

    def load(record)
      raise TypeError, "#{@model} loader can't load association for #{record.class}" unless record.is_a?(@model)
      return Promise.resolve(read_association(record)) if association_loaded?(record)

      super
    end

    # We want to load the associations on all records, even if they have the same id
    def cache_key(record)
      record.object_id
    end

    def perform(records)
      preload_association(records)
      records.each { |record| fulfill(record, read_association(record)) }
    end

    private

    def validate
      return if @model.reflect_on_association(@association_name)

      raise ArgumentError, "No association #{@association_name} on #{@model}"
    end

    def preload_association(records)
      ::ActiveRecord::Associations::Preloader.new.preload(records, @association_name)
    end

    def read_association(record)
      record.public_send(@association_name)
    end

    def association_loaded?(record)
      record.association(@association_name).loaded?
    end
  end
end


```

次に PostType に comments メソッドを作成します。
AssociationLoader 経由で comments を取得するように実装します。

```diff ruby:app/graphql/types/post_type.rb
module Types
  class PostType < Types::BaseObject
    field :body, String, null: false
    field :comments, [Types::CommentType], null: false
    field :created_at, GraphQL::Types::ISO8601DateTime, null: false
    field :id, ID, null: false
    field :title, String, null: false
    field :updated_at, GraphQL::Types::ISO8601DateTime, null: false

+   def comments
+     Loaders::AssociationLoader.for(Post, :comments).load(object)
+   end
  end
end
```

Types::BaseObject の field では同ファイル内にメソッドがあればそちらを優先に、なければ `object.comments` が実行されます。
任意で `field :comments, [Types::CommentType], null: false, method: :loader_comments` のようにメソッドを指定も可能です。

これで再度実行してみましょう。

```
Started POST "/graphql"
  Post Load (2.0ms)  SELECT "posts".* FROM "posts"
  Comment Load (2.4ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" IN ($1, $2, $3)  [[nil, 1], [nil, 2], [nil, 3]]
Completed 200 OK
```

これでシンプルな関連レコードの N+1 は解消されました。

## カウンターを実装

続いては投稿に対してのコメント数を取得してみます。
シンプルに書くとこうですが、N+1 がでます。

```diff ruby:post_type.rb
module Types
  class PostType < Types::BaseObject
  # 略

+   def count_comments
+     object.comments.count
+   end
  end
end
```

```graphql
query {
  posts {
    id
    title
    countComments
  }
}
```

```
Started POST "/graphql"
  Post Load (6.5ms)  SELECT "posts".* FROM "posts"
   (2.9ms)  SELECT COUNT(*) FROM "comments" WHERE "comments"."post_id" = $1  [["post_id", 1]]
   (4.9ms)  SELECT COUNT(*) FROM "comments" WHERE "comments"."post_id" = $1  [["post_id", 2]]
   (1.9ms)  SELECT COUNT(*) FROM "comments" WHERE "comments"."post_id" = $1  [["post_id", 3]]
Completed 200 OK
```

同じく Loader を実装していきます。

```
$ touch app/graphql/loaders/association_count_loader.rb
```

```ruby:app/graphql/loaders/association_count_loader.rb
module Loaders
  class AssociationCountLoader < GraphQL::Batch::Loader
    def self.validate(model, association_name)
      new(model, association_name)
      nil
    end

    def initialize(model, association_name)
      super()
      @model = model
      @association_name = association_name
      @reflection = reflect
    end

    def load(record)
      raise TypeError, "#{@model} loader can't load association for #{record.class}" unless record.is_a?(@model)

      super
    end

    def perform(records)
      counts = preload_counts(records)
      records.each { |record| fulfill(record, read_count(record, counts) || 0) }
    end

    private

    def reflect
      reflection = @model.reflect_on_association(@association_name)
      return reflection if reflection

      raise ArgumentError, "No association #{@association_name} on #{@model}"
    end

    def preload_counts(records)
      klass = @reflection.klass
      field = @reflection.join_primary_key
      klass.where(field => records).group(field).count
    end

    def read_count(record, counts)
      record_key = record[@reflection.active_record_primary_key]
      counts[record_key]
    end
  end
end
```

Loader を使って Count を取得するように変更します。

```diff ruby:app/graphql/types/post_type.rb
module Types
  class PostType < Types::BaseObject
    # 略

    def count_comments
-     object.comments.count
+     Loaders::AssociationCountLoader.for(Post, :comments).load(object)
    end
  end
end
```

それでは実行してみましょう。

![](https://storage.googleapis.com/zenn-user-upload/4f794d6ed8a57832bab147c2.jpg)

```
Started POST "/graphql"
  Post Load (2.6ms)  SELECT "posts".* FROM "posts"
   (2.6ms)  SELECT COUNT(*) AS count_all, "comments"."post_id" AS comments_post_id FROM "comments" WHERE "comments"."post_id" IN ($1, $2, $3, $4, $5, $6, $7, $8) GROUP BY "comments"."post_id"  [[nil, 1], [nil, 2], [nil, 3]]
Completed 200 OK
```

これで関連データを SQL の COUNT のような集計関数をつかって実装する方法ができました。

## 参考資料

https://blog.agile.esm.co.jp/entry/2017/06/16/113248
https://blog.kymmt.com/entry/graphql-batch-examples
https://blog.jamesbrooks.net/graphql-batch-count-loader.html

---
