---
title: 'Railsのルーティング(routes)書き方 早見表'
emoji: '🏃'
type: 'tech'
topics: ['rails']
published: true
---

index 以外の action は省略しています。

https://railsguides.jp/routing.html

## nested resources

- Prefix: ○（単数形）
- Path: ○
- Id: ○
- Controller: ○

[ネストしたリソース](https://railsguides.jp/routing.html#%E3%83%8D%E3%82%B9%E3%83%88%E3%81%97%E3%81%9F%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9)

```ruby
resources :posts do
  resources :comments
end
```

| HTTP Verb | Prefix        | URI Pattern              | Controller#Action |
| --------- | ------------- | ------------------------ | ----------------- |
| GET       | post_comments | /posts/:post_id/comments | comments#index    |

### namespace

指定したネームスペース配下にルーティングする。

- Prefix: ○
- Path: ○
- Id: ×
- Controller: ○

[コントローラの名前空間とルーティング](https://railsguides.jp/routing.html#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%A9%E3%81%AE%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%E3%81%A8%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0)

```ruby
namespace :posts do
  resources :comments
end
```

| HTTP Verb | Prefix         | URI Pattern     | Controller#Action    |
| --------- | -------------- | --------------- | -------------------- |
| GET       | posts_comments | /posts/comments | posts/comments#index |

### scope, path

- Prefix: -
- Path: ○
- Id: ×
- Controller: -

```ruby
scope :posts do
  resources :comments
end

# 同上
resources :comments, path: '/posts/comments'
```

| HTTP Verb | Prefix   | URI Pattern     | Controller#Action |
| --------- | -------- | --------------- | ----------------- |
| GET       | comments | /posts/comments | comments#index    |

### module

- Prefix: -
- Path: -
- Id: -
- Controller: ○

```ruby
scope module: :posts do
  resources :comments
end

# 同上
resources :comments, module: :posts
```

| HTTP Verb | Prefix   | URI Pattern | Controller#Action    |
| --------- | -------- | ----------- | -------------------- |
| GET       | comments | /comments   | posts/comments#index |

### as

- Prefix: ○
- Path: -
- Id: -
- Controller: -

[3.6 名前付きルーティング](https://railsguides.jp/routing.html#%E5%90%8D%E5%89%8D%E4%BB%98%E3%81%8D%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0)

```ruby
resources :posts, as: :articles
```

| HTTP Verb | Prefix   | URI Pattern | Controller#Action |
| --------- | -------- | ----------- | ----------------- |
| GET       | articles | /posts      | posts#index       |

## action

### member

指定の action を id 配下に追加する。

[メンバールーティングを追加する](https://railsguides.jp/routing.html#%E3%83%A1%E3%83%B3%E3%83%90%E3%83%BC%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)

```ruby
resources :posts do
  member do
    get :detail
  end
end

# 同上
resources :posts do
  get :detail, on: :member
end
```

| HTTP Verb | Prefix      | URI Pattern       | Controller#Action |
| --------- | ----------- | ----------------- | ----------------- |
| GET       | detail_post | /posts/:id/detail | posts#detail      |

### collection

指定の action を index 配下に追加する。

[コレクションルーティングを追加する](https://railsguides.jp/routing.html#%E3%82%B3%E3%83%AC%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)

```ruby
resources :posts do
  collection do
    get :search
  end
end

# 同上
resources :posts do
  get :search, on: :collection
end
```

| HTTP Verb | Prefix       | URI Pattern   | Controller#Action |
| --------- | ------------ | ------------- | ----------------- |
| GET       | search_posts | /posts/search | posts#search      |

### on: :new

[追加された new アクションへのルーティングを追加する](https://railsguides.jp/routing.html#%E8%BF%BD%E5%8A%A0%E3%81%95%E3%82%8C%E3%81%9Fnew%E3%82%A2%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%B8%E3%81%AE%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)

```ruby
resources :posts do
  get :preview, on: :new
end
```

| HTTP Verb | Prefix           | URI Pattern        | Controller#Action |
| --------- | ---------------- | ------------------ | ----------------- |
| GET       | preview_new_post | /posts/new/preview | posts#preview     |

### to, action & controller

Controller と Action を指定する。

[単数形リソース](https://railsguides.jp/routing.html#%E5%8D%98%E6%95%B0%E5%BD%A2%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9)

```ruby
get :profile, to: 'users#show'
# 同上
get :profile, controller: :users, action: :show
```

| HTTP Verb | Prefix  | URI Pattern | Controller#Action |
| --------- | ------- | ----------- | ----------------- |
| GET       | profile | /profile    | users#show        |

### redirect

[リダイレクト](https://railsguides.jp/routing.html#%E3%83%AA%E3%83%80%E3%82%A4%E3%83%AC%E3%82%AF%E3%83%88)

```ruby
get :posts, to: redirect('/articles')
get '/posts/:id', to: redirect('/articles/%{id}')
```

| HTTP Verb | Prefix | URI Pattern | Controller#Action               |
| --------- | ------ | ----------- | ------------------------------- |
| GET       | posts  | /posts      | redirect(301, /articles)        |
| GET       |        | /posts/:id  | redirect(301, /articles/％{id}) |

## resource

単数リソースとして扱う。index と :id がなくなる。

[単数形リソース](https://railsguides.jp/routing.html#%E5%8D%98%E6%95%B0%E5%BD%A2%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9)

```ruby
resource :profile
```

| HTTP Verb | Prefix       | URI Pattern   | Controller#Action |
| --------- | ------------ | ------------- | ----------------- |
| GET       | new_profile  | /profile      | profiles#new      |
| GET       | edit_profile | /profile/edit | profiles#edit     |
| GET       | profile      | /profile      | profiles#show     |
| PATCH/PUT |              | /profile      | profiles#update   |
| DELETE    |              | /profile      | profiles#destroy  |
| POST      |              | /profile      | profiles#create   |

## shallow

ネストを必要最低限にする。

```ruby
resources :posts, shallow: true do
  resources :comments
end

# 同上
resources :posts do
  shallow do
    resources :comments
  end
end

# 同上
resources :posts do
  resources :comments, shallow: true
end

# 同上
resources :posts do
  resources :comments, only: [:index, :new, :create]
end
resources :comments, only: [:show, :edit, :update, :destroy]
```

| HTTP Verb | Prefix           | URI Pattern                  | Controller#Action |
| --------- | ---------------- | ---------------------------- | ----------------- |
| GET       | post_comments    | /posts/:post_id/comments     | comments#index    |
| POST      |                  | /posts/:post_id/comments     | comments#create   |
| GET       | new_post_comment | /posts/:post_id/comments/new | comments#new      |
| GET       | edit_comment     | /comments/:id/edit           | comments#edit     |
| GET       | comment          | /comments/:id                | comments#show     |
| PATCH/PUT |                  | /comments/:id                | comments#update   |
| DELETE    |                  | /comments/:id                | comments#destroy  |
