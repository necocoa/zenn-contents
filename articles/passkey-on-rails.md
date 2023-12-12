この記事は STORES Advent Calendar 2023 22 日目の記事です。

こんにちは STORES 予約開発チームでエンジニアリングマネージャーをしています、なつめです。
昨今 Passkeys が各サービスで導入されており、勢いを感じています。

個人では 1password のパスワードマネージャーを使っています。Passkeys でのログインは ID/PW/OTA の autofill などに比べて 1step 省略される程度ですがログイン体験が良いと思っており、導入されていたらどんどん切り替えています。セキュアになる点は ID/PW と併用されているサービスがほぼで、まだ過渡期だなという感じです。

個人的に Passkeys の実際の挙動や導入する時の開発コストを知りたく、ガチャガチャ触ってみよう！ということで Passkeys 入門 Rails 編をやっていきます！

Passkeys の解説については STORES Advent Calendar 2023 2 日目に soh さんが解説していますので、ご参考までに。
https://product.st.inc/entry/2023/11/13/120000

## 前提

とりあえず触ろう！をコンセプトに解説をスキップしています。（詳しい方に丸投げ）
WebAuthn の用語をざっくり Passkeys に必要な情報、などに丸めてます。流れがわかったら解説記事などで深掘りすると良いかと思います。

## 構成

Backend は普段のアプリケーションで使っている Rails, Frontend はサクッと試せる Remix で作ってみました。それぞれディレクトリを分けつつ、ワンリポジトリで作っています。

- Backend

  - Ruby on Rails(API mode) v7.1.2

- Frontend
  - Remix v2.3.1

## Base setup

ベースのリポジトリを作成します。

```sh
mkdir passkey-on-rails
cd passkey-on-rails
git init
```

Rails と Remix はローカルで、Postgres と Redis は Docker で立てます。

```sh
docker run -d -p 5432:5432 --name passkey-on-rails-db \
-v passkey-on-rails-db:/var/lib/postgresql/data \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=password \
postgres
```

```sh
docker run -d -p 6379:6379 --name passkey-on-rails-redis \
-v passkey-on-rails-redis:/data \
redis
```

## Backend

### Base Setup

backend のディレクトリを切って、そこに `rails new` していきます。今回は API mode かつ minimal のオプションを付けて作成します。

自動で backend 配下に `git init` されるので new した後に削除しています。
`--skip-git` だと .gitignore などが作成されないため、使っていません。

```sh
mkdir backend
cd backend

bundle init
bundle add rails --version "7.1.2"

bundle exec rails _7.1.2_ new . -d postgresql --api --minimal --skip-test --skip-docker --force
rm -rf .git
```

:::details rails new logs

```sh
bundle exec rails _7.1.2_ new . -d postgresql --api --minimal --skip-test --skip-docker --force
Based on the specified options, the following options will also be activated:

  --skip-active-job [due to --minimal]
  --skip-action-mailer [due to --skip-active-job, --minimal]
  --skip-active-storage [due to --skip-active-job, --minimal]
  --skip-action-mailbox [due to --skip-active-storage, --minimal]
  --skip-action-text [due to --skip-active-storage, --minimal]
  --skip-javascript [due to --minimal, --api]
  --skip-hotwire [due to --skip-javascript, --minimal]
  --skip-action-cable [due to --minimal]
  --skip-bootsnap [due to --minimal]
  --skip-dev-gems [due to --minimal]
  --skip-jbuilder [due to --minimal]
  --skip-system-test [due to --minimal]
  --skip-asset-pipeline [due to --api]

       exist
      create  README.md
      create  Rakefile
      create  .ruby-version
      create  config.ru
      create  .gitignore
      create  .gitattributes
       force  Gemfile
         run  git init -b main from "."
Initialized empty Git repository in /Users/nat/work/rails/test/passkey-on-rails/backend/.git/
      create  app
      create  app/assets/config/manifest.js
      create  app/assets/stylesheets/application.css
      create  app/channels/application_cable/channel.rb
      create  app/channels/application_cable/connection.rb
      create  app/controllers/application_controller.rb
      create  app/helpers/application_helper.rb
      create  app/jobs/application_job.rb
      create  app/mailers/application_mailer.rb
      create  app/models/application_record.rb
      create  app/views/layouts/application.html.erb
      create  app/views/layouts/mailer.html.erb
      create  app/views/layouts/mailer.text.erb
      create  app/assets/images
      create  app/assets/images/.keep
      create  app/controllers/concerns/.keep
      create  app/models/concerns/.keep
      create  bin
      create  bin/rails
      create  bin/rake
      create  bin/setup
      create  config
      create  config/routes.rb
      create  config/application.rb
      create  config/environment.rb
      create  config/puma.rb
      create  config/environments
      create  config/environments/development.rb
      create  config/environments/production.rb
      create  config/environments/test.rb
      create  config/initializers
      create  config/initializers/assets.rb
      create  config/initializers/content_security_policy.rb
      create  config/initializers/cors.rb
      create  config/initializers/filter_parameter_logging.rb
      create  config/initializers/inflections.rb
      create  config/initializers/new_framework_defaults_7_1.rb
      create  config/initializers/permissions_policy.rb
      create  config/locales
      create  config/locales/en.yml
      create  config/master.key
      append  .gitignore
      create  config/boot.rb
      create  config/database.yml
      create  db
      create  db/seeds.rb
      create  lib
      create  lib/tasks
      create  lib/tasks/.keep
      create  lib/assets
      create  lib/assets/.keep
      create  log
      create  log/.keep
      create  public
      create  public/404.html
      create  public/422.html
      create  public/500.html
      create  public/apple-touch-icon-precomposed.png
      create  public/apple-touch-icon.png
      create  public/favicon.ico
      create  public/robots.txt
      create  tmp
      create  tmp/.keep
      create  tmp/pids
      create  tmp/pids/.keep
      create  tmp/cache
      create  tmp/cache/assets
      create  vendor
      create  vendor/.keep
      create  storage
      create  storage/.keep
      create  tmp/storage
      create  tmp/storage/.keep
      remove  app/assets
      remove  lib/assets
      remove  tmp/cache/assets
      remove  app/helpers
      remove  test/helpers
      remove  app/views
      remove  public/404.html
      remove  public/422.html
      remove  public/500.html
      remove  public/apple-touch-icon-precomposed.png
      remove  public/apple-touch-icon.png
      remove  public/favicon.ico
      remove  config/initializers/assets.rb
      remove  app/assets/config/manifest.js
      remove  app/assets/config
      remove  app/assets/stylesheets/application.css
      remove  app/jobs
      remove  app/views/layouts/mailer.html.erb
      remove  app/views/layouts/mailer.text.erb
      remove  app/mailers
      remove  test/mailers
      remove  app/javascript/channels
      remove  app/channels
      remove  test/channels
      remove  config/initializers/content_security_policy.rb
      remove  config/initializers/permissions_policy.rb
      remove  config/initializers/new_framework_defaults_7_1.rb
         run  bundle install
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Bundle complete! 5 Gemfile dependencies, 62 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
         run  bundle lock --add-platform=x86_64-linux
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Writing lockfile to /Users/nat/work/rails/test/passkey-on-rails/backend/Gemfile.lock
         run  bundle lock --add-platform=aarch64-linux
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Writing lockfile to /Users/nat/work/rails/test/passkey-on-rails/backend/Gemfile.lock
         run  bundle binstubs bundler
```

