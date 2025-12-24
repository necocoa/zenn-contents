---
title: 'MyVision を支えるシステムと技術構成'
emoji: '🚀'
type: 'tech'
topics: ['rails', 'nextjs', 'aws', 'graphql', 'ai']
published: false
publication_name: 'my_vision'
---

## はじめに

MyVision では転職支援サービスをしており、毎日たくさんの方の転職を支援させてもらっています。

> テクノロジーと仕組みを駆使して、日本一の転職支援企業になる

https://corporate.my-vision.co.jp/

とあるように、**テクノロジーと仕組みを駆使して、支援の質を最大化すること** を目指しています。

そのために、取引企業・求人・候補者の選考状況など、転職支援に関わるすべてを管理するアプリケーション「**InVision**」を開発しています。

今回は InVision で使っているシステム構成と開発の流れを紹介します！

## インフラアーキテクチャ

まずはインフラと全体像です！

![architecture diagram](https://storage.googleapis.com/zenn-user-upload/844669e75b32-20251217.png)

ざっくりすると、このような構成になっています。

- バックエンド: Ruby on Rails
- フロントエンド: Next.js / Astro
- AI バックエンド: FastAPI / Prefect
- インフラ: AWS ECS / RDS、一部 GCP

ファーストコミットが 2023 年 3 月と新しめのアプリケーションになっています。

全体的にシンプルな構成を保ちつつ、フロンドエンドでは一部 Astro を使ったり、AI Agents のワークフロー管理で Prefect を導入したりなど、新しい技術も取り入れています。

## バックエンド

![backend](https://storage.googleapis.com/zenn-user-upload/51419290ed01-20251223.png)

Ruby on Rails を使っており、メインのアプリケーションバックエンドです。

フロントエンドとは GraphQL を使って通信しています。
一画面でさまざまなリソースの情報を表示しているため、リソースを追加・変更する際にバックエンド・フロントエンドの変更が少なく済む点を気に入っています。
また、候補者の情報や選考の進捗など、複数人が同一の情報を見ていることが多いため、GraphQL Subscription を使って更新をリアルタイムで同期するようにしています。

https://zenn.dev/my_vision/articles/758a51b6e8a2aa

https://zenn.dev/my_vision/articles/37ea494a629590

### 主な技術

- FW: Ruby on Rails
- DB: RDS / PostgreSQL
- 非同期ジョブ: Sidekiq
  - スケジュールジョブも利用
- サーバ: ECS / Fargate
- 検索: OpenSearch
- KVS: ElastiCache / Redis
  - キャッシュ・メッセージキュー・GraphQL Subscription (Action Cable) のコネクション管理
- 認証: Firebase Authentication
- Storage: Cloud Storage for Firebase
  - 認証と含めて、ストレージの操作がラクなため

### 開発環境

- 実行環境: Docker
- 言語・CLI 管理: mise
  - mise は言語や CLI のバージョン管理として便利

開発環境で気に入っているのは、マスクされた本番同等のデータがローカルで扱える点です。
データをマスクして dump し S3 にアップしており、任意のタイミングでそのデータに入れ替えることができます。

staging / local どちらも、そのデータを元に開発を進めることができるため、UX の確認やデバッグが非常にラクです。

## フロントエンド

![frontend](https://storage.googleapis.com/zenn-user-upload/58d0242c116d-20251217.png)

主に Next.js を使っています。InVision のほかに、各ブランドのメディアサイトも管理しています。

CSR とメディアの一部で ISR を使っています。
ユーザー体験・表示速度が重要な応募フォームには Astro を使い、バンドルサイズなども最適化しています。

https://zenn.dev/my_vision/articles/f3dcb3e5f04b21

### 主な技術

- FW: Next.js / Astro
- インフラ: Vercel
- GraphQL Client: Apollo / graphql-codegen
- E2E: Playwright

### 開発環境

InVision と各ブランドメディアの Next.js / Astro がモノレポ構成になっています。
モノレポ管理として Turborepo を入れています。

全フロントの立ち上げや GraphQL のスキーマ型生成の codegen 実行だったり lint などを並列かつ、キャッシュも使いながら実行できるため、高速でとても快適です。

https://turborepo.com/

また、E2E テストは LLM を活用して Playwright を書かずに要件から E2E テストを実行できる環境もできています。

https://zenn.dev/my_vision/articles/8713fc4fde965c

## AI バックエンド

![ai backend](https://storage.googleapis.com/zenn-user-upload/ed3a00942ff2-20251223.png)

書類不備のチェックや求人のマッチング精度を上げるための構造化など、さまざまな AI Agents が動いています。

API として InVision から呼び出したり、ロングタイムになりがちなエージェントワークフローや定期実行を管理するために Prefect を導入しています。
ダッシュボードからワークフローの状況を確認できたりと便利です。

### 主な技術

- FW: FastAPI
- Workflow: Prefect
- AI: Azure / OpenAI (ChatGPT)
- サーバ: ECS / Fargate
- DB: RDS / PostgreSQL
  - Prefect の状態管理

## データ分析基盤

![data platform](https://storage.googleapis.com/zenn-user-upload/08d3b903bf9d-20251216.png)

- PostgreSQL → Embulk → BigQuery (生データ)→ dbt → BigQuery (分析用)
- Fluentd (ログ) → BigQuery

のような流れで BigQuery にデータを集約しています。KPI モニタリングやマーケ分析として活用しています。

## 開発・運用環境

![service](https://storage.googleapis.com/zenn-user-upload/8f7083b9e2a6-20251217.png)

### CI/CD

- CI/CD: GitHub Actions

デプロイの流れは Git Flow を採用しており、1 日に 1 回デプロイしています。

- feature/\* → develop(staging)→ main(production)

現在、リモート環境を staging 以外に作ったり、デプロイフローを整えており、変更をすぐに反映する GitHub Flow に移行する予定です。

- feature/\* → main(production)

### 監視・運用

- Bugsnag: エラートラッキング
- CloudWatch: 監視

### 開発ツール

![dev tools](https://storage.googleapis.com/zenn-user-upload/ffbc13744f2d-20251217.png)

- デザイン: Figma
- Coding Agent: Claude Code
- ドキュメント: Notion

全エンジニアが Claude Code をゴリゴリ使って開発しています。
Figma / Notion ともに MCP を使って連携し、要件整理から実装まで一連で AI を有効活用して開発を進めています。

また、現在はバックエンド・フロントエンドともにリポジトリが別ですが、モノレポにすることで両方の変更を一括で行えるようにモノレポへ移行中です。

## まとめ

新しめなアプリケーションなのもあって、シンプルな構成を保っているのではないでしょうか？

直近では AI Agents が機能・開発ともに、実現できる環境が立ちあがった年でした。

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

これからも爆速で開発を進めるための環境を整備していけるようにしたいと思います💪

<!-- textlint-enable ja-technical-writing/ja-no-mixed-period -->

聞いてみたい内容があったり気になった方は、ぜひカジュアル面談にお越しください！

https://corporate.my-vision.co.jp/engineering-careers
