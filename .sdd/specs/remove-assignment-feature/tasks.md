# 実装タスク

## 概要
ProjectAssignment機能を完全に削除し、システムを簡素化します。全ユーザーが全ての有効なプロジェクトに工数入力可能になります。

## タスクリスト

### Phase 1: バックエンド削除準備

- [ ] 1. ProjectAssignment関連の依存関係調査とコンパイルエラーの事前確認
  - ProjectAssignmentを参照している全クラスのリストアップ
  - 削除による影響範囲の特定
  - コンパイルエラーが発生する箇所の事前把握
  - _要件: 3.2, 3.4, 3.5_

### Phase 2: バックエンドコード削除（並列実行可能）

- [ ] 2.1 Domainレイヤーの削除
  - backend/src/main/java/com/devhour/domain/model/entity/ProjectAssignment.java を削除
  - backend/src/main/java/com/devhour/domain/repository/ProjectAssignmentRepository.java を削除
  - backend/src/main/java/com/devhour/domain/repository/ProjectAssignmentEventRepository.java を削除
  - backend/src/main/java/com/devhour/domain/event/ProjectAssignmentEvent.java を削除
  - _要件: 3.2, 10.1, 10.2_

- [ ] 2.2 Domainサービスの削除
  - backend/src/main/java/com/devhour/domain/service/ProjectAssignmentValidator.java を削除
  - backend/src/main/java/com/devhour/domain/service/SubordinateProjectAssignmentService.java を削除
  - backend/src/main/java/com/devhour/domain/service/ProjectAllocationService.java を削除
  - _要件: 3.3, 3.7, 10.5_

- [ ] 2.3 Applicationレイヤーの削除
  - backend/src/main/java/com/devhour/application/service/ProjectAssignmentApplicationService.java を削除
  - ProjectAllocationService への依存を削除
  - _要件: 3.4, 10.1_

- [ ] 2.4 Infrastructureレイヤーの削除
  - backend/src/main/java/com/devhour/infrastructure/repository/ProjectAssignmentRepositoryImpl.java を削除
  - backend/src/main/java/com/devhour/infrastructure/repository/ProjectAssignmentEventRepositoryImpl.java を削除
  - backend/src/main/java/com/devhour/infrastructure/mapper/ProjectAssignmentMapper.java を削除
  - backend/src/main/java/com/devhour/infrastructure/kafka/ProjectAssignmentEventPublisher.java を削除
  - _要件: 3.5, 3.6, 10.3, 10.4_

- [ ] 2.5 Presentationレイヤーの削除
  - backend/src/main/java/com/devhour/presentation/controller/ProjectAssignmentController.java を削除
  - backend/src/main/java/com/devhour/presentation/dto/request/ProjectAssignmentCreateRequest.java を削除
  - _要件: 3.1, 10.1_

### Phase 3: バックエンドテスト削除

- [ ] 3.1 ドメインテストの削除
  - backend/src/test/java/com/devhour/domain/model/entity/ProjectAssignmentTest.java を削除
  - backend/src/test/java/com/devhour/domain/service/ProjectAssignmentValidatorTest.java を削除
  - backend/src/test/java/com/devhour/domain/service/SubordinateProjectAssignmentServiceTest.java を削除
  - backend/src/test/java/com/devhour/domain/event/ProjectAssignmentEventTest.java を削除
  - _要件: 7.1, 7.3_

- [ ] 3.2 アプリケーション・インフラテストの削除
  - backend/src/test/java/com/devhour/application/service/ProjectAssignmentApplicationServiceTest.java を削除
  - backend/src/test/java/com/devhour/application/service/ProjectAssignmentApplicationServiceEventTest.java を削除
  - backend/src/test/java/com/devhour/infrastructure/repository/ProjectAssignmentRepositoryImplTest.java を削除
  - backend/src/test/java/com/devhour/infrastructure/repository/ProjectAssignmentEventRepositoryImplTest.java を削除
  - backend/src/test/java/com/devhour/infrastructure/mapper/ProjectAssignmentMapperTest.java を削除
  - backend/src/test/java/com/devhour/infrastructure/kafka/ProjectAssignmentEventPublisherTest.java を削除
  - backend/src/test/java/com/devhour/presentation/controller/ProjectAssignmentControllerTest.java を削除
  - _要件: 7.1, 7.3_