:::

### Database setup

DB の設定をします。環境変数で変更できるようにしていますが、ローカル上では fetch の後にセットしているデフォルト値を用います。

```diff yml:backend/config/database.yml
 default: &default
   adapter: postgresql
   encoding: unicode
   pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

 development:
   <<: *default
   database: backend_development
+  username: <%= ENV.fetch("DB_USERNAME") { "postgres" } %>
+  password: <%= ENV.fetch("DB_PASSWORD") { "password" } %>
+  host: <%= ENV.fetch("DB_HOSTNAME") { "localhost" } %>

 test:
   <<: *default
   database: backend_test
+  username: <%= ENV.fetch("DB_USERNAME") { "postgres" } %>
+  password: <%= ENV.fetch("DB_PASSWORD") { "password" } %>
+  host: <%= ENV.fetch("DB_HOSTNAME") { "localhost" } %>

 production:
   <<: *default
```

`bin/setup` をして、セットアップスクリプトを実行します。このタイミングで DB が作成されます。

```sh
bin/setup
== Installing dependencies ==
The Gemfile's dependencies are satisfied

== Preparing database ==

== Removing old logs and tempfiles ==

== Restarting application server ==
```

### Feature

### Base models

Model を作成します。

- User
  - ユーザー情報を保持する
- WebauthnCredential
  - Passkeys の認証情報を保持する

```sh
bin/rails g model user
```

```ruby:backend/db/migrate/*_create_users.rb
class CreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      t.string :username, null: false, index: { unique: true }
      t.string :webauthn_id, null: false

      t.timestamps
    end
  end
end
```

```
bin/rails db:migrate
== 20231125041633 CreateUsers: migrating ======================================
-- create_table(:users)
   -> 0.0174s
== 20231125041633 CreateUsers: migrated (0.0174s) =============================
```

```sh
bin/rails g model webauthn_credential
```

```ruby:backend/db/migrate/*_create_webauthn_credentials.rb
class CreateWebauthnCredentials < ActiveRecord::Migration[7.1]
  def change
    create_table :webauthn_credentials do |t|
      t.references :user, null: false, foreign_key: true
      t.string :webauthn_id, null: false, index: { unique: true }
      t.string :public_key, null: false
      t.integer :sign_count, null: false, default: 0

      t.timestamps
    end
  end
end
```

```
bin/rails db:migrate
== 20231125044030 CreateWebauthnCredentials: migrating ========================
-- create_table(:webauthn_credentials)
   -> 0.0219s
== 20231125044030 CreateWebauthnCredentials: migrated (0.0219s) ===============
```

```ruby:backend/app/models/user.rb
class User < ApplicationRecord
  has_many :webauthn_credentials, dependent: :destroy

  # ユーザー名は英数字、アンダースコア、ハイフンのみを許可
  VALID_USERNAME_REGEX = /\A[\w-]+\z/.freeze
  validates :username, presence: true,
                       length: { maximum: 50 },
                       format: { with: VALID_USERNAME_REGEX },
                       uniqueness: { case_sensitive: false }
  validates :webauthn_id, presence: true
end
```

```ruby:backend/app/models/webauthn_credential.rb
class WebauthnCredential < ApplicationRecord
  belongs_to :user

  validates :webauthn_id, presence: true, uniqueness: true
  validates :public_key, presence: true
  validates :sign_count, presence: true,
                         numericality: { only_integer: true, greater_than_or_equal_to: 0 }
end
```

#### Setup Cookies and Sessions

ブラウザと認証情報をやりとりするため Cookies と Sessions の設定をします。

