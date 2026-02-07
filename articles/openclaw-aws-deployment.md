---
title: "OpenClaw(Clawdbot AI)をAWSで動かす - CloudFormationによるサーバーレスデプロイ"
emoji: "☁️"
type: "tech"
topics: ["aws", "cloudformation", "ai", "openclaw", "infrastructure"]
published: false
---

## はじめに

AIエージェントを活用したい場合、ローカル環境での実行には課題があります。本記事では、OpenClaw（旧Clawdbot AI）をAWSにデプロイし、クラウドベースでAIエージェントを運用する方法を解説します。

CloudFormationテンプレートを使用したInfrastructure as Code（IaC）アプローチにより、再現性の高いデプロイメントを実現します。

## なぜクラウドデプロイなのか

### ローカル実行の課題

- Mac mini等の専用ハードウェアが必要
- 常時起動のための電力コストと管理負荷
- セキュリティリスク（個人マシンとの分離ができない）

### クラウドデプロイのメリット

| 観点 | メリット |
|------|---------|
| ハードウェア | 専用機不要 |
| アクセス性 | どこからでもアクセス可能 |
| 可用性 | 24/7稼働、ローカルPC不要 |
| セキュリティ | 分離された環境 |

## アーキテクチャ概要

```
[ユーザー] → [CloudFront/ALB] → [EC2インスタンス] → [LLMプロバイダー(OpenRouter等)]
```

CloudFormationで以下のリソースをプロビジョニングします：

- EC2インスタンス（t3.micro等）
- セキュリティグループ
- IAMロール
- VPC/サブネット（必要に応じて）

## デプロイ手順

### 1. CloudFormationスタックのデプロイ

AWSコンソールまたはCLIからテンプレートをデプロイします。所要時間は約5分です。

```bash
aws cloudformation create-stack \
  --stack-name openclaw-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM
```

### 2. 設定の構成

SSH接続後、以下を設定します：

- **動作モード**: ローカル
- **LLMプロバイダー**: OpenRouter（無料枠あり）
- **モデル選択**: 利用可能なモデルから選択

```bash
# 設定例
export LLM_PROVIDER=openrouter
export MODEL=gpt-oss-120b:free
```

### 3. サービス起動

設定完了後、生成されたURLにアクセスしてエージェントとの対話を開始できます。

## コスト分析

### EC2コスト

| 利用時間 | 月額コスト（概算） |
|---------|------------------|
| 8時間/日 | 約$2（約300円） |
| 24時間/日 | 約$6（約900円） |

### LLMプロバイダーコスト

OpenRouterの無料枠を使用する場合、追加コストは発生しません。有料モデルを使用する場合は、トークン使用量に応じた課金となります。

## トレードオフの考慮

### スポットインスタンスの活用

コスト削減のため、スポットインスタンスの利用を検討できます。ただし、中断リスクがあるため、ステートレスな設計が必要です。

### オートスケーリング

利用頻度に応じて、スケジュールベースのスケーリングやイベント駆動のスケーリングを実装することで、さらなるコスト最適化が可能です。

```yaml
# スケジュールベースの例
ScheduledAction:
  ScaleDown:
    Recurrence: "0 22 * * *"  # 22:00にスケールダウン
  ScaleUp:
    Recurrence: "0 8 * * *"   # 8:00にスケールアップ
```

## セキュリティ考慮事項

- セキュリティグループでインバウンドを必要最小限に制限
- IAMロールで最小権限の原則を適用
- APIキーはAWS Secrets Managerで管理
- VPC内のプライベートサブネットでの運用を推奨

## まとめ

OpenClawをAWSにデプロイすることで、専用ハードウェアなしでAIエージェントを運用できます。CloudFormationによるIaCアプローチにより、環境の再現性と管理性が向上します。

本記事で紹介したクラウドデプロイのアプローチは、OpenClawプロジェクト自体でも推奨されている手法です。AWSだけでなく、GCPやAzureへのデプロイも同様のアプローチで実現可能です。

## 参考リンク

- [OpenClaw GitHub](https://github.com/openclaw)
- [AWS CloudFormation ドキュメント](https://docs.aws.amazon.com/cloudformation/)
- [元記事（note）](https://note.com/granizm/n/n83515660ed41)
