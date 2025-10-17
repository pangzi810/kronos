# 実装計画

## 削除済みユーザー一覧取得API実装タスク

- [ ] 1. DTOクラスの作成とテスト実装
- [ ] 1.1 DeletedUserSearchCriteriaクラスの実装
  - 削除済みユーザー検索条件DTOのテストクラスを作成
  - バリデーションのテストケース（size > 100、search < 3文字）を実装
  - DeletedUserSearchCriteriaクラスを実装（ページング、フィルタ、検索パラメータ）
  - getOffset()メソッドとvalidate()メソッドを実装
  - _要件: REQ-3.1, 3.2, 4.1, 5.1_

- [ ] 1.2 DeletedUserListResponseクラスの実装
  - DeletedUserListResponseのテストクラスを作成
  - レスポンス構造のテストケースを実装
  - DeletedUserListResponse、DeletedUserDto、PageMetadataクラスを実装
  - 既存のAdminUserListResponseパターンに準拠
  - _要件: REQ-2.1, 2.2, 2.3, 2.4, 3.3, 3.4_

- [ ] 2. MyBatisマッパーの拡張とリポジトリ実装
- [ ] 2.1 UserMapperインターフェースの拡張
  - UserMapperTestクラスに削除済みユーザー取得のテストケースを追加
  - selectDeletedUsersメソッドのテスト（動的SQL条件）を実装
  - countDeletedUsersメソッドのテストを実装
  - UserMapperにselectDeletedUsersとcountDeletedUsersメソッドを追加
  - MyBatis動的SQLアノテーションで条件分岐を実装
  - _要件: REQ-4.2, 4.3, 4.4, 5.2, 5.3, 5.4_

- [ ] 2.2 UserRepositoryインターフェースとUserRepositoryImplの拡張
  - UserRepositoryImplTestに削除済みユーザー取得のテストを追加
  - findDeletedUsersとcountDeletedUsersメソッドのテストを実装
  - UserRepositoryインターフェースに新メソッドを定義
  - UserRepositoryImplでマッパーを呼び出す実装を追加
  - _要件: REQ-1.1, 4.1, 4.5, 5.1_

- [ ] 3. アプリケーションサービス層の実装
- [ ] 3.1 UserApplicationServiceの拡張
  - UserApplicationServiceTestに削除済みユーザー取得のテストを追加
  - findDeletedUsers正常系テスト（ページング、フィルタリング、検索）を実装
  - countDeletedUsersのテストケースを実装
  - UserApplicationServiceにfindDeletedUsersとcountDeletedUsersメソッドを実装
  - ビジネスロジックとエラーハンドリングを含む
  - _要件: REQ-1.2, 3.1, 3.2, 4.1, 4.2, 4.3, 5.1, 5.2_

- [ ] 3.2 ページネーション処理の実装
  - ページネーション計算のユニットテストを作成
  - DeletedUsersResult DTOクラスを作成（ページング結果を保持）
  - UserApplicationServiceでページメタデータ計算ロジックを実装
  - hasNext、hasPrevious判定ロジックを追加
  - _要件: REQ-3.1, 3.2, 3.3, 3.4_

- [ ] 4. プレゼンテーション層の実装
- [ ] 4.1 UserControllerの拡張
  - UserControllerTestにgetDeletedUsersエンドポイントのテストを追加
  - PMOロールでのアクセス成功テストを実装
  - DEVELOPERロールでの403エラーテストを実装
  - 認証なしでの401エラーテストを実装
  - UserControllerにgetDeletedUsersメソッドを実装
  - @PreAuthorize("hasRole('PMO')")アノテーションを設定
  - _要件: REQ-1.1, 1.2, 1.3, 1.4, 6.1, 6.2_

- [ ] 4.2 エラーハンドリングの拡張
  - GlobalExceptionHandlerTestにテストケースを追加
  - IllegalArgumentException（400）のテストを実装
  - AccessDeniedException（403）のテストを実装
  - DataSourceLookupFailureException（503）のテストを実装
  - GlobalExceptionHandlerに新しい例外ハンドラを追加
  - _要件: REQ-6.1, 6.2, 6.3, 6.4_

- [ ] 5. 監査ログ機能の実装
- [ ] 5.1 AuditLoggingAspectの作成
  - AuditLoggingAspectTestクラスを作成
  - 削除済みユーザーアクセスログのテストを実装
  - 成功時と失敗時のログ出力テストを実装
  - AuditLoggingAspectクラスを実装
  - @AroundアドバイスでgetDeletedUsersメソッドをインターセプト
  - _要件: REQ-7.1, 7.2, 7.3_

- [ ] 5.2 ログバック設定の更新
  - logback-spring.xmlにAUDITロガーの設定を追加
  - 監査ログ用の専用ファイルアペンダーを設定
  - ログローテーション設定を追加
  - _要件: REQ-7.1, 7.2_

- [ ] 6. 統合テストとエンドツーエンドテスト
- [ ] 6.1 統合テストの実装
  - DeletedUsersApiIntegrationTestクラスを作成
  - 完全なAPIフロー（認証→リクエスト→レスポンス）のテストを実装
  - ページネーション動作の統合テストを実装
  - フィルタリングと検索の統合テストを実装
  - JWTトークン生成とリクエストヘッダー設定を含む
  - _要件: REQ-1.1, 1.2, 3.1, 4.1, 5.1, 6.1, 7.1_

- [ ] 6.2 E2Eテストの実装
  - DeletedUsersE2ETestクラスを作成
  - TestContainersでMySQLコンテナを起動
  - 実データベースでの削除済みユーザー作成と取得テストを実装
  - ページング、フィルタリング、検索の実動作確認テストを実装
  - 監査ログ出力の確認テストを実装
  - _要件: REQ-1.1, 2.1, 3.1, 4.1, 5.1, 6.1, 7.1_

- [ ] 7. 最終統合と検証
- [ ] 7.1 全コンポーネントの統合確認
  - 全テストスイートの実行（./gradlew test）
  - JaCoCoカバレッジレポートの生成と80%以上の確認
  - API動作確認スクリプトtest-scripts/test-deleted-users.shを作成
  - curlコマンドで実際のAPIエンドポイントをテスト
  - _要件: 全要件の統合確認_