今回は Redis を使って Sessions を管理します。
API mode だと Cookies も無効になっているので有効にしていきます。

session_store には `redis-session-store` という gem を使います。

```sh
bundle add redis-session-store
```

```diff ruby:backend/config/application.rb
 # 略

 module Backend
   class Application < Rails::Application
     # 略
     config.api_only = true

+    config.middleware.use ActionDispatch::Cookies
+    config.session_store :redis_session_store,
+      key: "app_sid",
+      httponly: true,
+      secure: Rails.env.production?,
+      same_site: :lax,
+      signed: true,
+      expire_after: 604800, # 1week
+      redis: {
+        key_prefix: "app:session:",
+        url: "redis://#{ENV.fetch("REDIS_HOST") { "localhost" }}:6379/0",
+      }
+    config.middleware.use config.session_store, config.session_options
   end
 end

```

Controller 側も有効にします。
ついでに、認証ユーザー、認証必須とするメソッドも作っておきます。

```diff ruby:backend/app/controllers/application_controller.rb
 class ApplicationController < ActionController::API
+  include ActionController::Cookies
+
+  private
+
+  def current_user
+    return @current_user if defined?(@current_user)
+
+    @current_user = if session[:user_id].present?
+                      User.find_by(id: session[:user_id])
+                    end
+  end
+
+  def authenticate_user!
+    unless current_user
+      render json: { status: "unauthorized" }, status: :unauthorized
+    end
+  end
 end
```

### Setup WebAuthn

Webauthn を簡単に扱える `webauthn-ruby` を使って Passkeys に必要な処理を行います。

https://github.com/cedarcode/webauthn-ruby

```sh
bundle add webauthn
touch config/initializers/webauthn.rb
```

ドキュメントを参考に設定を書きます。

```ruby:backend/config/initializers/webauthn.rb
WebAuthn.configure do |config|
  # This value needs to match `window.location.origin` evaluated by
  # the User Agent during registration and authentication ceremonies.
  config.origin = ENV.fetch("WEBAUTHN_ORIGIN") { "http://localhost:8788" }

  # Relying Party name for display purposes
  config.rp_name = 'Passkey on Rails'

  # Optionally configure a client timeout hint, in milliseconds.
  # This hint specifies how long the browser should wait for any
  # interaction with the user.
  # This hint may be overridden by the browser.
  # https://www.w3.org/TR/webauthn/#dom-publickeycredentialcreationoptions-timeout
  # config.credential_options_timeout = 120_000

  # You can optionally specify a different Relying Party ID
  # (https://www.w3.org/TR/webauthn/#relying-party-identifier)
  # if it differs from the default one.
  #
  # In this case the default would be "auth.example.com", but you can set it to
  # the suffix "example.com"
  #
  # config.rp_id = "example.com"

  # Configure preferred binary-to-text encoding scheme. This should match the encoding scheme
  # used in your client-side (user agent) code before sending the credential to the server.
  # Supported values: `:base64url` (default), `:base64` or `false` to disable all encoding.
  #
  # config.encoding = :base64url

  # Possible values: "ES256", "ES384", "ES512", "PS256", "PS384", "PS512", "RS256", "RS384", "RS512", "RS1"
  # Default: ["ES256", "PS256", "RS256"]
  #
  # config.algorithms << "ES384"
end
```

### Base controller

認証できているか確認しやすい自分の情報を返す API とログアウトの API を作ります。

```
bin/rails g controller mes show
bin/rails g controller profiles show
bin/rails g controller sessions destroy
```

```diff ruby:backend/config/routes.rb
 Rails.application.routes.draw do
+  resource :me, only: [:show]
+  resource :profile, only: [:show]
+  resource :session, only: [:destroy]
 end
```

```ruby:backend/app/controllers/mes_controller.rb
class MesController < ApplicationController
  before_action :authenticate_user!

  def show
    render json: { id: current_user.id, username: current_user.username }
  end
end
```

```ruby:backend/app/controllers/profiles_controller.rb
class ProfilesController < ApplicationController
  before_action :authenticate_user!

  def show
    render json: { username: current_user.username }
  end
end
```

```ruby:backend/app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def destroy
    reset_session
    render json: { status: "ok" }
  end
end
```

### Webauthn Controller

WebAuthn に必要な API を作成します。
ざっくり各 API の説明です。

- Attestation 登録フロー

  - Registrations アカウントの作成と Passkeys に必要な情報を保持し、返す
  - Sessions クライアントで登録した Passkeys の情報を保存し、ログイン状態にする

- Assertion ログインフロー
  - Options Passkeys に必要な情報を保持し、返す
  - Sessions クライアントで発行した Passkeys の情報を元に、ログイン状態にする

それではコントローラを作成します。

```sh
bin/rails g controller Webauthn::Attestation::Registrations create
bin/rails g controller Webauthn::Attestation::Sessions create
bin/rails g controller Webauthn::Assertion::Options create
bin/rails g controller Webauthn::Assertion::Sessions create
```

```diff ruby:backend/config/routes.rb
 Rails.application.routes.draw do
+  namespace :webauthn do
+    namespace :attestation do
+      resource :registration, only: [:create]
+      resource :session, only: [:create]
+    end
+    namespace :assertion do
+      resource :options, only: [:create]
+      resource :session, only: [:create]
+    end
+  end

   resource :me, only: [:show]
   resource :profile, only: [:show]
   resource :session, only: [:destroy]
 end
```

#### `POST /webauthn/attestation/registration`

