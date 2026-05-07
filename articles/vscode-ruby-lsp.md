---
title: 'Shopify製Ruby LSPを導入し開発環境を改善する[VS Code]'
emoji: '💎'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['rails', 'ruby', 'vscode']
published: true
---

みなさんは VS Code・Ruby で開発するとき、Extention は何を入れていますか？
LSP である rebornix.Ruby を入れている方は多いかと思います。

ですが rebornix.Ruby は Deprecated になっており Shopify の ruby-lsp を使うように公式から勧められています。

> （DeelL訳）Shopifyのruby-lspと関連するvscode-ruby-lspは、このエクステンションに代わるものとして推奨される。TypeScriptのような他の言語に依存するよりも、Rubyそのものを使ったほうが、高品質のLSP実装を作るのはかなり簡単だ。

今回はその VS Code Extention である vscode-ruby-lsp へ切り替える方法を解説します。

ちなみにこちらの記事を見て、早くなるなら入れてみようと思ったのが最初の動機で、rebornix.Ruby が Deprecated だとは記事を書くタイミングで知りました。

https://shopify.engineering/improving-the-developer-experience-with-ruby-lsp

## 前提・期待していること

- rbenv: Ruby バージョン管理
- rubocop: Formatter & Linter

私はこれらのライブラリを主に使っています。
期待していることは、リポジトリごとに異なる Ruby バージョン対応、rubocop.yml やハイライト、rubocop のエラー表示、auto format に対応していることです。

また ruby-lsp は Ruby 2.7 以上が必要です。

> The Ruby LSP requires Ruby 2.7 or newer to run. Please upgrade your Ruby version

## 今までの構成

https://marketplace.visualstudio.com/items?itemName=rebornix.Ruby
https://marketplace.visualstudio.com/items?itemName=misogi.ruby-rubocop

### rebornix.Ruby の主な機能

- rvm、rbenv、chruby、asdf をサポートした Ruby 環境の自動検出
- RuboCop、Standard、Reek による Lint サポート
- RuboCop、Standard、Rufo、Prettier、RubyFMT によるフォーマットのサポート
- セマンティックコード折りたたみのサポート
- セマンティックハイライトのサポート
- 基本的なインテリセンスのサポート

### VS Code settings

```json
{
  "[ruby]": {
    "editor.defaultFormatter": "misogi.ruby-rubocop"
  },
  "ruby.useLanguageServer": true,
  "ruby.intellisense": "rubyLocate",
  "ruby.format": "rubocop",
  "ruby.useBundler": true,
  "ruby.rubocop.useBundler": true,
  "ruby.lint": {
    "rubocop": {
      "useBundler": true
    }
  }
}
```

## Shopify.ruby-lsp を入れる

https://marketplace.visualstudio.com/items?itemName=Shopify.ruby-lsp

こちらの Extention を入れます。rebornix.Ruby などはアンインストールして問題ありません。

https://marketplace.visualstudio.com/items?itemName=Shopify.ruby-extensions-pack

Extentions pack もあり、こちらでも問題ありません。
以下の Extention が入ります。

- Ruby LSP
- Ruby Sorbet
- VSCode rdbg Ruby Debugger

### Shopify.ruby-lsp の主な機能

- セマンティックハイライト
- シンボル検索とコードアウトライン
- RuboCop のエラーと警告（診断）
- 保存時のフォーマット（RuboCop または Syntax Tree を使用）
- タイプ時の書式
- パスの補完
- デバッグサポート
- VS Code の UI によるテストの実行とデバッグ

細かい機能郡はこちらに書いてあります。

https://shopify.github.io/ruby-lsp/RubyLsp/Requests.html

### VS Code settings

settings に設定します。細かい設定は Documents を読んでいてだければと思います。

```json
{
  "[ruby]": {
    "editor.defaultFormatter": "Shopify.ruby-lsp"
  },
  "rubyLsp.rubyVersionManager": "rbenv",
  "rubyLsp.formatter": "auto"
}
```

## まとめ

rubocop のエラー表示やフォーマット速度が段違いで早くなりました。
積極的に開発も進んでおり、今後の機能追加も期待できそうです。

https://shopify.github.io/ruby-lsp/RubyLsp/Requests/WorkspaceSymbol.html

ワークスペース内のシンボルを検索できる機能は便利そうで使ってみようと思いました。

ただ、メソッド呼び出し等まだ足りてない機能は多く、solargraph を併用するのが無難かなと思いました。Class, Module の定義ジャンプは精度が高い気はします。

https://shopify.github.io/ruby-lsp/RubyLsp/Requests/Definition.html

### 置き換えられた Extention

https://marketplace.visualstudio.com/items?itemName=rebornix.Ruby
https://marketplace.visualstudio.com/items?itemName=misogi.ruby-rubocop
https://marketplace.visualstudio.com/items?itemName=kaiwood.endwise

#### 余談

RuboCop 1.53 で rubocop にも LSP が導入され下記の記事を参考に Extention を入れてましたが、今回の Ruby LSP によって速度は改善されたので不要となっていました。

https://koic.hatenablog.com/entry/rubocop-lsp
