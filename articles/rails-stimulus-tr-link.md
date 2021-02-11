---
title: '[Rails]Stimulusを使ってtableの行全体<tr>をリンクにする(jQuery不要)'
emoji: '👏'
type: 'tech'
topics: ['rails', 'ruby', 'stimulus']
published: false
---

テーブルの行を押すと show に飛ぶ実装をしたいと思いました。

調べてみると jQuery での記事がたくさんでてきましたが、できれば jQuery は一切使いたくないですよね。

せっかくならと Stimulus を使ってみることにしました。

## 環境

Ruby: 2.7.2
Rails: 6.1.1

webpacker を使っていない方は先に webpacker のインストールから。

```sh
bin/rails webpacker:install
```

## Stimulus について

Stimulus は JS フレームワークです。
Rails を開発している Basecamp が作っています。

イメージとしては、jQuery で行うような JavaScript での挙動を簡単に実装できます。
特に DOM 操作のために id を指定したりする必要がなくなるので、コードの共通化などがシンプルになります。

チュートリアルも量が少ないので実際に見てみるとイメージが湧くかと。
https://stimulus.hotwire.dev/

## Stimulus のセットアップ

webpacker 経由で stimulus をインストールします。

```sh
bin/rails webpacker:install:stimulus
```

application.js に自動で `import "controllers"` が挿入されます。

```js:app/javascript/packs/application.js
...
import "controllers"
...
```

view 側で `application.js` を読み込んでない場合は読み込みましょう。

```erb:app/views/layouts/application.html.erb
...
<%= javascript_pack_tag 'application' %>
...
```

これで `application.js` を読み込んでいる erb 上で Stimulus が使えるようになりました。

## Stimulus を使って実装する

### <tr> に Stimulus で使うコントローラーをセットする

Stimulus を使う時は HTML 要素に `data-controller` と `data-action` をセットすればすぐに使えます。

```erb:index.html.erb
<tr
  data-controller="href"
  data-action="click->href#toHref"
  data-href="<%= obj_path(obj) %>"
  ">
</tr>
```

細かく解説していきます。

##### `data-controller="href"`

`data-controller` には JS コントローラー名をセットします。

今回の場合 `href` という名前の JS コントローラーをセットしました。

##### `data-action="click->href#toHref"`

`data-action` にはイベントハンドラーと実行するメソッドを指定します。
イベントハンドラーを指定しない場合、要素に対しての全てのイベントで実行されます。

コントローラー名とメソッド名を `href#toHref` この形でセットします。

今回はクリック時に遷移したいので先頭に `click->` をつけます。

##### `data-href="<%= obj_path(obj) %>"`

こちらは Stimulus とは関係ありません。
通常の DOM 操作で値を取得するために、data 属性を作ります。

今回は `data-href` にパスをセットします。

### JavaScript 部分を実装する

#### コントローラーファイル作成

<tr>要素をクリックした際に、指定パスに遷移する実装をします。
Stimulus で使う JavaScript のファイルは `app/javascript/controllers` 配下に置きます。
命名規則は `xxx_controller.js` です。
今回は `data-controller="href"` なので `href_controller.js` ファイルを作ります。

```sh
touch app/javascript/controllers/href_controller.js
```

#### メソッド作成

`Stimulus` 内にある `Controller` を継承して使います。

```js:app/javascript/controllers/href_controller.js
import { Controller } from 'stimulus'

export default class extends Controller {
}
```

Class の中にメソッドを追加していきます。

```js:app/javascript/controllers/href_controller.js
import { Controller } from 'stimulus'

export default class extends Controller {
  toHref(event) {
    event.preventDefault()
    const href = event.currentTarget.dataset['href']
    window.location.href = href
  }
}
```

これで<tr>要素をクリックすると `toHref` が実行され、指定したパスに遷移できます。