ユーザーが username を入力して Sign up ボタンを押された時に POST する API です。
今回は ID/PW のログイン方法は用意しないので受け取った username でそのままユーザーを作成し、Passkeys の情報を返します。

`WebAuthn::Credential.options_for_create` で Passkeys に必要な情報を用意します。

```ruby
session[:webauthn_challenge] = creation_options.challenge
session[:webauthn_user_id] = user.id
```

で Challenge と、この認証情報がどの user と紐づくかをセッションに保存します。

```ruby:backend/app/controllers/webauthn/attestation/registrations_controller.rb
class Webauthn::Attestation::RegistrationsController < ApplicationController
  def create
    user = User.new(create_params)

    unless user.update(webauthn_id: WebAuthn.generate_user_id)
      return render json: { errors: user.errors.full_messages }, status: :unprocessable_entity
    end

    creation_options = WebAuthn::Credential.options_for_create(
      user: { id: user.webauthn_id, name: user.username },
      exclude: user.webauthn_credentials.pluck(:webauthn_id)
    )

    # Store the newly generated challenge somewhere so you can have it
    # for the verification phase.
    session[:webauthn_challenge] = creation_options.challenge
    session[:webauthn_user_id] = user.id

    # Send `creation_options` back to the browser, so that they can be used
    # to call `navigator.credentials.create({ "publicKey": creationOptions })`
    render json: creation_options
  end

  private

  def create_params
    params.require(:registration).permit(:username)
  end
end
```

#### `POST /webauthn/attestation/session`

クライアント側で認証機から生成された Passkeys の情報を受け取り、それらの情報を Challenge とともに検証します。

```ruby
webauthn_credential = WebAuthn::Credential.from_create(create_params)
webauthn_credential.verify(session[:webauthn_challenge])
```

ここでセッションに保存した Challenge を用いて、検証しています。

```ruby
user = User.find(session[:webauthn_user_id])
user.webauthn_credentials.create!(
  webauthn_id: webauthn_credential.id,
  public_key: webauthn_credential.public_key,
  sign_count: webauthn_credential.sign_count
)
session[:user_id] = user.id
```

検証が通ったら、WebauthnCredentials に情報を保存し、ログイン状態にします。

```ruby:backend/app/controllers/webauthn/attestation/sessions_controller.rb
class Webauthn::Attestation::SessionsController < ApplicationController
  def create
    if session[:webauthn_challenge].nil?
      return render json: { error: "No challenge found in session" }, status: :unprocessable_entity
    end

    webauthn_credential = WebAuthn::Credential.from_create(create_params)

    begin
      webauthn_credential.verify(session[:webauthn_challenge])

      user = User.find(session[:webauthn_user_id])

      user.webauthn_credentials.create!(
        webauthn_id: webauthn_credential.id,
        public_key: webauthn_credential.public_key,
        sign_count: webauthn_credential.sign_count
      )

      session[:user_id] = user.id
      session.delete(:webauthn_challenge)
      session.delete(:webauthn_user_id)
    rescue WebAuthn::VerificationError, WebAuthn::Error => e
      return render json: { error: e.message }, status: :unprocessable_entity
    end

    render json: { status: "ok" }
  end

  private

  def create_params
    params.require(:session).permit(
      :id, :rawId, :type, :authenticatorAttachment,
      clientExtensionResults: {},
      response: [:attestationObject, :clientDataJSON, { transports: [] }]
    )
  end
end
```

#### `POST /webauthn/assertion/option`

Sign in 画面を開いた際に、Passkeys に必要な情報を取得するための API です。
Challenge をセッションに保存し、Passkeys に必要な情報を返しています。

```ruby:backend/app/controllers/webauthn/assertion/options_controller.rb
class Webauthn::Assertion::OptionsController < ApplicationController
  def create
    options = WebAuthn::Credential.options_for_get
    session[:webauthn_challenge] = options.challenge
    render json: options
  end
end
```

#### `POST /webauthn/assertion/session`

Sign in ボタンを押すと Passkeys(認証器) が出てきます 。認証ができるとその Passkeys の情報をこちらの API に POST し、検証後ログイン状態にします。

```ruby
webauthn_credential = WebAuthn::Credential.from_get(create_params)
stored_credential = WebauthnCredential.find_by!(webauthn_id: webauthn_credential.id)
```

クライアントから送られた情報をもとに WebauthnCredential を引きます。

```ruby
webauthn_credential.verify(
  session[:webauthn_challenge],
  public_key: stored_credential.public_key,
  sign_count: stored_credential.sign_count
)
```

セッションに保存した Challenge とレコードに保存した署名を検証しています。
検証が完了したらログイン状態にします。

```ruby:backend/app/controllers/webauthn/assertion/sessions_controller.rb
class Webauthn::Assertion::SessionsController < ApplicationController
  def create
    webauthn_credential = WebAuthn::Credential.from_get(create_params)
    stored_credential = WebauthnCredential.find_by!(webauthn_id: webauthn_credential.id)

    begin
      webauthn_credential.verify(
        session[:webauthn_challenge],
        public_key: stored_credential.public_key,
        sign_count: stored_credential.sign_count
      )

      stored_credential.update!(sign_count: webauthn_credential.sign_count)

      session[:user_id] = stored_credential.user.id
      session.delete(:webauthn_challenge)
    rescue WebAuthn::SignCountVerificationError, WebAuthn::Error => e
      # Cryptographic verification of the authenticator data succeeded, but the signature counter was less then or equal
      # to the stored value. This can have several reasons and depending on your risk tolerance you can choose to fail or
      # pass authentication. For more information see https://www.w3.org/TR/webauthn/#sign-counter
      return render json: { error: e.message }, status: :unprocessable_entity
    end

    render json: { status: "ok" }
  end

  private

  def create_params
    params.require(:session).permit(
      :id, :rawId, :type, :authenticatorAttachment,
      clientExtensionResults: {},
      response: [:clientDataJSON, :authenticatorData, :signature, :userHandle]
    )
  end
end
```

