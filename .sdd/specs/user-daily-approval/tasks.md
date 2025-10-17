# Implementation Plan

## 承認単位変更（申請者/日付単位）実装タスク

- [ ] 1. データベーススキーマとドメインモデルの作成
- [ ] 1.1 WorkRecordApprovalテーブルのマイグレーション作成
  - Flyway マイグレーションスクリプト V17__create_work_record_approval_table.sql を作成
  - user_id, date, approval_status, approver_id, approved_at, rejection_reason カラムを定義
  - プライマリキー（user_id, date）と必要なインデックスを作成
  - 既存 work_records テーブルとの外部キー制約を設定
  - _Requirements: 3.1, 3.2_

- [ ] 1.2 WorkRecordApprovalエンティティの実装とテスト
  - WorkRecordApproval エンティティクラスのテストを作成（承認、差し戻し、状態遷移）
  - domain/model/entity/WorkRecordApproval.java を実装
  - approve(), reject(), isEditable() などのビジネスメソッドを実装
  - ApprovalStatus 値オブジェクトを活用した状態管理
  - _Requirements: 3.3, 3.4, 3.5_

- [ ] 1.3 既存データ移行スクリプトの作成
  - V18__migrate_approval_data_to_work_record_approval.sql を作成
  - work_records の承認情報を WorkRecordApproval テーブルに移行
  - 申請者/日付でグループ化し、最も制限的なステータスを適用
  - ロールバック可能な移行処理を実装
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_

- [ ] 2. リポジトリ層の実装
- [ ] 2.1 WorkRecordApprovalRepository インターフェースとテスト
  - リポジトリインターフェースのテストを作成（CRUD操作、検索条件）
  - domain/repository/WorkRecordApprovalRepository.java を作成
  - findByUserIdAndDate, findByUsersAndStatuses などのメソッドを定義
  - save メソッドで create/update を統一管理
  - _Requirements: 3.1, 3.6, 3.7_

- [ ] 2.2 MyBatis マッパーとリポジトリ実装
  - WorkRecordApprovalMapper インターフェースのテストを作成
  - infrastructure/mapper/WorkRecordApprovalMapper.java を実装
  - アノテーションベースの SQL マッピングを定義
  - infrastructure/repository/WorkRecordApprovalRepositoryImpl.java を実装
  - _Requirements: 3.8, 7.4_

- [ ] 3. ドメインサービスの実装
- [ ] 3.1 ApprovalAuthorityService の実装とテスト
  - 承認権限検証のテストケースを作成（権限あり/なし、不正なリクエスト）
  - domain/service/ApprovalAuthorityService.java を更新
  - validateAuthority メソッドで申請者/日付単位の権限検証を実装
  - SupervisorRelationshipRepository を活用した上下関係の検証
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

- [ ] 3.2 ApprovalWorkflowService の実装とテスト
  - ワークフロー処理のテストを作成（状態遷移、履歴記録）
  - domain/service/ApprovalWorkflowService.java を更新
  - processApproval メソッドで承認フローを処理
  - recordApprovalHistory で承認履歴を記録
  - _Requirements: 1.3, 1.5, 3.4, 3.5_

- [ ] 4. アプリケーションサービスの実装
- [ ] 4.1 DailyApprovalApplicationService の作成とテスト
  - 承認・差し戻し処理のテストを作成（正常系、異常系、権限チェック）
  - application/service/DailyApprovalApplicationService.java を実装
  - approveDaily, rejectDaily メソッドで申請者/日付単位の承認を実装
  - トランザクション管理と例外処理を実装
  - _Requirements: 1.3, 1.5, 7.1_

- [ ] 4.2 ApprovalAggregationApplicationService の作成とテスト
  - 工数集計のテストを作成（カテゴリ別集計、案件内訳）
  - application/service/ApprovalAggregationApplicationService.java を実装
  - getPendingApprovals で承認待ち一覧を集計
  - aggregateByCategory でカテゴリ別工数を算出
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [ ] 4.3 ドメインイベント発行の統合
  - WorkRecordApprovedEvent, WorkRecordRejectedEvent のテストを作成
  - DailyApprovalApplicationService にイベント発行を統合
  - DomainEventPublisher を使用して Kafka にイベントを送信
  - イベント発行のエラーハンドリングを実装
  - _Requirements: 1.3, 1.5_

