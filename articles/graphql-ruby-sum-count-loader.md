---
title: ''
emoji: '🤖'
type: 'tech'
topics: ['graphql', 'rails', 'ruby']
published: false
---

## PostReview の作成

PostReview を作成します。
簡単に投稿に対してスコアをつける形にします。(バリデーションは省略)

migrate ファイルを用意し、score を `null: false` にします。

```
$ bin/rails g model PostReview score:integer post:references
```

```ruby
# xxxx_create_post_reviews.rb
class CreatePostReviews < ActiveRecord::Migration[6.1]
  def change
    create_table :post_reviews do |t|
      t.integer :score, null: false
      t.references :post, null: false, foreign_key: true

      t.timestamps
   end
  end
end

# post_review.rb
class PostReview < ApplicationRecord
  belongs_to :post
end
```

マイグレーションします。

```
$ bin/rails db:migrate
```

Post に has-many で設定します。

```diff ruby:app/models/post.rb
class Post < ApplicationRecord
  has_many :comments, dependent: :destroy
+ has_many :reviews, class_name: 'PostReview', dependent: :destroy
end

```