Backend の実装は以上になります。

## Frontend

続いては Frontend を作っていきます。
`create-remix` の cloudflare-pages テンプレートを使って Remix をセットアップします。

https://remix.run/docs/en/main/guides/templates#official-templates

```sh
npx create-remix@2.3.1 --template remix-run/remix/templates/cloudflare-pages
```

対話形式で以下の内容にします。

- frontend ディレクトリ配下に作成
- Git は不要
- npm install する

:::details create-remix logs

```sh
npx create-remix@2.3.1 --template remix-run/remix/templates/cloudflare-pages
Need to install the following packages:
  create-remix@2.3.1
Ok to proceed? (y)

 remix   v2.3.1 💿 Let's build a better website...

   dir   Where should we create your new project?
         frontend

      ◼  Template: Using remix-run/remix/templates/cloudflare-pages...
      ✔  Template copied

   git   Initialize a new git repository?
         No

  deps   Install dependencies with npm?
         Yes

      ✔  Dependencies installed

  done   That's it!

         Enter your project directory using cd ./frontend
         Check out README.md for development and deploy instructions.

         Join the community at https://rmx.as/discord

```

:::

以下は frontend ディレクトリで作業します。

```sh
cd frontend
```

Node.js のバージョンが 18.0.0 になっているので、最新にします。

```diff :frontend/.node-version
- 18.0.0
+ 20.10.0
```

API の向き先を指定するために ENV ファイルを作成します。
cloudflare workers 上で環境変数を読み込む `.dev.vars` を作成します。

```sh
touch .example.dev.vars
```

```sh:frontend/.example.dev.vars
API_URL="http://localhost:3000"
```

```sh
cp .example.dev.vars .dev.vars
```

### CSS

Tailwind CSS ベースのコンポーネントである Daisy UI を使ってみます。

```sh
npm install --save-dev tailwindcss daisyui
```

```sh
touch tailwind.config.ts
touch app/tailwind.css
```

```js:frontend/tailwind.config.ts
import type { Config } from "tailwindcss";

export default {
  content: ["./app/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [require("daisyui")],
} satisfies Config;
```

```css:frontend/app/tailwind.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

stylesheet の読み込みは後で変更します。

### Webauthn-json

`@github/webauthn-json` はブラウザの WebAuthn API を取り扱いやすくするライブラリです。
自前で書くと ArrayBuffer に変換する処理や型情報も用意する必要があるので、楽するために導入します。

https://github.com/github/webauthn-json

```sh
npm install @github/webauthn-json
```

### Pages

UI, データ取得, Passkeys の処理を追加していきます。
半分はただの UI 部分なので適宜解説を挟みつつ、ざーっと書いていきます。

#### Base Pages

ログイン・非ログイン状態がわかりやすいように最低限の UI を作ります。

API URL を Context から取得するメソッドです。

```ts:frontend/app/utils/api.server.ts
import type { AppLoadContext } from "@remix-run/cloudflare";
import type { Env } from "~/global";

export const getApiUrl = (context: AppLoadContext) => {
  const env = context.env as Env;
  return env.API_URL;
};
```

```ts:auth.d.ts
export type AuthUser = {
  id: string;
  username: string;
};
```

認証状態を取得する API です。

```ts:frontend/app/utils/auth.server.ts
import type { AppLoadContext } from "@remix-run/cloudflare";
import { getApiUrl } from "./api.server";
import type { AuthUser } from "./auth";

export const authentication = async (request: Request, context: AppLoadContext) => {
  const apiUrl = getApiUrl(context);
  const response = await fetch(`${apiUrl}/me`, {
    method: "GET",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      Cookie: request.headers.get("Cookie") || "",
    },
  });

  if (!response.ok) {
    return { user: null };
  }
  const user = (await response.json()) as AuthUser;
  return { user };
};

export const isAuthenticated = async (request: Request, context: AppLoadContext) => {
  const apiUrl = getApiUrl(context);
  const response = await fetch(`${apiUrl}/me`, {
    method: "GET",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      Cookie: request.headers.get("Cookie") || "",
    },
  });
  return response.ok;
};
```

ルートのレイアウトです。

```ts:frontend/app/root.tsx
import { Links, LiveReload, Meta, Outlet, Scripts, ScrollRestoration, useLoaderData } from "@remix-run/react";
import { json, type LinksFunction, type LoaderFunctionArgs } from "@remix-run/cloudflare";
import stylesheet from "~/tailwind.css";
import { authentication } from "./utils/auth.server";
import { Header } from "./compornents/Header";

export const links: LinksFunction = () => [{ rel: "stylesheet", href: stylesheet }];

export const loader = async ({ request, context }: LoaderFunctionArgs) => {
  const { user } = await authentication(request, context);
  return json({ user });
};

