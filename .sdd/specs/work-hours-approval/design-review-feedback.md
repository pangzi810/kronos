承認機能（Work Hours Approval）設計指針

  データ型の一貫性

  - ID型の統一: すべてのエンティティIDはString型を使用すること
    - WorkRecord, User, Project等の既存エンティティと一貫性を保つ
    - 新規エンティティ（SupervisorRelationship,
  ApprovalHistory等）も同様にString型を使用
    - ドメインイベント（WorkRecordChangedEvent）のID
  フィールドもString型で統一

  ドメインイベント設計

  - イベント統合: 作成・更新・承認・却下を単一のWorkRecordChangedEventに統合
    - actionフィールド（CREATE/UPDATE/APPROVE/REJECT）で操作を識別
    - Kafkaトピックはwork-record-changedに統一
  - イベント生成場所: Application
  Service層でイベントを生成（エンティティ内では生成しない）
  - ステータス取得:
  currentStatusは常にworkRecordの実際の値から取得（ハードコード禁止）

  API設計原則

  - 既存API活用: 新規エンドポイント作成より既存APIの拡張を優先
    - saveWorkRecords()メソッドを承認ステータス対応に改修
    - PUT /api/work-recordsエンドポイントを継続使用

  DDD実装パターン

  - レイヤー配置:
    - WorkRecord等のエンティティはDomain Layer
    - Repository実装はInfrastructure Layer
    - Application ServiceはApplication Layer
  - サービス間連携:
    -
  ApprovalAuthorityService（権限検証）はApprovalWorkflowServiceから呼び出し
    - 承認権限検証と承認処理を適切に分離

  システムアーキテクチャ図の記載方法

  - サービス責務: 各サービスボックスに責務を箇条書きで明記
  - UI表記: 「作業ログ承認画面」等、具体的な画面名を使用
  - 簡潔性: Entity/Value Objectの詳細は不要（ドメインモデル図と分離）
  - 関係性明示: Service間、Service-Repository間の依存関係を明確に表示

  実装時の注意事項

  - 型変換の排除: Long.parseLong()等の不要な型変換を避ける
  - null安全性: Optional型の適切な使用
  - トランザクション境界: Application Service層で@Transactionalを定義
  - 例外処理: ドメイン例外（SelfApprovalException等）を適切に定義・使用

  テスト実装

  - ID型の一貫性: テストコードでもString型IDを使用
  - カバレッジ: 80%以上のテストカバレッジを維持
  - 承認フローテスト: 申請→承認/却下→履歴確認の一連のフローをE2Eテストで検証

  データベース設計

  - 既存スキーマとの整合性:
  既存テーブルのID型（VARCHAR）と新規テーブルの整合性を保つ
  - Migration戦略: Flyway V7以降で承認機能関連のテーブルを追加
  - インデックス最適化: approval_status,
  supervisor_id等に適切なインデックスを設定

  ---
  このプロンプトをCLAUDE.mdの「## Important
  Constraints」または新規セクション「## Approval Feature Design
  Guidelines」として追加することで、今後の承認機能実装時に一貫性のある設計を
  維持できます。