- [ ] 5. REST API の実装
- [ ] 5.1 ApprovalController の改修とテスト
  - API エンドポイントのテストを作成（承認、差し戻し、一覧取得）
  - presentation/controller/ApprovalController.java を更新
  - POST /api/approvals/daily エンドポイントを実装
  - 申請者/日付単位での承認・差し戻し API を追加
  - _Requirements: 7.1, 7.2, 7.3_

- [ ] 5.2 承認待ち一覧 API の改修
  - GET /api/approvals/pending の統合テストを作成
  - 既存エンドポイントを改修して申請者/日付でグループ化
  - カテゴリ別集計と案件内訳を含むレスポンスを返却
  - ページネーションとソート機能を実装
  - _Requirements: 7.4, 7.5_

- [ ] 5.3 一括承認 API の実装
  - POST /api/approvals/batch のテストを作成
  - 複数の申請者/日付を一括承認する処理を実装
  - トランザクション管理で整合性を保証
  - 部分的な失敗時のエラーハンドリング
  - _Requirements: 1.4, 7.6_

- [ ] 6. フロントエンドコンポーネントの実装
- [ ] 6.1 承認 API クライアントの実装とテスト
  - approvalApi.ts のテストを作成（API 呼び出し、エラーハンドリング）
  - frontend/src/services/approvalApi.ts を作成
  - approveDaily, rejectDaily, getPendingApprovals メソッドを実装
  - エラーハンドリングと型定義を整備
  - _Requirements: 7.1, 7.4_

- [ ] 6.2 承認待ち一覧コンポーネントの作成
  - ApprovalList.vue のコンポーネントテストを作成
  - frontend/src/components/approvals/ApprovalList.vue を実装
  - 申請者/日付でグループ化された一覧表示
  - インライン承認・差し戻しボタンの実装
  - _Requirements: 5.1, 5.2, 5.4_

- [ ] 6.3 カテゴリ別集計表示の実装
  - CategorySummary.vue のテストを作成
  - カテゴリ別工数の集計表示コンポーネントを実装
  - 展開可能な案件内訳表示を追加
  - リアルタイム更新機能の実装
  - _Requirements: 2.2, 2.3, 2.4_

- [ ] 6.4 一括承認機能の実装
  - BatchApproval.vue のテストを作成
  - チェックボックスによる複数選択機能
  - 一括承認ボタンとモーダル確認
  - 処理中の進捗表示とエラーハンドリング
  - _Requirements: 5.3_

- [ ] 7. 申請者向け機能の実装
- [ ] 7.1 カレンダービューの承認状況表示
  - CalendarView.vue の更新テストを作成
  - WorkRecordApproval から承認状態を取得して表示
  - 承認済み（緑）、承認待ち（黄）、差し戻し（赤）で色分け
  - 差し戻し理由のツールチップ表示
  - _Requirements: 6.1, 6.2, 6.4, 6.5_

- [ ] 7.2 工数入力制御の実装
  - WorkRecordForm.vue の更新テストを作成
  - 承認済み日付の編集無効化処理を実装
  - 差し戻し時の再入力フォーム表示
  - API エラー（409）のハンドリング
  - _Requirements: 6.3, 7.2, 7.3_

- [ ] 8. レポート機能の実装
- [ ] 8.1 ApprovalReportApplicationService の実装
  - レポート生成のテストを作成
  - application/service/ApprovalReportApplicationService.java を実装
  - 承認状況サマリーの集計ロジック
  - リアルタイム更新用の差分データ取得
  - _Requirements: 9.1, 9.2, 9.3_

- [ ] 8.2 レポート画面コンポーネントの作成
  - ApprovalReport.vue のテストを作成
  - frontend/src/components/reports/ApprovalReport.vue を実装
  - 承認状況のダッシュボード表示
  - 30秒ごとの自動更新機能
  - _Requirements: 9.4_

- [ ] 9. 統合とE2Eテスト
- [ ] 9.1 システム統合テストの実装
  - 全コンポーネントの統合テストを作成
  - 承認フロー全体の動作確認（権限検証 → 承認 → イベント発行）
  - データ移行後の動作確認
  - 既存機能との互換性テスト
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

- [ ] 9.2 E2E テストシナリオの実装
  - Playwright を使用した E2E テストを作成
  - 承認者フロー：ログイン → 一覧表示 → 承認/差し戻し → 確認
  - 申請者フロー：ログイン → カレンダー確認 → 工数入力 → 承認状態確認
  - マネージャーフロー：ログイン → レポート確認 → リアルタイム更新確認
  - _Requirements: 全体的な要件カバレッジ_