# Salesforce 標準機能 リファレンス（AI開発ガイドライン）

このディレクトリは、本リポジトリで Salesforce のカスタマイズ（Apex / Flow / LWC など）を
AI（Claude等）が提案・実装する際に、**Salesforce標準機能の範囲・限界・ベストプラクティス**を
正しく踏まえた回答をするための「拠り所」となるドキュメント集です。

`force-app/` と同じ階層に `docs/` を置き、機能領域ごとにフォルダを分けています。

```
salesforce/
├── force-app/                # SFDXソースコード（本体）
├── config/                   # スクラッチ組織定義など
├── docs/                     # ← 本ディレクトリ（標準機能リファレンス）
│   ├── 00-automation-decision-guide.md   # 「何を使うべきか」の判断基準
│   ├── apex/                 # Apex（コード）
│   ├── flow/                 # Flow（画面/自動化フロー）
│   ├── lwc-aura/             # Lightning Web Components / Aura
│   ├── validation-formula/   # 入力規則・数式・ロールアップ集計
│   ├── security-sharing/     # プロファイル・権限セット・共有ルール
│   ├── integration/          # 外部連携（API・Connected App・Platform Events）
│   ├── reports-dashboards/   # レポート・ダッシュボード
│   └── setup/                # Salesforce組織への接続方法
└── sfdx-project.json
```

## AIへの基本方針（回答・実装の優先順位）

Salesforce実装を提案・生成する際は、**必ず以下の優先順位**で「最も宣言的（ノーコード/ローコード）な
手段」から検討し、コードが本当に必要な場合のみ Apex へ進むこと。上位の手段で要件を満たせるなら、
下位の手段（Apex等）を提案しない。

1. **標準オブジェクト／標準機能の設定**（項目・レイアウト・パス・重複ルールなど）
2. **入力規則（Validation Rule）／数式項目／ロールアップ集計項目** → [`validation-formula/`](./validation-formula/README.md)
3. **Flow（画面フロー／レコードトリガーフロー／スケジュールフロー）** → [`flow/`](./flow/README.md)
4. **Apex（トリガー／クラス／バッチ／非同期処理）** → [`apex/`](./apex/README.md)
   - Flowで実現不可能（複雑なアルゴリズム、外部コールアウトの複雑な制御、大量データの高度な最適化、
     再利用性の高いドメインロジック等）な場合のみ選択する。
5. **LWC／Aura（UI拡張）** → [`lwc-aura/`](./lwc-aura/README.md)
   - 標準ページ・標準コンポーネント（Dynamic Forms, Path, Highlights Panel等）で不足する場合のみ。

判断に迷う場合は [`00-automation-decision-guide.md`](./00-automation-decision-guide.md) の判断基準表を参照すること。

## 回答時の遵守事項

- **Process Builder / Workflow Rule は新規作成禁止**（Salesforceにより非推奨・廃止方針、Flowへ統合済み）。
  既存資産の話が出た場合は Flow への移行を提案する。
- Apex を書く場合は必ず `docs/apex/README.md` のガバナ制限・バルク化・テスト網羅率(75%以上)・
  トリガーフレームワークの規約に従う。
- Flow を書く場合は必ず `docs/flow/README.md` の種類選定・エラーハンドリング（フォルトパス）・
  再帰制御の規約に従う。
- 権限・共有の話は `docs/security-sharing/README.md` の最小権限の原則（Profile最小化 + Permission Set加算方式）
  に従う。
- 外部連携は `docs/integration/README.md` に従い、認証情報はNamed Credential／Connected Appで管理し、
  エンドポイントやトークンをコード・Flowにハードコードしない。
- 組織への接続方法は `docs/setup/connecting-to-salesforce.md` を参照。

## 各ドキュメントの位置づけ

| フォルダ | 内容 |
|---|---|
| `00-automation-decision-guide.md` | 自動化手段（宣言的機能 vs Apex）の選定フローチャート |
| `apex/` | Apexの用途・ガバナ制限・トリガー設計・非同期処理・テスト規約 |
| `flow/` | Flowの種類・トリガー条件・ベストプラクティス・制限値 |
| `lwc-aura/` | LWC/Auraの使い分け、標準コンポーネント、Wireアダプタ |
| `validation-formula/` | 入力規則・数式項目・ロールアップ集計の標準機能 |
| `security-sharing/` | プロファイル・権限セット・OWD・共有ルール・MFA |
| `integration/` | REST/Bulk API・Connected App・Platform Events・Named Credential |
| `reports-dashboards/` | 標準レポートタイプ・ダッシュボードの制限と活用 |
| `setup/` | `sf` CLIによる組織接続・認証・デプロイ手順 |
