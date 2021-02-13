---
title: '[Rails]Stimulusを使ってtableの行全体<tr>をリンクにする(jQuery不要)'
emoji: '📌'
type: 'tech'
topics: ['rails', 'ruby', 'stimulus']
published: true
---

テーブルの行を押すと show に飛ぶ実装をしたいと思いました。

調べてみると jQuery での記事がたくさんでてきましたが、できれば jQuery は一切使いたくない。せっかくならと Stimulus を使ってみることにしました。

#### 完成イメージ

![tableのtr要素クリック](https://storage.googleapis.com/zenn-user-upload/q3pkxofvr704lbd63dtor4l5t4a1)

## 環境

Ruby: 2.7.2
Rails: 6.1.1

webpacker を使っていない方は先に webpacker のインストールから。

```sh
bin/rails webpacker:install
```

## Stimulus について

Stimulus は JS フレームワークです。Rails を開発している Basecamp が作っています。

イメージとしては、jQuery で行うような DOM 操作などを簡単に実装できます。
特に DOM 操作のために id 属性付ける必要がなくなるので、コードの共通化などがシンプルになります。

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

### `<tr>` に Stimulus で使うコントローラーをセットする

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

##### 1. `data-controller="href"`

`data-controller` には JS コントローラー名をセットします。

今回の場合 `href` という名前の JS コントローラーをセットしました。

##### 2. `data-action="click->href#toHref"`

`data-action` にはイベントハンドラーと実行するメソッドを指定します。イベントハンドラーを指定しない場合、要素に対しての全てのイベントで実行されます。

コントローラー名とメソッド名を `href#toHref` というでセットします。

今回はクリック時に遷移したいので先頭に `click->` をつけます。

##### 3. `data-href="<%= obj_path(obj) %>"`

こちらは Stimulus とは関係ありません。
通常の DOM 操作で値を取得するために、data 属性を作ります。

今回は `data-href` にパスをセットします。

### JavaScript 部分を実装する

#### コントローラーファイル作成

`<tr>`要素をクリックした際に、指定パスに遷移する実装をします。

Stimulus で使う JavaScript のファイルは `app/javascript/controllers` 配下に置きます。命名規則は `xxx_controller.js` です。

今回は `data-controller="href"` なので `href_controller.js` ファイルを作ります。

```sh
touch app/javascript/controllers/href_controller.js
```

#### メソッド作成

`stimulus` 内にある `Controller` を継承して使います。

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

## 最終的なファイル構成

最終的に出来上がったコードを上げておきます。以下の機能も追加しています。

1. `<tr>`要素をリンクタグと同じ挙動にした
   1. フォーカス対象にする
   2. ホバーした時ポインターにする
   3. フォーカス時に Enter を押すと遷移する
2. そのままだと `<tr>` 子要素にあるリンクなどが動作しないので、`event#stopPropagation` を作成

```erb:index.html.erb
<table>
  <thead>
    <tr>
      <th>ID</th>
      <th>編集</th>
    </tr>
  </thead>
  <tbody>
    <% @posts.each do |post| %>
      <tr
        data-controller="href"
        data-action="click->href#toHref keydown->href#enterToHref"
        data-href="<%= post_path(post) %>"
        tabindex="0"
        role="link"
        style="cursor: pointer;">
        <th><%= post.id %></th>
        <td>
          <%= link_to post_path(post), data: { controller: 'event', action: 'event#stopPropagation' } do %>
            <i class="fas fa-edit"></i>
          <% end %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
```

```js:app/javascript/controllers/href_controller.js
import { Controller } from 'stimulus'

export default class extends Controller {
  // 親要素のdata-href属性にあるパスに遷移する
  toHref(event) {
    event.preventDefault()
    const href = event.currentTarget.dataset['href']
    window.location.href = href
  }

  // フォーカスした要素上でエンターを押した時、遷移する
  enterToHref(event) {
    if (event.target !== event.currentTarget) return

    if (event.key === 'Enter') {
      this.toHref(event)
    }
  }
}

```

```js:app/javascript/controllers/event_controller.js
import { Controller } from 'stimulus'

export default class extends Controller {
  // 親要素のイベントを発生させたくない時に使う
  // <button data-controller="event" data-action="event#stopPropagation">
  stopPropagation(event) {
    event.stopPropagation()
  }
}
```

## まとめ

Stimulus は副作用が少なくすぐに導入できると思います。既存の jQuery を置き換えたり、処理の共通化などとても便利そうです。`partial collection`のときに起きがちな id をユニークにする処理なども一切不要になるのがとても嬉しいですね。

React/Vue.js を使うまでもない JavaScript を書く時に積極的に使っていこうと思います。
