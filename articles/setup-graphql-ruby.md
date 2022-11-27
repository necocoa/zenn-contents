---
title: 'RailsでGraphQL APIをつくるチュートリアル[graphql-ruby]'
emoji: '✍️'
type: 'tech'
topics: ['graphql', 'rails', 'ruby']
published: true
---

この記事では GraphQL 自体の説明や GraphQL 特有の用語の説明などは行いません。
Rails をつかって GraphQL の API をサクっと作ってみるという趣旨で作成しています。

```
ruby: 3.0.1
rails: 6.1.7
graphql-ruby: 2.0.15
```

## GraphQL のセットアップ

API モードで `rails new` します。
minitest を使わないためオフにしています。

```
$ rails new rails-graphql-api -T --api -d postgresql
```

### GraphQL Ruby の導入

```diff ruby:Gemfile.rb
# Gemfile
+ gem 'graphql'
```

```
$ bundle install
```

続いて `bin/rails g graphql:install` を実行し、必要なファイルをセットアップします。

```
$ bin/rails g graphql:install
```

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

:::details 実行結果

<!-- textlint-enable -->

```
$ bin/rails g graphql:install
Running via Spring preloader in process 50049
      create  app/graphql/types
      create  app/graphql/types/.keep
      create  app/graphql/rails_graphql_api_schema.rb
      create  app/graphql/types/base_object.rb
      create  app/graphql/types/base_argument.rb
      create  app/graphql/types/base_field.rb
      create  app/graphql/types/base_enum.rb
      create  app/graphql/types/base_input_object.rb
      create  app/graphql/types/base_interface.rb
      create  app/graphql/types/base_scalar.rb
      create  app/graphql/types/base_union.rb
      create  app/graphql/types/query_type.rb
add_root_type  query
      create  app/graphql/mutations
      create  app/graphql/mutations/.keep
      create  app/graphql/mutations/base_mutation.rb
      create  app/graphql/types/mutation_type.rb
add_root_type  mutation
      create  app/controllers/graphql_controller.rb
       route  post "/graphql", to: "graphql#execute"
Skipped graphiql, as this rails project is API only
  You may wish to use GraphiQL.app for development: https://github.com/skevy/graphiql-app
      create  app/graphql/types/node_type.rb
      insert  app/graphql/types/query_type.rb
      create  app/graphql/types/base_connection.rb
      create  app/graphql/types/base_edge.rb
      insert  app/graphql/types/base_object.rb
      insert  app/graphql/types/base_object.rb
      insert  app/graphql/types/base_union.rb
      insert  app/graphql/types/base_union.rb
      insert  app/graphql/types/base_interface.rb
      insert  app/graphql/types/base_interface.rb
      insert  app/graphql/rails_graphql_api_schema.rb
```

:::

### GraphiQL IDE の導入

GraphQL API の動作確認のため GraphiQL を入れます。
入れる方法はアプリか Gem の二通りがあります。

1. GraphiQL.app
2. gem graphiql-rails

Gem 経由だと Sprockets で IDE が実行されますが、API モードですと Sprockets が有効になっていないため有効にする必要があります。
※API モードではない場合 graphiql-rails を入れた状態で `bin/rails g graphql:install` を実行すると勝手にセットアップしてくれます。

IDE のために Sprockets を有効にするのは微妙な気がするため、自分はアプリをインストールする方法を選びました。

:::message

