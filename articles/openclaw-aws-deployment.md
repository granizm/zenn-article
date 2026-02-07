---
title: "OpenClaw(Clawdbot AI)をAWS + Amazon Bedrockで動かす - ワンクリックデプロイの裏側"
emoji: "☁️"
type: "tech"
topics: ["aws", "bedrock", "cloudformation", "ai", "openclaw"]
published: false
---

## はじめに

AIエージェントを活用したい場合、ローカル環境での実行にはMac等の専用ハードウェアが必要です。本記事では、AWS公式サンプルとして提供されている「OpenClaw on AWS with Bedrock」を使用し、**Amazon Bedrockを活用した**クラウドベースのAIエージェント環境を構築する方法を解説します。

:::message
本記事は、AWS公式リポジトリ [aws-samples/sample-OpenClaw-on-AWS-with-Bedrock](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock) の内容に基づいています。
:::

## なぜAmazon Bedrockなのか

従来のセットアップでは、OpenRouter等の外部LLMプロバイダーのAPIキー管理が必要でした。AWS公式のBedrockソリューションでは：

- **APIキー管理が不要**: IAMロールベースの認証
- **複数モデル対応**: Nova、Claude、DeepSeek、Llamaなど8つのモデルから選択可能
- **エンタープライズグレードのセキュリティ**: VPC Endpointsによるプライベート通信

## アーキテクチャ概要

```
[ユーザー] → [WhatsApp/Telegram等] → [EC2(OpenClaw)] → [Amazon Bedrock]
                                           ↓
                                    [VPC Endpoints]
                                           ↓
                                      [CloudTrail]
```

CloudFormationで以下のリソースがプロビジョニングされます：

| サービス | 用途 |
|---------|------|
| EC2 (t4g.medium) | OpenClaw実行環境（Graviton ARM） |
| Amazon Bedrock | AIモデルAPI |
| IAM | Bedrock認証用ロール |
| VPC Endpoints | プライベートネットワーク通信 |
| CloudTrail | API呼び出し監査 |
| SSM Session Manager | セキュアアクセス |

## 前提条件

1. **Bedrockモデルの有効化**: AWSコンソールからBedrockの使用するモデルを有効化
2. **EC2キーペアの作成**: SSH接続用（SSM Session Manager使用時は不要）
3. **AWSアカウント**: 必要な権限を持つアカウント

## デプロイ手順

### 1. ワンクリックデプロイ

AWS公式リポジトリの「Launch Stack」ボタンをクリックするだけでデプロイが開始されます。

**所要時間: 約8分**

```bash
# CLIでデプロイする場合
aws cloudformation create-stack \
  --stack-name openclaw-bedrock \
  --template-url https://[S3-URL]/template.yaml \
  --capabilities CAPABILITY_IAM \
  --parameters \
    ParameterKey=KeyName,ParameterValue=your-keypair
```

### 2. 出力値の確認

CloudFormationの出力タブから以下を取得：
- **アクセスURL**
- **認証トークン**

### 3. SSM Session Managerでのアクセス

```bash
# ポートフォワーディング設定
aws ssm start-session \
  --target i-xxxxxxxxx \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["18789"],"localPortNumber":["18789"]}'
```

ブラウザで `http://localhost:18789` にアクセス。

## コスト分析

### インフラストラクチャコスト（月額）

| 項目 | 費用 |
|------|------|
| EC2 (t4g.medium) | $24.19 |
| EBS (gp3, 30GB) | $2.40 |
| VPC Endpoints | $21.60 |
| データ転送 | $5-10 |
| **小計** | **$53-58** |

### Bedrock使用料（モデル別）

| モデル | 入力 (100万トークン) | 出力 (100万トークン) |
|--------|---------------------|---------------------|
| Nova 2 Lite（推奨） | $0.30 | $2.50 |
| Nova Pro | $0.80 | $3.20 |
| Claude Sonnet | $3.00 | $15.00 |

**月額総費用**: 軽量利用で約$58-66程度

:::message alert
従来の「月額$2」という情報は誤りでした。VPC Endpointsのコストなどを含めると、**最低でも月額$53程度**が必要です。
:::

## インスタンスタイプの選択

### Linux（推奨）

| タイプ | RAM | 月額 |
|--------|-----|------|
| t4g.small | 2GB | $12 |
| t4g.medium（デフォルト） | 4GB | $24 |
| t4g.large | 8GB | $48 |
| c7g.xlarge | 8GB | $108 |

### macOS（iOS/macOS開発向け）

| タイプ | チップ | RAM | 月額 |
|--------|--------|-----|------|
| mac2.metal | M1 | 16GB | $468 |
| mac2-m2.metal | M2 | 24GB | $632 |

## 対応プラットフォーム

OpenClawは以下のメッセージングプラットフォームに対応：

- **WhatsApp**（推奨）
- **Telegram**
- **Discord**
- **Slack**
- **Microsoft Teams**

## セキュリティ設計

このソリューションは以下のセキュリティ対策を実装：

1. **VPC Endpoints**: Bedrockへのプライベートアクセス
2. **IAMロール**: 最小権限の原則
3. **CloudTrail**: 全API呼び出しの監査ログ
4. **SSM Session Manager**: SSH不要のセキュアアクセス

## トレードオフの考慮

### スポットインスタンスの制限

Graviton (ARM) インスタンスはスポット価格が安定していますが、OpenClawのような常時稼働サービスでは、オンデマンドまたはReserved Instancesが推奨されます。

### コスト最適化のオプション

- 使用しない時間帯のインスタンス停止
- 小さいインスタンスタイプ（t4g.small: $12/月）への変更
- Nova 2 Lite等の低コストモデルの選択

## まとめ

OpenClawをAWS + Amazon Bedrockで運用することで、APIキー管理の煩雑さを解消し、エンタープライズグレードのセキュリティを実現できます。ただし、VPC Endpointsなどのコストを含めると月額$53程度は必要である点に注意が必要です。

なお、本記事で紹介したクラウドデプロイのアプローチは、OpenClawプロジェクトでもAWSネイティブな運用方法として公式にサポートされています。オープンソースプロジェクト「OpenClaw」への貢献を通じて、クラウドインフラやAI統合のスキルを実践的に学ぶことができます。

## 参考リンク

- [OpenClaw on AWS with Bedrock (AWS公式サンプル)](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock)
- [Amazon Bedrock ドキュメント](https://docs.aws.amazon.com/bedrock/)
- [AWS CloudFormation ドキュメント](https://docs.aws.amazon.com/cloudformation/)
- [元記事（note）](https://note.com/granizm/n/n83515660ed41)
