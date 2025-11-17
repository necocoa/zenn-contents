---
title: 'GitHub Actions の新ランナー ubuntu-slim でコスト削減する'
emoji: '🐳'
type: 'tech'
topics: ['githubactions', 'github']
published: true
publication_name: 'my_vision'
---

## はじめに

<!-- textlint-disable prh -->

2025 年 10 月 28 日に GitHub Actions から新ランナー「ubuntu-slim」が発表されました。

https://github.blog/changelog/2025-10-28-1-vcpu-linux-runner-now-available-in-github-actions-in-public-preview/

軽量なオペレーションを想定とした低コストなランナーとして提供されています。
指定しているランナーを書き換えるだけなのでサクッと導入してみました！

## コスパ比較

|       Runner        |  vCPU  | コスト(分) |  RAM  | 実行時間制限 | 　実行環境 |
| :-----------------: | :----: | :--------: | :---: | ------------ | ---------- |
|     ubuntu-slim     | 1 vCPU |   $0.002   | 5 GB  | 　15分       | 　コンテナ |
|    ubuntu-latest    | 2 vCPU |   $0.008   | 7 GB  | 6時間        | 　VM       |
| ubuntu-latest-4core | 4 vCPU |   $0.016   | 16 GB | 6時間        | 　VM       |

※ 表はプライベートリポジトリの価格です。

通常の `ubuntu-latest` に比べて 1/4 のコストになっています。

Actions の課金体系は分単位での課金となります。実行時間が 1 秒でも切り上げて 1 分の課金がされます。
数秒で終わるが実行回数が多いアクションは、実行時間は対して変わらない割にはコストは 1/4 となるためチリツモになります。

## 切り替え方法

actions ファイルの `runs-on` を `ubuntu-slim` に変更します。

```diff yaml:.github/workflows/sample.yml
# 変更後
jobs:
  lint:
-   runs-on: ubuntu-latest
+   runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v5
      - run: npm lint
```

claude code などに「軽い Actions だけランナーを `ubuntu-slim` に切り替えて」と頼みましょう！

## 注意事項

### プリインストールツール

`ubuntu-latest` にはプリインストールされているが `ubuntu-slim` には含まれていないツールがあります。

例:

- 各種言語のバージョン管理ツール（nvm, rbenv 等）
- ビルドツール（make 等）
- データベースクライアント

現時点での `ubuntu-latest` に入っているツールはこちらです。
https://github.com/actions/runner-images/blob/releases/ubuntu24/20251102/images/ubuntu/ubuntu2404-Readme.md

`ubuntu-slim` に入っているツールは探してみたのですが見つかりませんでした。

不足しているパッケージは別途インストールで対応できますが、実行時間も余計にかかります。`ubuntu-latest` がタイパ・コスパどちらもよい場合は大いにありそうです。

### 環境変数

アクション上で走らせた Ruby ERB が `US-ASCII` として解釈され、日本語を読み込んだ際にエラーが発生しました。
環境変数として `LANG` `LC_ALL` が設定されていないため、日本語を扱うアクションには明示的に指定しました。

```diff yaml
env:
+  LANG: C.UTF-8
+  LC_ALL: C.UTF-8
```

### 実行環境

通常の `ubuntu-latest` などは専用の VM インスタンスが立ち上がりますが `ubuntu-slim` ではコンテナ内で実行されます。

### 実行時間

実行時間が 15 分で終了するため、長時間かかる想定のアクションは切り替えない方がよさそうです。

## どのようなケースに向いているか

GitHub からは以下のようなケースに推奨されています。実行時間が短く、CPU/RAM 等のリソース依存度が低いものを置き換えると良さそうです。

- 自動ラベルやアサイン付け
- シンプルなビルドやコンパイル
- Lint / Format

## 移行した結果

`ubuntu-latest` を使っているアクションのうち、半分ほどを `ubuntu-slim` に切り替えることができました。

残りは実行時間が長かったりコアを増やしているアクションになるのでそれらを含めて、概算ですが CI コストを 10%~15% ほど削減できたのではないでしょうか！

## まとめ

タイパがよいコスト削減になるのでぜひやってみましょう！（CI 時間が伸びすぎないように前後で比較してね）

### リファレンス

https://github.blog/changelog/2025-10-28-1-vcpu-linux-runner-now-available-in-github-actions-in-public-preview/

https://docs.github.com/ja/billing/reference/actions-runner-pricing

https://dev.classmethod.jp/articles/github-actions-ubuntu-slim-runner-public-preview/
