# 実装計画

- [ ] 1. ドメイン層：重複検証ロジック実装
  - [ ] 1.1 DuplicateAssignmentException ドメイン例外クラス作成
    - DomainException を継承するカスタム例外クラスを作成
    - プロジェクト名・ユーザー名を含むコンストラクタ実装
    - com.devhour.domain.exception パッケージに配置
    - _要件: REQ-2.1, REQ-2.2, REQ-2.3_

  - [ ] 1.2 ProjectAssignmentValidator ドメインサービス実装
    - テスト駆動でバリデータクラスの単体テストを先に作成
    - validateDuplicateAssignment メソッドの実装
    - validateAssignmentUpdate メソッドの実装（更新時用）
    - ProjectAssignmentRepository への依存注入設定
    - _要件: REQ-1.1, REQ-1.2, REQ-1.3, REQ-3.1, REQ-3.2_

  - [ ] 1.3 ProjectAssignment エンティティの拡張
    - 既存 ProjectAssignment エンティティに validateUniqueness メソッド追加
    - isActiveAssignment メソッドでアクティブ状態判定機能追加
    - ドメインロジックをエンティティに集約
    - _要件: REQ-1.4, REQ-1.5, REQ-3.5_

- [ ] 2. インフラストラクチャ層：データアクセス拡張
  - [ ] 2.1 ProjectAssignmentMapper 拡張
    - countActiveByProjectAndUser メソッドを MyBatis マッパーに追加
    - 効率的な重複チェック SQL 実装（INDEX 活用）
    - @Select アノテーションでパラメータ化クエリ作成
    - _要件: REQ-4.1, REQ-4.2, REQ-4.5_

  - [ ] 2.2 ProjectAssignmentRepositoryImpl 拡張
    - findActiveByProjectAndUser メソッド実装
    - 重複チェック用リポジトリメソッド追加
    - テスト駆動で Repository テストを先に作成
    - MyBatis マッパーとの統合テスト実装
    - _要件: REQ-4.1, REQ-4.3, REQ-4.4_

  - [ ] 2.3 データベースインデックス最適化（データベース制約は不要）
    - 既存テーブルのクエリパフォーマンス分析
    - 重複チェッククエリの実行計画確認
    - 必要に応じた既存インデックスの活用
    - _要件: REQ-4.1, REQ-4.5_

- [ ] 3. アプリケーション層：業務ロジック統合
  - [ ] 3.1 ProjectAssignmentApplicationService 拡張
    - createAssignment メソッドに重複検証ロジック統合
    - updateAssignment メソッドに重複チェック追加
    - @Transactional アノテーションで原子性保証
    - テスト駆動でアプリケーションサービステストを先に実装
    - _要件: REQ-3.1, REQ-3.2, REQ-3.3, REQ-4.3_

  - [ ] 3.2 エラーハンドリング統合
    - DuplicateAssignmentException のキャッチと適切な処理
    - 日本語エラーメッセージの組み立て
    - プロジェクト名・ユーザー名取得のための Repository 呼び出し
    - _要件: REQ-2.1, REQ-2.2, REQ-2.3, REQ-3.4_

- [ ] 4. プレゼンテーション層：API エラーレスポンス
  - [ ] 4.1 グローバル例外ハンドラー実装
    - @ControllerAdvice でグローバル例外ハンドリング設定
    - DuplicateAssignmentException 専用のハンドラーメソッド追加
    - 400 Bad Request ステータスコードとエラー JSON レスポンス返却
    - ErrorResponse DTO と DuplicateAssignmentErrorDetails DTO 作成
    - _要件: REQ-2.1, REQ-2.4, REQ-2.5_

  - [ ] 4.2 ProjectAssignmentController API テスト拡張
    - 重複作成試行時の 400 エラーレスポンステスト追加
    - 日本語エラーメッセージの検証テスト実装
    - 正常ケースと異常ケースの統合テスト作成
    - _要件: REQ-2.4, REQ-3.1, REQ-3.2_

- [ ] 5. 包括的テスト実装
  - [ ] 5.1 単体テスト完全実装
    - ProjectAssignmentValidatorTest：全分岐カバレッジ達成
    - DuplicateAssignmentExceptionTest：例外詳細情報検証
    - 各層のモックを使用した単体テストで 80% カバレッジ達成
    - JUnit 5 + Mockito でのテスト実装
    - _要件: REQ-5.1, REQ-5.2, REQ-5.3_

  - [ ] 5.2 統合テスト実装
    - @SpringBootTest での API エンドポイント統合テスト
    - データベース制約違反時の例外ハンドリングテスト
    - 同時リクエスト処理での競合状態テスト
    - H2 テストデータベースでの統合テスト実行
    - _要件: REQ-5.2, REQ-5.4, REQ-4.3_

  - [ ] 5.3 エンドツーエンドテスト
    - create-project.sh テストスクリプトの拡張
    - 重複割り当て試行シナリオの追加
    - JWT 認証 → 割り当て作成 → 重複試行 → エラー確認フロー
    - API レスポンス形式の検証
    - _要件: REQ-5.2, REQ-5.4, REQ-5.5_

- [ ] 6. 品質保証と統合検証
  - [ ] 6.1 JaCoCo カバレッジ検証
    - ./gradlew check で 80% カバレッジ達成確認
    - カバレッジレポートで未テスト箇所の特定と追加テスト実装
    - domain パッケージの高カバレッジ（90%）達成
    - _要件: REQ-5.5_

  - [ ] 6.2 パフォーマンス検証
    - 重複チェッククエリの実行時間測定（目標：50ms以下）
    - 大量データでのパフォーマンステスト実行
    - インデックス効果の確認
    - _要件: REQ-4.1, REQ-4.5_

  - [ ] 6.3 全機能統合検証
    - 既存の ProjectAssignmentApplicationService 機能に影響がないことを確認
    - バックワード互換性の検証
    - 重複防止機能が既存 API フローに正しく統合されていることを確認
    - Spring Security JWT 認証との統合動作確認
    - _要件: REQ-1.1, REQ-2.1, REQ-3.1, REQ-4.1, REQ-5.1_