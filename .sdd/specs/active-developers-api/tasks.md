# 実装計画

## 概要
アクティブな開発者一覧を返却するAPI `/api/users/active/developers` の実装。既存のDDDアーキテクチャ（Domain-Driven Design）に準拠し、Spring Boot + MyBatis技術スタックとの統合を重視してテスト駆動開発で段階的に実装する。

## 実装タスク

### 1. レスポンス用DTOクラスの実装
- [ ] **1.1 ActiveDeveloperResponseクラスの作成**
  - `com.devhour.application.dto.ActiveDeveloperResponse` クラスを作成
  - JSON シリアライゼーション用の `@JsonProperty` アノテーションを設定
  - Lombok アノテーション（`@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`）を使用
  - ネストクラス `DeveloperInfo` を含む構造を実装
  - 要件マッピング: _要件4.1, 要件4.2_

- [ ] **1.2 ActiveDeveloperResponse DTOのユニットテストの実装**
  - `ActiveDeveloperResponseTest` クラスを作成
  - JSON シリアライゼーション/デシリアライゼーションのテストを実装
  - Builder パターンの動作確認テストを実装
  - null値処理のテストを実装
  - 要件マッピング: _要件4.1, 要件4.3_

### 2. ドメイン層の拡張
- [ ] **2.1 UserRepositoryインターフェースの拡張**
  - `com.devhour.domain.repository.UserRepository` に `findActiveDevelopers()` メソッドを追加
  - メソッドシグネチャ: `List<User> findActiveDevelopers()`
  - Javadocコメントを追加してビジネスロジックを明確化
  - 要件マッピング: _要件2.1, 要件2.2_

### 3. インフラストラクチャ層の実装
- [ ] **3.1 UserMapperの拡張**
  - `com.devhour.infrastructure.mapper.UserMapper` に `selectActiveDevelopers()` メソッドを追加
  - MyBatis `@Select` アノテーションでSQLクエリを実装
  - SQL: `SELECT id, name, email FROM users WHERE is_active = true AND role = 'DEVELOPER' ORDER BY name ASC`
  - パフォーマンス最適化：必要なフィールドのみ選択
  - 要件マッピング: _要件2.1, 要件2.2, 要件2.3, 要件2.4_

- [ ] **3.2 UserRepositoryImplの拡張**
  - `com.devhour.infrastructure.repository.UserRepositoryImpl` に `findActiveDevelopers()` メソッドを実装
  - UserMapper.selectActiveDevelopers()を呼び出す処理を実装
  - 例外処理とログ記録を追加
  - 要件マッピング: _要件2.1, 要件2.2, 要件6.2_

- [ ] **3.3 データアクセス層のユニットテストの実装**
  - `UserMapperTest` クラスでMyBatisマッパーのテストを実装
  - テストデータベース（H2）を使用したアクティブ開発者フィルタリングのテスト
  - 境界値テスト：is_active=false, role='PMO'のユーザーが除外されることを確認
  - `UserRepositoryImplTest` クラスでリポジトリ実装のテストを実装
  - 要件マッピング: _要件2.3, 要件2.4_

### 4. アプリケーション層の実装
- [ ] **4.1 UserApplicationServiceの拡張**
  - `com.devhour.application.service.UserApplicationService` に `findActiveDevelopers()` メソッドを追加
  - `@Transactional(readOnly = true)` アノテーションを設定
  - UserRepository.findActiveDevelopers()を呼び出すビジネスロジックを実装
  - ログ記録（実行時間とデータ件数）を追加
  - 要件マッピング: _要件2.1, 要件7.2_

- [ ] **4.2 キャッシュ機能の実装**
  - UserApplicationServiceのfindActiveDevelopers()メソッドに `@Cacheable` アノテーションを追加
  - キャッシュ名: "activeDevelopers", キー: "all"
  - キャッシュ無効化メソッド `evictActiveDevelopersCache()` を実装（`@CacheEvict`）
  - 要件マッピング: _要件6.1, 要件6.3_

- [ ] **4.3 キャッシュ設定クラスの作成**
  - `com.devhour.config.CacheConfig` クラスを作成
  - `@EnableCaching` と Caffeine キャッシュマネージャーを設定
  - TTL: 5分、最大サイズ: 1000エントリ、統計記録有効化
  - 要件マッピング: _要件6.1, 要件6.3_

- [ ] **4.4 アプリケーション層のユニットテストの実装**
  - `UserApplicationServiceTest` クラスでfindActiveDevelopers()メソッドのテストを実装
  - Mockito を使用したUserRepositoryのモック化
  - 正常系テスト：アクティブ開発者リストが正しく返されることを確認
  - キャッシュ動作のテスト：2回目の呼び出しでキャッシュが使用されることを確認
  - 要件マッピング: _要件2.1, 要件6.1_

### 5. プレゼンテーション層の実装
- [ ] **5.1 UserControllerの拡張**
  - `com.devhour.presentation.controller.UserController` に `getActiveDevelopers()` メソッドを追加
  - `@GetMapping("/active/developers")` アノテーションを設定
  - Spring Security `@PreAuthorize("hasRole('USER')")` で認証制御
  - UserApplicationService.findActiveDevelopers()を呼び出し
  - User エンティティリストをActiveDeveloperResponse DTOに変換
  - アクセスログの記録を実装
  - 要件マッピング: _要件1.1, 要件1.2, 要件1.3, 要件1.4, 要件3.2, 要件7.1_