GraphiQL は Facebook 製ですが、Prisma 製の [GraphQL Playground](https://github.com/graphql/graphql-playground) や Apollo 製の [Apollo Studio](https://www.apollographql.com/docs/studio/) など GraphQL を効率的に開発するツールがどんどん出てきているようです。
GraphiQL しか使ったことがなかったので、他 IDE も使ってみようと思います。

:::

#### 1. GraphiQL アプリをインストールする

https://github.com/skevy/graphiql-app

brew cask 経由でインストールします。

```
$ brew install --cask graphiql
```

インストールが完了したら GraphiQL を開きます。
この際セキュリティとプライバシーでブロックされるので許可します。

GraphQL Endpoint に `http://localhost:3000/graphql` を入力します。
generate したままだと SampleQuery として testField が用意されているので実行できるか確認してみましょう。
メニューバーの ▶ を押すと実行されます。

```graphql
{
  testField
}
```

![](https://storage.googleapis.com/zenn-user-upload/j0e8th31txyfn7ki49kx7hbonfm7)

#### 2. Gem によるインストール

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

:::details Gem によるインストール方法

<!-- textlint-enable -->

Rails 上で GraphiQL IDE が使えるようになる graphiql-rails を追加します。

```ruby:Gemfile
gem 'graphiql-rails'
```

`/graphiql` のパスに GraphiQL が表示されるように `routes.rb` を編集します。

```ruby:config/routes.rb
# 以下を追加
if Rails.env.development?
  mount GraphiQL::Rails::Engine, at: "/graphiql", graphql_path: "/graphql"
end
```

続いて Sprockets を使って GraphiQL の画面をセットします。
API モードだと Sprockets が無効になっているため、有効にしていきます。

> Note on API Mode
> If you're using Rails 5 in "API mode", you'll also need to add require "sprockets/railtie" to your application.rb.

```diff ruby:config/application.rb
-  # require "sprockets/railtie"
+  require "sprockets/railtie"
```

sprockets に必要な manifest を作成します。
今回は開発環境のみ sprockets を使用するため、中身は空のままで問題ありません。

```
$ mkdir -p app/assets/config && touch app/assets/config/manifest.js
```

assets ファイルを作り、開発環境のときに GraphiQL の assets を読み込むようにします。

```
$ touch app/initializers/assets.rb
```

```ruby:config/initializers/assets.rb
if Rails.env.development?
  Rails.application.config.assets.precompile += %w[graphiql/rails/application.js
                                                   graphiql/rails/application.css]
end

```

```
$ bin/rails s
```

http://localhost:3000/graphiql 以下の画面が表示されれば OK です。

![](https://storage.googleapis.com/zenn-user-upload/ki3ahslz7m0fs425p6ytevf2w834)

:::

## Query を作ってみる

### Model の作成

Post と Comment のモデルを作成します。

```
$ bin/rails g model post title:string body:text
$ bin/rails g model comment post:references body:text
```

各カラムに `null: false` を追加します。

```ruby
# xxxx_create_posts.rb
class CreatePosts < ActiveRecord::Migration[6.1]
  def change
    create_table :posts do |t|
      t.string :title, null: false
      t.text :body, null: false

      t.timestamps
    end
    add_index :posts, :created_at
  end
end

# xxxx_create_comments.rb
class CreateComments < ActiveRecord::Migration[6.1]
  def change
    create_table :comments do |t|
      t.references :post, null: false, foreign_key: true
      t.text :body, null: false

      t.timestamps
    end
  end
end
```

### Object Type の作成

つづいて Post と Comment の型を作成していきます。
モデルと同じ名前で作成すると勝手に各カラムと型を付けてくれます。
has-many 等は読み込んでくれないようなので自分で設定します。

`bin/rails g graphql:object 型名 カラム名:型` で作成できます。

```sh
$ bin/rails g graphql:object post comments:Comment
$ bin/rails g graphql:object comment
```

```diff ruby:app/graphql/types/post_type.rb
module Types
  class PostType < Types::BaseObject
    field :id, ID, null: false
    field :title, String, null: false
    field :body, String, null: false
-   field :comments, Types::CommentType, null: true
+   field :comments, [Types::CommentType], null: false
    field :created_at, GraphQL::Types::ISO8601DateTime, null: false
    field :updated_at, GraphQL::Types::ISO8601DateTime, null: false
  end
end
```

```diff ruby:app/graphql/types/comment_type.rb
module Types
  class CommentType < Types::BaseObject
    field :id, ID, null: false
-   field :post_id, Integer, null: false
+   field :post, Types::PostType, null: false
    field :body, String, null: false
    field :created_at, GraphQL::Types::ISO8601DateTime, null: false
    field :updated_at, GraphQL::Types::ISO8601DateTime, null: false
  end
end
```

### Query の作成

`query_type.rb` に Post を取得する Query を作成します。

```diff ruby:app/graphql/types/query_type.rb
module Types
  class QueryType < Types::BaseObject
    # Add `node(id: ID!) and `nodes(ids: [ID!]!)`
    include GraphQL::Types::Relay::HasNodeField
    include GraphQL::Types::Relay::HasNodesField

+   field :post, Types::PostType, null: false do
+     description 'Find a post by ID'
+     argument :id, ID, required: true
+   end
+
+   def post(id:)
+     Post.find(id)
+   end
  end
end

```

取得できるか試すため、データを作成します。

```sh
$ bin/rails c
post = Post.create(title: 'test', body: 'body')
post.comments.create(body: 'comment 1')
post.comments.create(body: 'comment 2')
```

GraphiQL で Query を実行してみましょう。

```graphql
query {
  post(id: "1") {
    id
    title
    body
    comments {
      id
      body
    }
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/j7lopcobppkkdzutwvgb4q33dp0j)

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

Post と Post に紐づく Comments を取得できました 🎉

<!-- textlint-enable -->

### Resolver に変えてみる

`query_type.rb` はどんどん肥大化していくため、件数が増えた場合や複雑な取得方法などで Resolver という形で分離します。
最初から Resolver を常に使う形でもよいかもしれません。

resolvers ディレクトリを作成します。

```
$ mkdir app/graphql/resolvers
```

他の Base に合わせて BaseResolver を作成します。

```ruby:app/graphql/resolvers/base_resolver.rb
module Resolvers
  class BaseResolver < GraphQL::Schema::Resolver
    argument_class Types::BaseArgument
  end
end
```

`post_resolver.rb` を作成します。
`def resolve ... end` 内で返したいデータの処理を行います。
resolve の引数には argument の値（複数ある場合は複数入ってくる）が入ってくるため `resolve(id:)` や `resolve(**args)` のような形で引数を受け取ります。

```ruby:app/graphql/resolvers/post_resolver.rb
module Resolvers
  class PostResolver < Resolvers::BaseResolver
    description 'Find a post by ID'
    type Types::PostType, null: false

    argument :id, ID, required: true

    def resolve(id:)
      Post.find(id)
    end
  end
end
```

`query_type.rb` の `field :post` を resolver の形に書き換えます。

```diff ruby:app/graphql/types/query_type.rb
module Types
  class QueryType < Types::BaseObject
    # Add `node(id: ID!) and `nodes(ids: [ID!]!)`
    include GraphQL::Types::Relay::HasNodeField
    include GraphQL::Types::Relay::HasNodesField

+   field :post, resolver: Resolvers::PostResolver
-   field :post, Types::PostType, null: false do
-     description 'Find a post by ID'
-     argument :id, ID, required: true
-   end
-
-   def post(id:)
-     Post.find(id)
-   end
  end
end
```

これで Resolver での取得に変更できました。
GraphiQL で同様の Query を使って取得できるか確認してみてください。

## Mutation をつくってみる

### Create Post Mutation

つづいては Post を作成する Mutation をつくります。
`g graphql:mutation` を使って作成すると `mutation_type.rb` への追加と `create_post.rb` の作成が行われます。

```
$ bin/rails g graphql:mutation create_post
```

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

:::details 実行結果

<!-- textlint-enable -->

```
$ bin/rails g graphql:mutation create_post
Running via Spring preloader in process 56616
       exist  app/graphql/mutations
   identical  app/graphql/mutations/.keep
   identical  app/graphql/mutations/base_mutation.rb
        skip  app/graphql/types/mutation_type.rb
add_root_type  mutation
      create  app/graphql/mutations/create_post.rb
        gsub  app/graphql/types/mutation_type.rb
```

:::

```diff ruby:app/graphql/types/mutation_post.rb
module Types
  class MutationType < Types::BaseObject
+   field :create_post, mutation: Mutations::CreatePost
  end
end
```

`create_post.rb` を編集していきます。
field は返り値の指定となります。
resolve の返り値は field の返り値に合わせてハッシュで返します。

```diff ruby:app/graphql/mutations/create_post.rb
module Mutations
  class CreatePost < BaseMutation
+   field :post, Types::PostType, null: false
+
+   argument :body, String, required: true
+   argument :title, String, required: true
+
+   def resolve(**params)
+     post = Post.create!(params)
+     { post: post }
+   end
  end
end
```

GraphiQL で試してみましょう。

```graphql
mutation ($input: CreatePostInput!) {
  createPost(input: $input) {
    post {
      title
      body
    }
  }
}
```

左下の QUERY VARIABLES をクリックすると入力できるようになります。

```json:variables
{
  "input": {
    "title": "new title",
    "body": "new body"
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/hh2n9gagroaoqn1xijhp4imhxjzo)

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

新しい Post を作成できました 🎉

<!-- textlint-enable -->

### Update Post Mutation ※InputObject での実装

argument は InputObject を使ってまとめることで共通化ができます。

試しに UpdatePost は InputObject を使ってみます。

inputs ディレクトリを作成します。

```
$ mkdir app/graphql/types/inputs
```

BaseInputObject はすでにあるので、`post_input_type.rb` を作成します。

```ruby:app/graphql/types/inputs/post_input_type.rb
module Types
  module Inputs
    class PostInputType < Types::BaseInputObject
      argument :id,    Int,    required: true
      argument :body,  String, required: false
      argument :title, String, required: false
    end
  end
end
```

つづいて、`update_post.rb` を作成します。

```
$ bin/rails g graphql:mutation update_post
```

```diff ruby:app/graphql/types/mutation_post.rb
module Types
  class MutationType < Types::BaseObject
    field :create_post, mutation: Mutations::CreatePost
+   field :update_post, mutation: Mutations::UpdatePost
  end
end
```

argument に Types::Inputs::PostInputType を渡すことで params は PostInputType の型を持つことができます。

```diff ruby:app/graphql/mutations/update_post.rb
module Mutations
  class UpdatePost < Mutations::BaseMutation
+   argument :params, Types::Inputs::PostInputType, required: true
+
+   def resolve(params:)
+     post_params = params.to_h
+     post = Post.find(post_params.delete(:id))
+     post.update!(post_params.compact)
+     post
+   end
  end
end
```

GraphiQL で実行してみましょう。

```graphql
mutation ($params: UpdatePostInput!) {
  updatePost(input: $params) {
    post {
      id
      title
      body
    }
  }
}
```

```json:variables
{
  "input": {
    "params": {
      "id": 1,
      "title": "update title"
    }
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/5upytboje347o4pogs2hqzl4k3ls)

Post の title を変更できました 🎉
InputObject は共通化したい argment ができたタイミングで作ると良いと思います。

#### 参考にさせてもらった記事

https://graphql-ruby.org/getting_started

https://qiita.com/dkawabata/items/4fd965ee6d7295386a8b

https://github.com/rmosolgo/graphiql-rails/issues/75

https://zenn.dev/kei178/articles/2f4ffc6b89618c
