# 実装タスク

## 概要
工数承認機能の実装タスクを段階的に定義します。各タスクは独立して実装・テスト可能な単位で分割され、テストファーストアプローチを採用しています。

## 前提条件
- ✅ 要件定義書承認済み
- ✅ 技術設計書承認済み
- 既存システム: Spring Boot 3.5.4 + DDD アーキテクチャ
- テストカバレッジ: 80%以上（JaCoCo）

## フェーズ1: ドメインモデル基盤実装

### Task 1.1: 承認ステータスValue Object実装
- [ ] ApprovalStatusTest.java作成 - PENDING/APPROVED/REJECTED状態遷移のテストケース
- [ ] ApprovalStatus.java実装 - enum型Value Object、状態遷移バリデーション含む
- [ ] ApprovalStatusTypeHandlerTest.java作成 - MyBatisとの連携テスト
- [ ] ApprovalStatusTypeHandler.java実装 - VARCHAR⇔Enum変換処理
- [ ] TypeHandler設定をMyBatisConfigに追加
- _Requirements: 1.1, 1.4, 1.5_

### Task 1.2: 上長関係エンティティ実装
- [ ] SupervisorRelationshipTest.java作成 - 関係設定・循環参照検証テスト
- [ ] SupervisorRelationship.java実装 - エンティティクラス（id, supervisorId, subordinateId, isActive, createdAt, updatedAt）
- [ ] 自己参照防止ロジック実装 - 同一ユーザーの上長設定を禁止
- [ ] 循環参照検証メソッド実装 - A→B→C→Aのような循環を検出
- [ ] SupervisorRelationshipRepository.javaインターフェース定義
- _Requirements: 2.1, 2.2, 2.4, 2.8_

### Task 1.3: 承認履歴エンティティ実装
- [ ] ApprovalHistoryTest.java作成 - 履歴作成・不変性保証テスト
- [ ] ApprovalHistory.java実装 - エンティティクラス（id, workRecordId, approverId, status, comment, approvedAt）
- [ ] 不変性保証実装 - Setter非公開、Builder patternまたはFactory method採用
- [ ] 却下理由の必須バリデーション実装
- [ ] ApprovalHistoryRepository.javaインターフェース定義
- _Requirements: 1.7, 6.1, 6.2, 6.3_

### Task 1.4: WorkRecordエンティティ拡張
- [ ] WorkRecordTest.java拡張 - 承認ステータス関連テスト追加
- [ ] WorkRecord.javaに承認関連フィールド追加（approvalStatus, approverId）
- [ ] 承認状態による編集可否判定メソッド追加
- [ ] 承認待ち自動設定ロジック実装（新規作成時）
- [ ] 既存リポジトリインターフェース拡張
- _Requirements: 1.1, 1.9, 1.10_

## フェーズ2: データアクセス層実装

### Task 2.1: Flywayマイグレーションスクリプト作成
- [ ] V11__add_approval_status_to_work_records.sql作成 - work_recordsテーブル拡張
- [ ] V12__create_supervisor_relationships_table.sql作成 - 上長関係テーブル
- [ ] V13__create_approval_histories_table.sql作成 - 承認履歴テーブル
- [ ] インデックス定義追加（approval_status, approver_id, created_at）
- [ ] 既存データへのデフォルト値設定（approval_status = 'PENDING'）
- _Requirements: 1.1, 2.1, 6.2_

### Task 2.2: SupervisorRelationshipMapper実装
- [ ] SupervisorRelationshipMapperTest.java作成 - CRUD操作テスト
- [ ] SupervisorRelationshipMapper.java実装 - @Mapperアノテーション付与
- [ ] findBySupervisorId()実装 - 部下一覧取得
- [ ] findBySubordinateId()実装 - 上長取得
- [ ] findUsersWithoutSupervisor()実装 - 承認者未設定ユーザー検索
- [ ] SupervisorRelationshipRepositoryImpl.java実装
- _Requirements: 2.2, 2.4, 2.6_

### Task 2.3: ApprovalHistoryMapper実装
- [ ] ApprovalHistoryMapperTest.java作成 - 履歴検索・保存テスト
- [ ] ApprovalHistoryMapper.java実装 - @Mapperアノテーション付与
- [ ] insert()実装 - 履歴作成（UPDATE不可）
- [ ] findByWorkRecordId()実装 - 工数別履歴取得
- [ ] findByApproverId()実装 - 承認者別履歴取得
- [ ] findByDateRange()実装 - 期間別履歴取得
- [ ] ApprovalHistoryRepositoryImpl.java実装
- _Requirements: 6.2, 6.5_

