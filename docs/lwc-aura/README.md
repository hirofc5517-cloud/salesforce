# Lightning Web Components (LWC) / Aura 標準リファレンス

UIカスタマイズも「標準ページ・標準コンポーネントで実現できないか」を最優先で検討したうえで、
どうしても不足する場合のみコンポーネント開発（LWC）に進みます。

## 1. LWCの前に検討すべき標準UI機能

| やりたいこと | 標準機能 |
|---|---|
| 項目の条件付き表示/非表示 | Dynamic Forms |
| レコードの進捗表示 | パス（Path） |
| 主要情報の強調表示 | ハイライトパネル・コンパクトレイアウト |
| 関連情報のタブ表示 | 関連リスト・関連レコード一覧コンポーネント |
| 簡易な集計・グラフ | レポートグラフコンポーネント、ダッシュボードコンポーネント |
| クイック入力 | クイックアクション（画面フローを紐付け可能） |
| 承認操作 | 承認履歴コンポーネント・標準承認ボタン |

これらで賄えない、独自のインタラクティブUIが必要な場合のみLWCを新規開発する。

## 2. LWC vs Aura

- **新規開発はすべてLWC**を使用する。Auraコンポーネントは既存資産の保守のみとし、新規作成は禁止。
- LWCで実現できずAura固有機能が必要になるケースは基本的に存在しない
  （LWCはAuraコンテナ内に配置可能で相互運用できる）。

## 3. 標準的な構成規約

- **Base Lightning Components**（`lightning-input`, `lightning-button`, `lightning-datatable`,
  `lightning-record-edit-form`, `lightning-record-picker` 等）を最大限活用し、独自HTML/CSSでの
  再実装を避ける。
- データ取得は原則 **Wire Adapter**（`@wire(getRecord)`, `@wire(getObjectInfo)` 等の
  Lightning Data Service）を使用し、Apexコールは標準Wireで賄えない場合のみ使用する。
- Apexを呼ぶ場合は `@AuraEnabled(cacheable=true)` を可能な限り付与し、キャッシュ・パフォーマンスを
  最適化する。
- コンポーネント間通信は Pub/Sub や CustomEvent を使用し、疎結合を保つ。
- Lightning Design System（SLDS）のユーティリティクラスを使用し、独自CSSでの見た目再現は最小限にする。

## 4. セキュリティ規約

- LWCから呼ぶApexメソッドは `with sharing` を基本とし、CRUD/FLSチェックを行う
  （`Security.stripInaccessible()` 等）。
- ユーザー入力をそのままHTMLに埋め込まない（XSS対策。LWCのテンプレートエンジンは既定でエスケープされるが、
  `lwc:dom="manual"` や `innerHTML` 相当の操作は避ける）。

## 5. 実装前チェックリスト

- [ ] 標準ページ/標準コンポーネント/Dynamic Formsで実現できないか確認したか
- [ ] Base Lightning Componentsで代替できないか確認したか
- [ ] データ取得はWire Adapter（Lightning Data Service）を優先したか
- [ ] 新規にAuraコンポーネントを作成していないか（禁止）
