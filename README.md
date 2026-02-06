# zenn-article

Zenn記事公開用リポジトリ

## 概要

このリポジトリはZennへの記事公開を管理します。
- **PR作成**: 記事を下書き状態に設定
- **PRマージ**: 記事を公開状態に設定

Zennとの連携はGitHubリポジトリ連携で行うため、APIトークンは不要です。

## ディレクトリ構成

```
zenn-article/
├── articles/           # 記事
├── books/              # 本
├── images/             # 画像
├── .github/workflows/  # GitHub Actions
└── package.json
```

## ワークフロー

| イベント | アクション | Zenn状態 |
|----------|-----------|----------|
| PR作成・更新 | published: false に設定 | 下書き |
| PRマージ | published: true に設定 | 公開 |

## 記事の作成

```bash
npm install
npm run new:article
```

## Frontmatter

```yaml
---
title: "記事タイトル"
emoji: "📝"
type: "tech"  # tech or idea
topics: ["topic1", "topic2"]
published: false  # PRマージ時にtrueになる
---
```

## 必要な設定

### Zenn連携
1. [Zenn](https://zenn.dev) にGitHubアカウントでログイン
2. ダッシュボード → GitHubからのデプロイ
3. このリポジトリを連携

### GitHub Secrets（オプション）
- `DISCORD_WEBHOOK`: Discord通知用Webhook URL

## 関連リンク

- [granizm-blog](https://github.com/granizm/granizm-blog) - アイデア・下書き管理
- [Zenn CLI](https://zenn.dev/zenn/articles/zenn-cli-guide)
