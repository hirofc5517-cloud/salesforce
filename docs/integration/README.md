# 外部連携 標準リファレンス

外部システムとの連携も、標準機能（設定ベース）を最優先し、認証情報のハードコードを避けます。

## 1. 標準の連携手段

| 手段 | 用途 |
|---|---|
| **Named Credential** | 外部エンドポイント・認証情報を組織設定として一元管理（コード/Flowにトークンを書かない） |
| **External Services** | 外部のOpenAPI(Swagger)定義を取り込み、Flow/Apexから型安全に呼び出せるアクションとして自動生成 |
| **Connected App** | Salesforceを外部からOAuth連携させる（外部アプリがSalesforceに接続する場合） |
| **Platform Events** | 疎結合な非同期イベント連携（Pub/Sub）。Flow/Apexの双方から発行・購読可能 |
| **Change Data Capture (CDC)** | レコード変更を外部システムへリアルタイム配信 |
| **REST API / Bulk API** | 標準API。大量データはBulk API、リアルタイム単発処理はREST APIを使う |

## 2. 認証フロー（Connected App / OAuth）の使い分け

| フロー | 用途 |
|---|---|
| **Web Server Flow (Authorization Code)** | ユーザーが介在するWebアプリ連携（ブラウザでログイン） |
| **JWT Bearer Flow** | サーバー間連携、CI/CDでの自動デプロイ・自動ログイン（ユーザー操作不要） |
| **Device Flow** | ブラウザを開けないデバイスからの連携 |
| **Client Credentials Flow** | ユーザーコンテキスト不要のシステム間連携 |

本リポジトリのCI/CD・自動デプロイ用途では、原則 **JWT Bearer Flow** を使用する
（[`docs/setup/connecting-to-salesforce.md`](../setup/connecting-to-salesforce.md) 参照）。

## 3. 実装規約

- コールアウトのエンドポイントURL・APIキー・トークンは**Named CredentialまたはCustom Metadata Type**で
  管理し、Apex/Flowのソースコードに直書きしない。
- 外部コールアウトはApexの場合 `HttpRequest`/`HttpResponse` を使用し、タイムアウト・エラーハンドリングを
  必ず実装する（[`docs/apex/README.md`](../apex/README.md) の非同期Apex参照）。
- 同期コールアウト後にDMLを行う場合の制限（コールアウト後のDMLは同一トランザクションで制約あり）に注意し、
  必要に応じて `@future(callout=true)` やQueueable Apexで非同期化する。
- Platform Eventの発行は**バルク（複数件をまとめて）**で行い、1件ずつ発行するループを避ける。

## 4. 実装前チェックリスト

- [ ] 認証情報はNamed Credentialで管理し、コード上にハードコードしていないか
- [ ] サーバー間の自動連携にはJWT Bearer Flowを使用しているか
- [ ] 大量データの連携はBulk API/Batch Apexを使用しているか
- [ ] コールアウトにタイムアウト・エラーハンドリングを実装しているか