export default function App() {
  const { user } = useLoaderData<typeof loader>();

  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <Meta />
        <Links />
      </head>
      <body>
        <div className="grid bg-base-200" style={{ minHeight: "100lvh", gridTemplateRows: "auto 1fr auto" }}>
          <Header user={user} />
          <main>
            <Outlet />
          </main>
        </div>
        <ScrollRestoration />
        <Scripts />
        <LiveReload />
      </body>
    </html>
  );
}
```

ヘッダにはログイン中は username を表示します。

```ts:frontend/app/compornents/Header.tsx
import { Form, Link } from "@remix-run/react";
import type { SerializeFrom } from "@remix-run/cloudflare";
import type { AuthUser } from "~/utils/auth";

type Props = {
  user: SerializeFrom<AuthUser> | null;
};

export function Header({ user }: Props) {
  return (
    <header className="navbar bg-base-100">
      <div className="flex-1">
        <Link to={"/"} className="btn btn-ghost text-xl normal-case">
          Passkey App
        </Link>
      </div>
      {user && (
        <div className="flex-none">
          <div className="dropdown dropdown-end">
            <div tabIndex={0} role="button" className="btn btn-ghost normal-case">
              {user.username}
            </div>
            <ul
              tabIndex={0}
              className="menu dropdown-content rounded-box menu-sm z-10 mt-3 w-52 bg-base-100 p-2 shadow"
            >
              <li>
                <Link to={"/profile"}>Profile</Link>
              </li>
              <li>
                <Form method="post" action="/session/destroy">
                  <button type="submit">Logout</button>
                </Form>
              </li>
            </ul>
          </div>
        </div>
      )}
    </header>
  );
}
```

トップページは、非ログイン中は Sign up, Sign in、ログイン中は Profile のボタンを表示します。

```ts:frontend/app/routes/_index.tsx
import { Link, useLoaderData } from "@remix-run/react";
import { json, type LoaderFunctionArgs, type MetaFunction } from "@remix-run/cloudflare";
import { isAuthenticated } from "~/utils/auth.server";

export const meta: MetaFunction = () => {
  return [{ title: "Let's try passkey" }];
};

export const loader = async ({ request, context }: LoaderFunctionArgs) => {
  const authenticated = await isAuthenticated(request, context);
  return json({ authenticated });
};