### Phase 4: バックエンドビルド確認

- [ ] 4.1 コンパイルエラーの解消
  - ./gradlew clean build を実行してコンパイルエラーを確認
  - 残存する参照箇所があれば削除または修正
  - _要件: 3.2_

- [ ] 4.2 テストカバレッジの確認
  - ./gradlew test を実行して全テストが通過することを確認
  - ./gradlew check でカバレッジ80%以上を維持していることを確認
  - _要件: 7.2_

### Phase 5: 工数入力ロジックの更新

- [ ] 5.1 WorkRecordApplicationService の更新テスト作成
  - プロジェクトアサインチェックなしで工数記録が作成できるテストを作成
  - 任意のユーザーが有効なプロジェクトに工数入力できることを確認するテストを作成
  - _要件: 5.2, 6.1, 7.2_

- [ ] 5.2 WorkRecordApplicationService の実装更新
  - アサインチェックロジックを削除（もし存在する場合）
  - プロジェクトの有効性チェックのみを実装
  - 認証状態のチェックのみでアクセスを許可
  - _要件: 5.1, 5.3, 6.2, 6.4_

- [ ] 5.3 ProjectService の更新
  - 全ての有効なプロジェクトを返すメソッドを確認または追加
  - アサイン関連のフィルタリングを削除
  - _要件: 1.1, 1.2_

### Phase 6: フロントエンド削除（並列実行可能）

- [ ] 6.1 Vueコンポーネントの削除
  - frontend/src/views/AssignmentView.vue を削除
  - frontend/src/components/assignment/ フォルダ全体を削除
    - AssignmentProjectToDeveloperDialog.vue
    - AssignmentSubordinateList.vue
    - AssignmentDeveloperDetailDialog.vue
    - AssignmentDeveloperToProjectDialog.vue
    - AssignmentProjectList.vue
  - _要件: 4.2, 10.6, 10.7_

- [ ] 6.2 サービス・コンポーザブルの削除
  - frontend/src/composables/useProjectAssignment.ts を削除
  - frontend/src/services/domains/project-assignment.service.ts を削除
  - _要件: 4.3, 10.8, 10.9_

- [ ] 6.3 フロントエンドテストの削除
  - frontend/src/views/__tests__/AssignmentView.spec.ts を削除
  - frontend/src/components/assignment/__tests__/AssignmentProjectToDeveloperDialog.spec.ts を削除
  - _要件: 7.4_

### Phase 7: フロントエンドルーティング・ナビゲーション更新

- [ ] 7.1 ルーティングの更新
  - frontend/src/router/index.ts から /assignments ルートを削除
  - アサイン関連のルートガードを削除
  - _要件: 4.4_

- [ ] 7.2 ナビゲーションメニューの更新
  - App.vue または該当するナビゲーションコンポーネントから「プロジェクトアサイン」メニュー項目を削除
  - アサイン関連のナビゲーションリンクを全て削除
  - _要件: 1.4, 4.4_

### Phase 8: 工数入力画面の更新

- [ ] 8.1 WorkRecordView コンポーネントのテスト更新
  - 全プロジェクトが選択可能であることを確認するテストを作成
  - アサインチェックなしで工数入力できることを確認するテストを作成
  - _要件: 5.1, 7.4_

- [ ] 8.2 WorkRecordView の実装更新
  - アサインされたプロジェクト取得ロジックを削除
  - projectService.getActiveProjects() を使用して全有効プロジェクトを取得
  - ドロップダウンに全プロジェクトを表示
  - _要件: 1.2, 5.1, 5.4_

- [ ] 8.3 工数入力サービスの更新
  - アサインチェック関連のAPIコールを削除
  - プロジェクト選択制限ロジックを削除
  - _要件: 5.2, 6.3_

### Phase 9: フロントエンドビルド確認

