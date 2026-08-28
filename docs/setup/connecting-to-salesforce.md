# Salesforce組織への接続方法

このリポジトリはSalesforce DX（SFDX）プロジェクトとして構成されています。
`sf`（Salesforce CLI）はこの環境に **インストール済み** です。

```bash
sf --version
# => @salesforce/cli/2.149.9 linux-x64 node-v22.22.2
```

組織への実際の認証（ログイン）には、Salesforceの実アカウント（Consumer Key/証明書、または
ユーザー名・パスワード＋ブラウザ操作）が必要なため、**このドキュメントに記載の手順をユーザー自身が
実行**してください。認証情報を安全に扱うため、認証操作をAIが代行することは想定していません。

## 方法A: Webブラウザログイン（開発者が自分のPCで作業する場合）

ローカルPC（ブラウザが使える環境）で以下を実行します。

```bash
# 本番/Developer Edition組織へ接続
sf org login web --alias myOrg --instance-url https://login.salesforce.com

# Sandboxへ接続する場合
sf org login web --alias mySandbox --instance-url https://test.salesforce.com
```

ブラウザが開き、Salesforceにログインすると認証情報がローカルにセキュアに保存されます
（`.sfdx/` `.sf/` 配下。`.gitignore` で除外済みのためコミットされません）。

接続後、デフォルト組織として設定:

```bash
sf config set target-org myOrg
```

## 方法B: JWT Bearer Flow（この環境やCI/CDなど、ブラウザを開けない場所での接続）

本リモート実行環境（コンテナ）のように対話的なブラウザ操作ができない場所から接続する場合は、
JWTベースの認証を使用します。**事前にSalesforce組織側でConnected Appの作成が必要**です。

### 事前準備（Salesforce組織側・初回のみ）

1. 自己署名証明書と秘密鍵を作成（**秘密鍵は絶対にコミットしない**）:
   ```bash
   openssl genrsa -out server.key 2048
   openssl req -new -key server.key -out server.csr
   openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
   ```
2. Salesforceの「設定 → アプリケーションマネージャ → 新規接続アプリケーション」で:
   - 「デジタル署名を使用」にチェックし、`server.crt` をアップロード
   - OAuth範囲: `api`, `refresh_token`, `offline_access` 程度を付与
   - 作成後に発行される **Consumer Key** を控える
3. 接続アプリケーションのプロファイル/権限セットで、対象の統合用ユーザーにアクセスを許可する。

### この環境での接続コマンド

秘密鍵ファイル（`server.key`）とConsumer Keyを、**環境変数または安全な場所から**渡してください
（リポジトリには絶対にコミットしないこと。`.gitignore` に `*.key` を追加済み）。

```bash
sf org login jwt \
  --username your-integration-user@example.com \
  --jwt-key-file /path/to/server.key \
  --client-id <Connected AppのConsumer Key> \
  --instance-url https://login.salesforce.com \
  --alias myOrg
```

## スクラッチ組織を使う場合（Dev Hub連携）

Dev Hub組織を有効化済みであれば、この環境からスクラッチ組織を都度作成して検証できます。

```bash
# Dev HubをWebログイン or JWTで認証後
sf org login web --alias DevHub --set-default-dev-hub
# もしくは
sf org login jwt --username devhub@example.com --jwt-key-file server.key --client-id <ConsumerKey> --set-default-dev-hub --alias DevHub

# スクラッチ組織を作成（config/project-scratch-def.json を使用）
sf org create scratch --definition-file config/project-scratch-def.json --alias scratchOrg --duration-days 7 --set-default

# ソースをプッシュ
sf project deploy start --target-org scratchOrg
```

## ソースの取得・反映コマンド（接続後）

```bash
# 組織 → ローカル（メタデータの取得）
sf project retrieve start --target-org myOrg

# ローカル → 組織（デプロイ）
sf project deploy start --target-org myOrg

# デプロイ前の検証のみ（本番はcheck-onlyを推奨）
sf project deploy start --target-org myOrg --dry-run
```

## 接続情報の確認

```bash
sf org list          # 認証済み組織の一覧
sf org display --target-org myOrg   # 接続中組織の詳細
```

## セキュリティ上の注意

- `.sfdx/`, `.sf/`, `*.key`, `.env` はコミットしないこと（`.gitignore` 済み）。
- CI/CDでJWTを使う場合は、秘密鍵をGitHub ActionsのSecretsなど安全なストア経由で渡すこと。
- 詳細な権限設計は [`docs/security-sharing/README.md`](../security-sharing/README.md)、
  外部連携の一般的な設計方針は [`docs/integration/README.md`](../integration/README.md) を参照。
