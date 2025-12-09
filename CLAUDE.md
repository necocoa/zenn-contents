Zenn の技術ブログ記事を管理しています。

## ディレクトリ構造

```
articles/           # 技術記事（Markdown）
books/              # 本（複数章で構成される長編コンテンツ）
```

## 開発コマンド

```bash
# プレビューサーバー起動（http://localhost:8000）
pnpm dev

# 新規記事作成
pnpm new

# Lint
pnpm lint:check    # チェックのみ
pnpm lint:fix      # 自動修正（単一ファイル）
pnpm lint:all      # 全ファイル自動修正

# フォーマット
pnpm format:check  # チェックのみ
pnpm format:fix    # 自動修正
```

## 変更

- commit前に `format:check` と `lint:check` を実行

## ブログレビュー

ブログ記事のレビューは、`.claude/commands/blog-review.md` のドキュメントを参照してください
