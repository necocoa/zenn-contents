---
title: 'MyVision を支えるインフラ・アプリケーション構成'
emoji: '😎'
type: 'tech'
topics: []
published: false
publication_name: 'my_vision'
---

---

## はじめに

MyVision では転職支援サービスをしており、毎日たくさんの方の転職を支援させてもらっています。

> テクノロジーと仕組みを駆使して、日本一の転職支援企業になる

https://corporate.my-vision.co.jp/

とあるように、テクノロジーと仕組みを駆使して、支援の質を最大化することを目指しています。

そのために、取引企業・求人・候補者の選考状況などを管理する SFA/CRM アプリケーション「**InVision**」を開発しています。

今回は InVision で使っているシステムの全体像を紹介します！

## インフラアーキテクチャ全体像

まずは全体像です。

![architecture diagram](https://storage.googleapis.com/zenn-user-upload/f07355ece035-20251216.png)

ざっくりするとこのような構成になっています。

- バックエンド: Ruby on Rails
- フロントエンド: Next.js / astro
- AI バックエンド: FastAPI / Prefect
- インフラ: AWS ECS / RDS

全体的にシンプルな構成を保ちつつ、フロンドエンドでは一部 astro を使ったり、AI Agents のワークフロー管理で Prefect を導入したりなど、新しい技術も取り入れています。

## バックエンド

![backend](https://storage.googleapis.com/zenn-user-upload/46d950badfcc-20251216.png)

Ruby on Rails の　API　モードを使っています。
フロントエンドとは GraphQL、AI バックエンドとは REST でやりとりをしています。

非同期・スケジュールジョブは sidekiq です。
求人検索などに OpenSearch を使っています。

## フロントエンド

![frontend](https://storage.googleapis.com/zenn-user-upload/f9220ac44650-20251216.png)

主に Next.js を使っています。管理アプリケーション InVision のほかに、各ブランドのメディアサイトも運用しています。

ユーザー体験・表示速度が重要な応募フォームには astro を使い、バンドルサイズなどを最適化しています。

インフラはどちらも Vercel を使っています。

## AI バックエンド

![ai backend](https://storage.googleapis.com/zenn-user-upload/28b9cb4c40b6-20251216.png)

書類不備のチェックや求人のマッチング精度を上げるための構造化など、さまざまな AI Agents が動いています。

API として FastAPI、ロングタイムなワークフローを管理するために Prefect を導入しています。

Prefect の状態管理として、RDS が立っています。

## データ分析基盤

![data platform](https://storage.googleapis.com/zenn-user-upload/08d3b903bf9d-20251216.png)

## その他サービス

![service](https://storage.googleapis.com/zenn-user-upload/227d52815839-20251216.png)

### CI/CD

GitHub Actions を使って CI/CD を構築しています。

### 監視・運用

- Bugsnag: エラートラッキング
- CloudWatch: 監視