export default function Index() {
  const { authenticated } = useLoaderData<typeof loader>();

  return (
    <div className="hero h-full">
      <div className="hero-content w-screen">
        <div className="card w-96 bg-neutral text-neutral-content">
          <div className="card-body items-center text-center">
            <h2 className="card-title">Passkey App!</h2>
            <div className="card-actions justify-end">
              {authenticated ? (
                <>
                  <div>
                    <Link to={"/profile"} className="btn btn-primary normal-case">
                      Profile
                    </Link>
                  </div>
                </>
              ) : (
                <>
                  <div>
                    <Link to={"/sign-up"} className="btn btn-primary normal-case">
                      Sign up
                    </Link>
                  </div>
                  <div>
                    <Link to={"/sign-in"} className="btn btn-secondary normal-case">
                      Sign in
                    </Link>
                  </div>
                </>
              )}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

username だけ表示するプロフィールページです。

```ts:frontend/app/routes/profile.tsx
import { Form, useLoaderData } from "@remix-run/react";
import { json, type LoaderFunctionArgs, type MetaFunction } from "@remix-run/cloudflare";
import { getApiUrl } from "~/utils/api.server";

export const meta: MetaFunction = () => {
  return [{ title: "Profile" }];
};

type Profile = {
  username: string;
};

export const loader = async ({ request, context }: LoaderFunctionArgs) => {
  const apiUrl = getApiUrl(context);
  const response = await fetch(`${apiUrl}/profile`, {
    method: "GET",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      Cookie: request.headers.get("Cookie") || "",
    },
  });

  if (!response.ok) throw json({}, { status: response.status });
  const profile = (await response.json()) as Profile;
  return json({ profile });
};

export default function Profile() {
  const { profile } = useLoaderData<typeof loader>();

  return (
    <div className="hero h-full">
      <div className="hero-content w-screen">
        <div className="card w-96 bg-neutral text-neutral-content">
          <Form method="post" action="/session/destroy">
            <div className="card-body">
              <h2 className="card-title">Profile</h2>
              <p>Name: {profile.username}</p>
              <div className="card-actions justify-end">
                <button className="btn btn-neutral">Logout</button>
              </div>
            </div>
          </Form>
        </div>
      </div>
    </div>
  );
}
```

ログアウトボタンのアクションです。

```ts:frontend/app/routes/session.destroy.tsx
import { redirect, type ActionFunctionArgs } from "@remix-run/cloudflare";
import { getApiUrl } from "~/utils/api.server";

export const action = async ({ context, request }: ActionFunctionArgs) => {
  const apiUrl = getApiUrl(context);
  const response = await fetch(`${apiUrl}/session`, {
    method: "DELETE",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      Cookie: request.headers.get("Cookie") || "",
    },
  });
  const cookie = response.headers.get("Set-Cookie") || "";
  return redirect("/", { headers: { "Set-Cookie": cookie } });
};
```

### Sign up, Sign in Pages

ログイン状態ならリダイレクトする処理を噛ましています。

```ts:frontend/app/routes/_auth.tsx
import { redirect, type LoaderFunctionArgs } from "@remix-run/cloudflare";
import { isAuthenticated } from "~/utils/auth.server";

export const loader = async ({ request, context }: LoaderFunctionArgs) => {
  if (await isAuthenticated(request, context)) {
    throw redirect("/profile");
  }
  return null;
};
```

Sign up 画面です。

username を入力し登録ボタンを押すと、サーバ側から Passkeys の情報が返ってきます。

```ts
const createCredential = useCallback(async () => {
  if (actionData?.credential == null) {
    return
  }

  const credentialWithAttestation = await create(
    actionData.credential as CredentialCreationOptionsJSON,
  )
  const formData = new FormData()
  formData.append('credential', JSON.stringify(credentialWithAttestation))
  submit(formData, {
    action: '/sign-up/passkey',
    method: 'post',
  })
}, [actionData, submit])

useEffect(() => {
  createCredential()
}, [actionData, createCredential])
```

その情報を useEffect でキャッチし `create(actionData.credential)` を実行するとブラウザ上で Passkeys(認証器) が出てきます。

認証が完了すると後続の処理が行われ、アカウント登録が完了する流れになっています。

```ts:frontend/app/routes/_auth.sign-up.tsx
import { useCallback, useEffect } from "react";
import { Form, useActionData, useSubmit } from "@remix-run/react";
import { json } from "@remix-run/cloudflare";
import type { ActionFunctionArgs, MetaFunction } from "@remix-run/cloudflare";
import { create } from "@github/webauthn-json";
import type { CredentialCreationOptionsJSON } from "@github/webauthn-json/browser-ponyfill";
import type { PublicKeyCredentialCreationOptionsJSON } from "node_modules/@github/webauthn-json/dist/types/basic/json";
import { getApiUrl } from "~/utils/api.server";

export const meta: MetaFunction = () => {
  return [{ title: "Sign Up" }];
};

export const action = async ({ request, context }: ActionFunctionArgs) => {
  const apiUrl = getApiUrl(context);

  const formData = await request.formData();
  const username = formData.get("username");

  const response = await fetch(`${apiUrl}/webauthn/attestation/registration`, {
    method: "POST",
    body: JSON.stringify({ username }),
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      Cookie: request.headers.get("Cookie") || "",
    },
  });
  const cookie = response.headers.get("Set-Cookie") || "";
  if (!response.ok) {
    const error = await response.json();
    return json({ credential: null, error }, { status: response.status, headers: { "Set-Cookie": cookie } });
  }

  const options = (await response.json()) as PublicKeyCredentialCreationOptionsJSON;
  return json({ credential: { publicKey: options } }, { status: response.status, headers: { "Set-Cookie": cookie } });
};

export default function SignUp() {
  const submit = useSubmit();
  const actionData = useActionData<typeof action>();

  const createCredential = useCallback(async () => {
    if (actionData?.credential == null) {
      return;
    }

    const credentialWithAttestation = await create(actionData.credential as CredentialCreationOptionsJSON);
    const formData = new FormData();
    formData.append("credential", JSON.stringify(credentialWithAttestation));
    submit(formData, {
      action: "/sign-up/passkey",
      method: "post",
    });
  }, [actionData, submit]);

  useEffect(() => {
    createCredential();
  }, [actionData, createCredential]);

  return (
    <div className="hero h-full">
      <div className="hero-content w-screen">
        <div className="card w-96 bg-neutral text-neutral-content">
          <Form method="post">
            <div className="card-body">
              <h2 className="card-title">Sign Up</h2>
              <div className="form-control">
                <label htmlFor="username" className="label label-text text-neutral-content">
                  Username
                </label>
                <input
                  id="username"
                  type="username"
                  name="username"
                  placeholder="Username"
                  className="input input-bordered"
                  required
                />
              </div>
              <div className="card-actions">
                <button type="submit" className="btn btn-primary w-full">
                  Sign up
                </button>
              </div>
            </div>
          </Form>
        </div>
      </div>
    </div>
  );
}
```

```ts:frontend/app/routes/_auth.sign-up.passkey.tsx
import { json, redirect, type ActionFunctionArgs } from "@remix-run/cloudflare";
import { getApiUrl } from "~/utils/api.server";

export const action = async ({ request, context }: ActionFunctionArgs) => {
  const apiUrl = getApiUrl(context);

  const formData = await request.formData();
  const credential = formData.get("credential");

  const response = await fetch(`${apiUrl}/webauthn/attestation/session`, {
    method: "POST",
    body: credential,
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      Cookie: request.headers.get("Cookie") || "",
    },
  });
  const cookie = response.headers.get("Set-Cookie") || "";
  if (!response.ok) {
    const error = await response.json();
    throw json({ error }, { status: response.status, headers: { "Set-Cookie": cookie } });
  }

  return redirect("/profile", {
    status: 303,
    headers: { "Set-Cookie": cookie },
  });
};
```

Sigin in 画面です。

```ts:frontend/app/routes/_auth.sign-in.tsx
import type { FormEventHandler } from "react";
import { Form, useLoaderData, useSubmit } from "@remix-run/react";
import { json, type LoaderFunctionArgs, type MetaFunction } from "@remix-run/cloudflare";
import { get } from "@github/webauthn-json";
import type {
  CredentialRequestOptionsJSON,
  PublicKeyCredentialRequestOptionsJSON,
} from "node_modules/@github/webauthn-json/dist/types/basic/json";
import { getApiUrl } from "~/utils/api.server";

export const meta: MetaFunction = () => {
  return [{ title: "Sign In" }];
};

export const loader = async ({ request, context }: LoaderFunctionArgs) => {
  const apiUrl = getApiUrl(context);
  const response = await fetch(`${apiUrl}/webauthn/assertion/options`, {
    method: "POST",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      "Set-Cookie": request.headers.get("Cookie") || "",
    },
  });
  const cookie = response.headers.get("Set-Cookie") || "";
  if (!response.ok) {
    throw json(
      { message: "Failed to fetch credential options" },
      {
        status: response.status,
        headers: { "Set-Cookie": cookie },
      }
    );
  }

  const options = (await response.json()) as PublicKeyCredentialRequestOptionsJSON;
  return json(
    { credential: { publicKey: options } },
    {
      status: 200,
      headers: { "Set-Cookie": cookie },
    }
  );
};

export default function SignIn() {
  const submit = useSubmit();
  const { credential } = useLoaderData<typeof loader>();

  const onSubmit: FormEventHandler = async (event) => {
    event.preventDefault();
    const credentialWithAssertion = await get(credential as CredentialRequestOptionsJSON);
    const formData = new FormData();
    formData.append("credential", JSON.stringify(credentialWithAssertion));
    submit(formData, { action: "/sign-in/passkey", method: "post" });
  };

  return (
    <div className="hero h-full">
      <div className="hero-content w-screen">
        <div className="card w-96 bg-neutral text-neutral-content">
          <Form method="post" onSubmit={onSubmit}>
            <div className="card-body">
              <h2 className="card-title">Sign In with a passkey</h2>
              <div className="card-actions">
                <button type="submit" className="btn btn-primary w-full">
                  Sign in with a passkey
                </button>
              </div>
            </div>
          </Form>
        </div>
      </div>
    </div>
  );
}
```

```ts:frontend/app/routes/_auth.sign-in.passkey.tsx
import { json, redirect, type ActionFunctionArgs } from "@remix-run/cloudflare";
import { getApiUrl } from "~/utils/api.server";

export const action = async ({ request, context }: ActionFunctionArgs) => {
  const apiUrl = getApiUrl(context);

  const formData = await request.formData();
  const credential = formData.get("credential");

  const response = await fetch(`${apiUrl}/webauthn/assertion/session`, {
    method: "POST",
    body: credential,
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      Cookie: request.headers.get("Cookie") || "",
    },
  });
  const cookie = response.headers.get("Set-Cookie") || "";
  if (!response.ok) {
    const error = await response.json();
    throw json({ error }, { status: response.status, headers: { "Set-Cookie": cookie } });
  }

  return redirect("/profile", {
    status: 303,
    headers: { "Set-Cookie": cookie },
  });
};
```

:::details 最終的なファイルツリー。

```sh
ls -T
./
├── backend/
│  ├── .gitattributes
│  ├── .gitignore
│  ├── .ruby-version
│  ├── app/
│  │  ├── controllers/
│  │  │  ├── application_controller.rb
│  │  │  ├── mes_controller.rb
│  │  │  ├── profiles_controller.rb
│  │  │  └── sessions_controller.rb
│  │  └── models/
│  │     ├── application_record.rb
│  │     ├── user.rb
│  │     └── webauthn_credential.rb
│  ├── bin/
│  │  ├── bundle*
│  │  ├── rails*
│  │  ├── rake*
│  │  └── setup*
│  ├── config/
│  │  ├── application.rb
│  │  ├── boot.rb
│  │  ├── credentials.yml.enc
│  │  ├── database.yml
│  │  ├── environment.rb
│  │  ├── environments/
│  │  │  ├── development.rb
│  │  │  ├── production.rb
│  │  │  └── test.rb
│  │  ├── initializers/
│  │  │  ├── cors.rb
│  │  │  ├── filter_parameter_logging.rb
│  │  │  ├── inflections.rb
│  │  │  └── webauthn.rb
│  │  ├── locales/
│  │  │  └── en.yml
│  │  ├── master.key
│  │  ├── puma.rb
│  │  └── routes.rb
│  ├── config.ru
│  ├── db/
│  │  ├── migrate/
│  │  │  ├── 20231125041633_create_users.rb
│  │  │  └── 20231125044030_create_webauthn_credentials.rb
│  │  ├── schema.rb
│  │  └── seeds.rb
│  ├── Gemfile
│  ├── Gemfile.lock
│  ├── lib/
│  ├── log/
│  ├── public/
│  │  └── robots.txt
│  ├── Rakefile
│  ├── README.md
│  ├── storage/
│  ├── tmp/
│  └── vendor/
└── frontend/
   ├── .dev.vars
   ├── .eslintrc.cjs
   ├── .example.dev.vars
   ├── .gitignore
   ├── .node-version
   ├── app/
   │  ├── compornents/
   │  │  └── Header.tsx
   │  ├── entry.client.tsx
   │  ├── entry.server.tsx
   │  ├── global.d.ts
   │  ├── root.tsx
   │  ├── routes/
   │  │  ├── _auth.sign-in.passkey.tsx
   │  │  ├── _auth.sign-in.tsx
   │  │  ├── _auth.sign-up.passkey.tsx
   │  │  ├── _auth.sign-up.tsx
   │  │  ├── _auth.tsx
   │  │  ├── _index.tsx
   │  │  ├── profile.tsx
   │  │  └── session.destroy.tsx
   │  ├── tailwind.css
   │  └── utils/
   │     ├── api.server.ts
   │     └── auth.server.ts
   ├── functions/
   ├── package-lock.json
   ├── package.json
   ├── public/
   │  ├── _headers
   │  ├── _routes.json
   │  └── favicon.ico
   ├── README.md
   ├── remix.config.js
   ├── remix.env.d.ts
   ├── server.ts
   ├── tailwind.config.ts
   └── tsconfig.json
```

:::
