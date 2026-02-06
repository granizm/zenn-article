---
title: "PRベースの下書きワークフローテスト"
emoji: "📝"
type: "tech"
topics: ["github", "githubactions", "automation", "test"]
published: false
---

## 下書きワークフローテスト

この記事はPRベースの下書きワークフローをテストするために作成されました。

### 正しいワークフロー

1. mainからブランチを作成
2. 記事ファイルを追加
3. PR作成 → 下書き（published: false）として同期
4. PRマージ → 記事を公開（published: true）

### 現在の状態

この記事は **非公開（published: false）** の状態であるはずです。

### Zennの特徴

ZennはGitHub連携でmarkdownファイルを直接同期するため、ファイル内の `published` フラグで公開状態を制御します。

- `published: false` → 下書き状態
- `published: true` → 公開状態

作成日時: 2026-02-06