### Task 2.4: WorkRecordMapper拡張
- [ ] WorkRecordMapperTest.java拡張 - 承認ステータス検索テスト追加
- [ ] findPendingByApproverId()実装 - 承認待ち一覧取得
- [ ] updateApprovalStatus()実装 - ステータス更新
- [ ] bulkUpdateApprovalStatus()実装 - 一括承認用
- [ ] countByApprovalStatus()実装 - ステータス別件数取得
- [ ] WorkRecordRepositoryImpl.java拡張
- _Requirements: 1.1, 3.2, 5.2_

## フェーズ3: ドメインサービス層実装

### Task 3.1: ApprovalAuthorityService実装
- [ ] ApprovalAuthorityServiceTest.java作成 - 権限検証テスト
- [ ] ApprovalAuthorityService.java実装 - ドメインサービスクラス
- [ ] canApprove()メソッド実装 - 承認権限チェック
- [ ] validateNotSelfApproval()実装 - 自己承認防止
- [ ] UnauthorizedApprovalException定義 - 権限なし例外
- [ ] SupervisorRelationshipRepositoryとの連携実装
- _Requirements: 2.8, 2.9, 6.7_

### Task 3.2: ApprovalWorkflowService実装
- [ ] ApprovalWorkflowServiceTest.java作成 - 承認フロー全体テスト
- [ ] ApprovalWorkflowService.java実装 - ワークフロー制御
- [ ] approve()メソッド実装 - 単体承認処理（権限チェック→更新→履歴作成）
- [ ] reject()メソッド実装 - 却下処理（理由必須→更新→履歴作成）
- [ ] bulkApprove()メソッド実装 - 一括承認（トランザクション制御）
- [ ] 状態遷移整合性チェック実装
- _Requirements: 1.4, 1.5, 1.6, 3.6_

## フェーズ4: アプリケーションサービス層実装

### Task 4.1: ApprovalApplicationService実装
- [ ] ApprovalApplicationServiceTest.java作成 - ユースケーステスト
- [ ] ApprovalApplicationService.java実装 - アプリケーションサービス
- [ ] getPendingApprovals()実装 - 承認待ち一覧取得（ページング対応）
- [ ] approveWorkRecord()実装 - 単体承認処理呼び出し
- [ ] rejectWorkRecord()実装 - 却下処理呼び出し
- [ ] bulkApproveWorkRecords()実装 - 一括承認処理
- [ ] DTOクラス定義（ApprovalRequest, ApprovalResponse）
- _Requirements: 3.2, 3.5, 3.6, 3.8_

### Task 4.2: SupervisorApplicationService実装
- [ ] SupervisorApplicationServiceTest.java作成 - 上長管理テスト
- [ ] SupervisorApplicationService.java実装 - 上長関係管理
- [ ] assignSupervisor()実装 - 上長設定（管理者権限必須）
- [ ] getSubordinates()実装 - 部下一覧取得
- [ ] getUnassignedUsers()実装 - 承認者未設定ユーザー取得
- [ ] changeSupervisor()実装 - 上長変更（承認待ち移管処理含む）
- [ ] DTOクラス定義（SupervisorAssignmentRequest, SubordinateResponse）
- _Requirements: 2.1, 2.3, 2.4, 2.6_

## フェーズ5: REST API層実装

### Task 5.1: ApprovalController実装
- [ ] ApprovalControllerTest.java作成 - REST APIテスト（MockMvc使用）
- [ ] ApprovalController.java実装 - @RestController定義
- [ ] GET /api/approvals/pending実装 - 承認待ち一覧API
- [ ] POST /api/approvals/{id}/approve実装 - 承認API
- [ ] POST /api/approvals/{id}/reject実装 - 却下API
- [ ] POST /api/approvals/bulk-approve実装 - 一括承認API
- [ ] JWT認証・上長権限チェック統合
- [ ] エラーハンドリング実装（@ExceptionHandler）
- _Requirements: 3.2, 3.5, 3.6, 3.8_

### Task 5.2: SupervisorController実装
- [ ] SupervisorControllerTest.java作成 - 管理APIテスト
- [ ] SupervisorController.java実装 - @RestController定義
- [ ] GET /api/supervisors/unassigned実装 - 未設定ユーザー一覧
- [ ] POST /api/supervisors/assign実装 - 上長設定API
- [ ] GET /api/supervisors/{id}/subordinates実装 - 部下一覧API
- [ ] PUT /api/users/{id}/supervisor実装 - 上長変更API
- [ ] Admin権限チェック実装（@PreAuthorize）
- _Requirements: 2.1, 2.3, 2.4, 2.6_

### Task 5.3: WorkRecordController拡張
- [ ] WorkRecordControllerTest.java拡張 - 承認情報テスト追加
- [ ] getWorkRecords()拡張 - レスポンスに承認ステータス追加
- [ ] 承認済み工数の編集制限実装 - PUTメソッドでのチェック
- [ ] 承認ステータスフィルタリング追加 - クエリパラメータ対応
- [ ] WorkRecordResponse拡張 - 承認情報フィールド追加
- _Requirements: 1.9, 1.10, 4.1, 4.3_

