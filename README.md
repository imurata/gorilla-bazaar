# gorilla-bazaar

Kong Konnect に接続するシンプルな APIOps サンプルリポジトリです。
OpenAPI Specification (OAS) を起点に、decK と kongctl を使って Kong Gateway の設定と API カタログへのデプロイを自動化します。

## リポジトリ構成

```
gorilla-bazaar/
├── .github/
│   └── workflows/
│       └── apiops.yml        # GitHub Actions ワークフロー
├── apis/
│   ├── apis.yaml             # kongctl 用 API カタログ定義
│   └── specs/
│       └── openapi.yaml      # OpenAPI Specification
└── deck/
    ├── consumers.yaml        # Consumer 定義
    └── plugins.yaml          # Plugin 定義 (deck file add-plugins 形式)
```

## ワークフロー概要

`main` ブランチへの push または PR をトリガーに以下が実行されます。

### validate ジョブ（push / PR 共通）

| ステップ | 内容 |
|---|---|
| 1. openapi2kong | OAS を Kong 宣言的設定に変換 |
| 2. merge | Consumer 定義をマージ |
| 3. add-plugins | Key Auth プラグインを追加 |
| 4. add-tags | 全リソースに `gorilla-bazaar` タグを一括付与 |
| 5. validate | 最終設定ファイルを検証 |

### deploy ジョブ（`main` push 時のみ）

| ステップ | 内容 |
|---|---|
| 6. deck gateway sync | Kong Gateway へ設定を同期 |
| 7. kongctl apply | Konnect API カタログへ OAS をデプロイ |

## 事前準備

### 1. Konnect の Personal Access Token を取得

[Kong Konnect](https://cloud.konghq.com/) にログインし、
**Personal Access Token (PAT)** を発行してください。

### 2. GitHub リポジトリへの Secret / Variable 登録

`gh` コマンドを使って以下を登録します。

```bash
# Secret: Konnect PAT
gh secret set KONNECT_PAT --body "<your-konnect-pat>"

# Variable: 対象の Control Plane 名
gh variable set KONNECT_CONTROL_PLANE_NAME --body "<your-control-plane-name>"
```

登録内容の確認：

```bash
gh secret list
gh variable list
```

### 3. Consumer の API Key を変更する

[deck/consumers.yaml](deck/consumers.yaml) 内のプレースホルダーを実際のキーに書き換えてください。

```yaml
keyauth_credentials:
  - key: demo-api-key-change-me  # ← 変更してください
```

> **注意:** API Key は Secret Manager などで管理し、リポジトリに平文でコミットしないことを推奨します。

## 使用ツール

| ツール | 用途 |
|---|---|
| [decK](https://github.com/Kong/deck) | OAS 変換・設定マージ・Gateway 同期 |
| [kongctl](https://github.com/Kong/kongctl) | Konnect API カタログへの OAS デプロイ |
