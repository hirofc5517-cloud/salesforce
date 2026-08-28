# リポジトリガイド（Claude Code向け）

このリポジトリはSalesforce DX（SFDX）プロジェクトです。

- `force-app/` … Salesforceソースコード本体（Apex, Flow, LWC, オブジェクト定義等）
- `docs/` … Salesforce標準機能のリファレンス・実装方針ガイド
- `config/` … スクラッチ組織定義
- `sfdx-project.json` … SFDXプロジェクト定義

## 実装・回答の必須ルール

Salesforceのカスタマイズ（Apex、Flow、LWC、権限、連携など）を提案・実装する際は、
**必ず着手前に `docs/README.md` とその配下の該当ドキュメントを読み、その方針に従うこと。**

特に以下を厳守する:

1. 実装手段の選定は [`docs/00-automation-decision-guide.md`](./docs/00-automation-decision-guide.md)
   の優先順位（標準機能 → 入力規則/数式 → Flow → Apex → LWC）に従う。
2. Apexを書く場合は [`docs/apex/README.md`](./docs/apex/README.md) のガバナ制限・バルク化・
   トリガー設計・テスト規約（カバレッジ75%以上）に従う。
3. Flowを書く場合は [`docs/flow/README.md`](./docs/flow/README.md) の種類選定・before/after-save
   の使い分け・フォルトパス必須のルールに従う。
4. LWC/Auraは [`docs/lwc-aura/README.md`](./docs/lwc-aura/README.md) に従い、Auraの新規作成は禁止。
5. 権限設計は [`docs/security-sharing/README.md`](./docs/security-sharing/README.md) の
   権限セット中心・最小権限の原則に従う。
6. 外部連携は [`docs/integration/README.md`](./docs/integration/README.md) に従い、
   認証情報はNamed Credentialで管理する（ハードコード禁止）。
7. **Process Builder / Workflow Ruleの新規作成は禁止**（Flowへ統合済みのため）。

組織への接続方法は [`docs/setup/connecting-to-salesforce.md`](./docs/setup/connecting-to-salesforce.md)
を参照。`sf`（Salesforce CLI）はこの環境にインストール済み（`sf --version` で確認可能）。

## よく使うコマンド

```bash
sf org list                                            # 認証済み組織一覧
sf project deploy start --target-org <alias>           # デプロイ
sf project retrieve start --target-org <alias>         # メタデータ取得
sf apex run test --target-org <alias> --code-coverage  # Apexテスト実行
npm run lint                                           # LWC/Aura Lint
npm run test:unit                                      # LWC Jestテスト
```