- [ ] 9.1 TypeScript型チェック
  - npm run type-check を実行してTypeScriptエラーがないことを確認
  - ProjectAssignment関連の型定義を削除
  - _要件: 4.2_

- [ ] 9.2 フロントエンドテスト実行
  - npm run test:unit を実行して単体テストが通過することを確認
  - npm run test:e2e を実行してE2Eテストが通過することを確認
  - _要件: 7.4_

### Phase 10: データベースマイグレーション

- [x] 10.1 バックアップマイグレーションスクリプトの作成
  - backend/src/main/resources/db/migration/V34__drop_project_assignments_table.sql を作成
  - project_assignmentsテーブルのバックアップを作成
  - インデックスと外部キー制約を削除
  - project_assignmentsテーブルを削除
  - _要件: 2.1, 2.2, 2.3, 2.4, 9.3_

- [x] 10.2 マイグレーション実行とテスト
  - ./gradlew flywayMigrate でマイグレーションを実行
  - データベース接続テストを実行
  - 既存の工数データが正しく表示されることを確認
  - _要件: 9.1, 9.4_

### Phase 11: 統合テスト

- [ ] 11.1 APIエンドポイントテスト
  - 削除されたProjectAssignment APIが404を返すことを確認
  - 工数入力APIがアサインチェックなしで動作することを確認
  - 全ユーザーが全プロジェクトにアクセスできることを確認
  - _要件: 1.3, 3.1_

- [ ] 11.2 フロントエンド統合テスト
  - ユーザーログイン後、全プロジェクトが表示されることを確認
  - 任意のプロジェクトに工数入力できることを確認
  - アサイン画面へのアクセスが適切にハンドリングされることを確認
  - _要件: 1.1, 1.2, 4.1_

### Phase 12: ドキュメント更新

- [ ] 12.1 技術ドキュメントの更新
  - README.md からProjectAssignment機能の記述を削除
  - CLAUDE.md のアーキテクチャセクションを更新
  - .sdd/steering/structure.md からProjectAssignment関連ファイルを削除
  - _要件: 8.1, 8.3_

- [ ] 12.2 API仕様書の更新
  - Swagger/OpenAPI仕様からProjectAssignmentエンドポイントを削除
  - APIドキュメントのサンプルを更新
  - _要件: 8.2_

### Phase 13: 最終検証

- [ ] 13.1 完全なシステムテスト
  - バックエンドの全テストスイートを実行（./gradlew test）
  - フロントエンドの全テストを実行（npm run test）
  - テストカバレッジが80%以上であることを確認
  - _要件: 7.2_

- [ ] 13.2 パフォーマンス検証
  - 全プロジェクト取得のパフォーマンスを測定
  - 工数入力のレスポンスタイムを確認
  - データベースクエリの実行計画を確認
  - _要件: 9.4_

## 並列実行可能なタスク

以下のタスクグループは並列で実行可能です：

**グループA（Phase 2）**: バックエンドコード削除
- タスク 2.1, 2.2, 2.3, 2.4, 2.5 は相互依存がないため並列実行可能

**グループB（Phase 6）**: フロントエンド削除
- タスク 6.1, 6.2, 6.3 は相互依存がないため並列実行可能

**グループC**: バックエンドとフロントエンドの削除
- Phase 2-4（バックエンド）とPhase 6-9（フロントエンド）は独立して実行可能

## 実装順序の推奨

1. **Phase 1**: 依存関係調査（必須最初のステップ）
2. **Phase 2-4 と Phase 6-9**: 並列実行（バックエンドとフロントエンド）
3. **Phase 5 と Phase 8**: 工数入力ロジック更新
4. **Phase 10**: データベースマイグレーション
5. **Phase 11-13**: 統合テストと最終検証

## 完了条件

- [ ] 全てのProjectAssignment関連コードが削除されている
- [ ] コンパイルエラーがない
- [ ] 全てのテストが通過する
- [ ] テストカバレッジ80%以上を維持
- [ ] 全ユーザーが全プロジェクトに工数入力可能
- [ ] ドキュメントが更新されている