## フェーズ6: フロントエンドコンポーネント実装

### Task 6.1: 承認リストコンポーネント実装
- [ ] ApprovalList.spec.ts作成 - コンポーネントテスト（Vitest）
- [ ] ApprovalList.vue実装 - 承認待ち一覧表示
- [ ] フィルタリング機能実装（期間、プロジェクト、部下）
- [ ] ページネーション実装
- [ ] 一括選択チェックボックス実装
- [ ] approvalApi.ts作成 - API通信サービス
- _Requirements: 3.2, 3.5, 3.7_

### Task 6.2: 承認アクションコンポーネント実装
- [ ] ApprovalActions.spec.ts作成 - アクションボタンテスト
- [ ] ApprovalActions.vue実装 - 承認・却下ボタン
- [ ] 承認確認ダイアログ実装
- [ ] RejectionDialog.vue実装 - 却下理由入力モーダル
- [ ] 一括承認ボタン実装
- [ ] Vuetify3 UIコンポーネント統合
- _Requirements: 3.6, 3.8, 7.1, 7.2_

### Task 6.3: 承認ステータス表示コンポーネント実装
- [ ] ApprovalStatus.spec.ts作成 - ステータス表示テスト
- [ ] ApprovalStatus.vue実装 - ステータスアイコン・色分け
- [ ] ApprovalHistory.vue実装 - 承認履歴タイムライン
- [ ] ステータスバッジコンポーネント実装
- [ ] 長期承認待ちアラート表示実装
- _Requirements: 4.1, 4.2, 4.3, 5.7, 5.10_

### Task 6.4: 上長管理画面実装
- [ ] SupervisorManagement.spec.ts作成 - 管理画面テスト
- [ ] SupervisorManagement.vue実装 - 管理者向け上長設定
- [ ] 承認者未設定ユーザーリスト実装
- [ ] ドラッグ&ドロップ上長割り当てUI実装
- [ ] SubordinatesList.vue実装 - 部下一覧表示
- [ ] supervisorApi.ts作成 - 上長管理API通信
- _Requirements: 2.3, 2.6, 2.7_

### Task 6.5: ダッシュボード統合
- [ ] Dashboard.spec.ts拡張 - 承認関連ウィジェットテスト
- [ ] 承認待ち件数カード実装
- [ ] 承認進捗チャート実装（Chart.js使用）
- [ ] 部下の承認待ち一覧ウィジェット実装
- [ ] リアルタイム更新実装（WebSocket/ポーリング）
- _Requirements: 5.1, 5.3, 5.8_

## フェーズ7: 統合・E2Eテスト実装

### Task 7.1: バックエンド統合テスト実装
- [ ] ApprovalFlowIntegrationTest.java作成 - 承認フロー全体テスト
- [ ] 工数入力→承認待ち→承認完了フローテスト
- [ ] 却下→修正→再申請フローテスト
- [ ] 一括承認の整合性テスト
- [ ] 権限チェック統合テスト
- [ ] パフォーマンステスト（1000件一括処理）
- _Requirements: 全要件の統合検証_

### Task 7.2: フロントエンド統合テスト実装
- [ ] approval.integration.spec.ts作成 - UI統合テスト
- [ ] 承認画面操作フローテスト
- [ ] ダッシュボード更新テスト
- [ ] 上長設定→承認フローテスト
- [ ] エラーハンドリングテスト
- _Requirements: 7.1, 7.2_

### Task 7.3: E2Eテスト実装
- [ ] approval.e2e.spec.ts作成 - Playwright E2Eテスト
- [ ] 開発者の工数入力→上長承認シナリオ
- [ ] 管理者の上長設定→承認フローシナリオ
- [ ] 却下→修正→再承認シナリオ
- [ ] 複数ユーザー同時操作シナリオ
- [ ] CI/CDパイプライン統合設定
- _Requirements: 全ユーザーストーリーの検証_

### Task 7.4: セキュリティ・監査テスト実装
- [ ] SecurityAuditTest.java作成 - セキュリティテスト
- [ ] 権限昇格攻撃テスト
- [ ] SQLインジェクションテスト
- [ ] 監査ログ記録確認テスト
- [ ] 承認履歴の改ざん防止テスト
- [ ] GDPR/個人情報保護コンプライアンステスト
- _Requirements: 6.4, 6.7, 6.8_

## 完了チェックリスト

### 機能要件
- [ ] 全7要件カテゴリ実装完了
- [ ] 全93受入条件を満たす
- [ ] ユーザーストーリー全実現

### 品質要件
- [ ] JaCoCoカバレッジ80%以上
- [ ] 全APIエンドポイントテスト完了
- [ ] E2Eテスト自動化完了

### 非機能要件
- [ ] 1000件一括処理3秒以内
- [ ] 監査ログ完全性保証
- [ ] WCAG 2.1 Level AA準拠