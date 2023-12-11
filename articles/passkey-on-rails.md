## Setup

## Frontend

```sh
npx create-remix@latest --template remix-run/remix/templates/cloudflare-pages
```

## Base setup

```sh
mkdir passkey-on-rails
cd passkey-on-rails
git init
```

```sh
docker run -d -p 5432:5432 --name passkey-on-rails-db \
-v passkey-on-rails-db:/var/lib/postgresql/data:cached \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=password \
postgres
```

```sh
docker run -d -p 6379:6379 --name passkey-on-rails-redis \
-v passkey-on-rails-redis:/data:cached \
redis
```

## Backend

### Setup

```sh
mkdir backend
cd backend

bundle init
bundle add rails --version "~> 7.1.2"

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

Redis を使って Sessions を管理します。
API mode だと Cookies も無効になっているので、有効にしていきます。

今回は `redis-session-store` を使います。

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
+      serializer: :json,
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

### Setup Webauthn

https://github.com/cedarcode/webauthn-ruby

```sh
bundle add webauthn
touch config/initializers/webauthn.rb
```

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

簡素な認証ユーザー取得と認証必須のメソッドを作成します。

```diff ruby:backend/app/controllers/application_controller.rb
 class ApplicationController < ActionController::API
   include ActionController::Cookies
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

```
bin/rails g controller mes show
bin/rails g controller profiles show
bin/rails g controller sessions destroy
```

```diff ruby:backend/config/routes.rb
 Rails.application.routes.draw do
   # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

   # Reveal health status on /up that returns 200 if the app boots with no exceptions, otherwise 500.
   # Can be used by load balancers and uptime monitors to verify that the app is live.
   get "up" => "rails/health#show", as: :rails_health_check

+  resource :me, only: [:show]
+  resource :profile, only: [:show]
+  resource :session, only: [:destroy]
 end

```

## Frontend

https://remix.run/docs/en/main/guides/templates#official-templates

```sh
npx create-remix@2.3.1 --template remix-run/remix/templates/cloudflare-pages
```

- frontend というディレクトリで作成します。
- Git は不要です
- npm install はしましょう

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

以下は frontend ディレクトリで作業します。

```sh
cd frontend
```

```diff .node-version
- 18.0.0
+ 20.10.0
```

### CSS

```
npm install --save-dev tailwindcss daisyui
```

```
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

```
npm install @github/webauthn-json
```