- [ ] **5.2 DTO変換ユーティリティの実装**
  - UserControllerにprivateメソッド `buildResponse()` を実装
  - `List<User>` から `ActiveDeveloperResponse` への変換ロジック
  - developersリストとtotalCountの設定
  - null値とemptyリストの適切な処理
  - 要件マッピング: _要件4.1, 要件4.3_

### 6. エラーハンドリングとセキュリティ
- [ ] **6.1 GlobalExceptionHandlerの拡張**
  - 既存の `com.devhour.presentation.exception.GlobalExceptionHandler` を確認
  - 必要に応じてDataAccessExceptionのハンドリングを追加
  - ErrorResponse DTOとの統合確認
  - 要件マッピング: _要件5.1, 要件5.2, 要件5.3_

- [ ] **6.2 セキュリティ設定の確認と調整**
  - `com.devhour.config.SecurityConfig` で新しいエンドポイントの認証設定を確認
  - `/api/users/active/developers` へのアクセス権限設定
  - 既存のJWT認証フィルターとの統合確認
  - 要件マッピング: _要件3.1, 要件3.2, 要件3.3, 要件3.4_

### 7. 統合テストとAPI テスト
- [ ] **7.1 コントローラ統合テストの実装**
  - `UserControllerIntegrationTest` クラスを拡張
  - `@SpringBootTest` と `@AutoConfigureTestDatabase` を使用
  - 認証ありでのAPIコール成功テスト（HTTP 200 + 正しいJSONレスポンス）
  - 認証なしでのAPIコール失敗テスト（HTTP 401）
  - 無効JWTでのAPIコール失敗テスト（HTTP 401）
  - レスポンス構造の検証（開発者情報とtotalCount）
  - 要件マッピング: _要件1.4, 要件3.1, 要件3.3, 要件4.1, 要件4.2_

- [ ] **7.2 エンドツーエンドテストの実装**
  - `ActiveDevelopersApiE2ETest` クラスを作成
  - 実際のデータベース（テスト用MySQL）を使用
  - ユーザーログイン → JWT取得 → API呼び出しの完全フロー
  - 複数のユーザータイプでのテスト（PMOとDEVELOPER）
  - 要件マッピング: _要件1.1, 要件2.1, 要件2.2, 要件3.2_

### 8. パフォーマンステスト
- [ ] **8.1 パフォーマンステストの実装**
  - `ActiveDevelopersApiPerformanceTest` クラスを作成
  - レスポンス時間要件（2秒以内）の検証テスト
  - 同時接続テスト（100ユーザー）
  - データベースクエリ実行時間の測定
  - キャッシュ効果の性能測定
  - 要件マッピング: _要件6.1, 要件6.2, 要件6.4_

- [ ] **8.2 データベース最適化の実装**
  - パフォーマンス最適化用インデックスの作成
  - `CREATE INDEX idx_users_active_role ON users(is_active, role);` 
  - Flyway マイグレーションスクリプトとして実装（V7__add_active_role_index.sql）
  - 要件マッピング: _要件6.2, 要件6.4_

### 9. 監視とログ機能
- [ ] **9.1 アクセスログとメトリクスの実装**
  - UserController.getActiveDevelopers()にアクセスログ追加
  - 実行時間測定とログ記録
  - Spring Boot Actuator メトリクスとの統合
  - ログレベルとフォーマットの統一
  - 要件マッピング: _要件7.1, 要件7.2, 要件7.3_

- [ ] **9.2 ヘルスチェック機能の実装**
  - Spring Boot Actuator のヘルスインディケーター設定
  - データベース接続状態の監視
  - キャッシュ統計情報の監視
  - 要件マッピング: _要件7.4_

### 10. 最終統合とテストカバレッジ検証
- [ ] **10.1 全機能の統合テスト**
  - 全レイヤー（Presentation → Application → Domain → Infrastructure）の統合確認
  - 設計書のシーケンス図に基づく処理フローの検証
  - エラーケース（DB接続エラー、認証エラー等）の処理確認
  - 要件マッピング: _要件1.1, 要件1.2, 要件1.3, 要件1.4_

- [ ] **10.2 JaCoCo テストカバレッジの検証**
  - `./gradlew test jacocoTestReport` でカバレッジレポート生成
  - 80%最小カバレッジ要件の確認
  - `./gradlew check` でカバレッジ検証の実行
  - 未カバー箇所の特定と追加テスト作成
  - 要件マッピング: _テスト要件 - 80%カバレッジ_

- [ ] **10.3 API仕様書の更新**
  - 新しいエンドポイント `/api/users/active/developers` の Swagger 仕様追加
  - リクエスト/レスポンス例の記載
  - エラーコードとステータスコードの文書化
  - 要件マッピング: _要件1.4, 要件4.1, 要件4.4_

## 完了基準
- [ ] **全ての自動テストが成功**（ユニット、統合、E2E、パフォーマンス）
- [ ] **JaCoCo カバレッジ80%以上達成**
- [ ] **既存機能への影響がないことを確認**
- [ ] **API レスポンス時間が2秒以内**
- [ ] **認証・認可が適切に動作**
- [ ] **エラーハンドリングが要件通りに動作**
- [ ] **キャッシュ機能が正常に動作**

---
**ステータス**: 実装計画完了  
**開始準備**: 全承認完了、実装開始可能