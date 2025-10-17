# 実装計画

## フェーズ1: リポジトリ層の拡張

- [ ] 1.1 UserRepositoryインターフェースの拡張
  - `findByRoleAndStatus(User.Role role, boolean isActive)`メソッドをUserRepository.javaに追加
  - `findAllByRole(User.Role role)`メソッドをUserRepository.javaに追加
  - JavaDocコメントでメソッドの責務と使用目的を明記
  - _要件: 1.1, 5.1, 5.2_

- [ ] 1.2 UserRepositoryImplの実装拡張
  - UserRepositoryImplクラスに新しいメソッドの実装を追加
  - findByRoleAndStatusメソッドの実装（roleとis_activeでフィルタリング）
  - findAllByRoleメソッドの実装（roleのみでフィルタリング、ステータス不問）
  - _要件: 1.1, 5.1, 5.2_

- [ ] 1.3 UserMapperのMyBatisクエリ追加
  - UserMapper.javaに`@Select`アノテーションで新しいクエリメソッドを追加
  - findByRoleAndStatus: `SELECT * FROM users WHERE role = #{role} AND is_active = #{isActive} ORDER BY full_name`
  - findAllByRole: `SELECT * FROM users WHERE role = #{role} ORDER BY full_name`
  - 既存のTypeHandlerを活用してUser.Roleの変換を処理
  - _要件: 1.1, 1.2, 5.4_

## フェーズ2: DTOとレスポンスモデルの作成

- [ ] 2.1 AdminUserDtoクラスの作成
  - com.devhour.presentation.dto パッケージにAdminUserDto.javaを作成
  - フィールド: id, username, fullName, email, status (String), createdAt (LocalDateTime)
  - @JsonFormatアノテーションで日付フォーマット指定（yyyy-MM-dd'T'HH:mm:ss）
  - Lombokの@Data, @NoArgsConstructor, @AllArgsConstructorアノテーション使用
  - _要件: 3.1, 3.3_

- [ ] 2.2 AdminUserListResponseクラスの作成
  - com.devhour.presentation.dto パッケージにAdminUserListResponse.javaを作成
  - フィールド: users (List<AdminUserDto>), totalCount (int), timestamp (LocalDateTime)
  - @JsonFormatアノテーションでtimestampフィールドのフォーマット指定
  - Lombokアノテーションでボイラープレートコード削減
  - _要件: 3.1, 3.3_

## フェーズ3: アプリケーションサービス層の拡張

- [ ] 3.1 UserApplicationServiceへの管理者取得メソッド追加
  - `getAdminUsers(String status)`メソッドを実装（statusパラメータによる条件分岐）
  - status="ACTIVE"の場合: userRepository.findByRoleAndStatus(Role.PMO, true)を呼び出し
  - status="INACTIVE"の場合: userRepository.findByRoleAndStatus(Role.PMO, false)を呼び出し
  - statusがnullまたは空の場合: userRepository.findAllByRole(Role.PMO)を呼び出し
  - 不正なstatusパラメータの場合はIllegalArgumentExceptionをスロー
  - _要件: 1.1, 5.1, 5.2, 5.3_

- [ ] 3.2 UserApplicationServiceのヘルパーメソッド実装
  - `getAllAdminUsers()`メソッド: 全PMOユーザーを取得（findAllByRole使用）
  - `getActiveAdminUsers()`メソッド: アクティブPMOユーザーのみ取得
  - `getInactiveAdminUsers()`メソッド: 非アクティブPMOユーザーのみ取得
  - 各メソッドにトランザクション境界と適切なログ出力を追加
  - _要件: 1.1, 1.4, 5.4_

## フェーズ4: コントローラー層の実装

- [ ] 4.1 UserControllerへの管理者一覧エンドポイント追加
  - `@GetMapping("/admin")`アノテーションでエンドポイントマッピング
  - `@PreAuthorize("hasRole('PMO')")`でPMOロール必須の認可設定
  - `getAdminUsers(@RequestParam(required = false) String status, HttpServletRequest request)`メソッド実装
  - UserApplicationService.getAdminUsers(status)を呼び出してユーザーリスト取得
  - _要件: 1.1, 2.1, 2.3_

- [ ] 4.2 DTOへの変換ロジック実装
  - 取得したUserエンティティリストをAdminUserDtoリストに変換
  - User.isActive()の値を"ACTIVE"/"INACTIVE"文字列に変換してstatusフィールドに設定
  - AdminUserListResponseオブジェクトを構築（users, totalCount, timestamp設定）
  - ResponseEntity.ok()でHTTP 200レスポンスとして返却
  - _要件: 1.2, 3.1, 3.3_

- [ ] 4.3 エラーハンドリングとログ出力の実装
  - リクエスト受信時のログ出力（userId, role, statusパラメータを含む）
  - 不正なstatusパラメータの場合のエラーレスポンス（HTTP 400）
  - 成功時のレスポンスログ（取得件数、処理時間を含む）
  - SecurityContextから現在のユーザー情報を取得してログに記録
  - _要件: 2.4, 4.3, 4.4_

## フェーズ5: テストの実装

- [ ] 5.1 UserRepositoryのテスト作成
  - UserRepositoryTestクラスにfindByRoleAndStatusのテストケース追加
  - PMOロール＋アクティブ、PMOロール＋非アクティブ、全PMOユーザー取得のテスト
  - 結果の件数、ソート順、返却されるユーザーの属性を検証
  - H2インメモリデータベースでテストデータを準備
  - _要件: 1.1, 5.1, 5.2_

- [ ] 5.2 UserApplicationServiceの単体テスト作成
  - UserApplicationServiceTestクラスに管理者取得メソッドのテスト追加
  - Mockitoを使用してUserRepositoryをモック化
  - 各statusパラメータ値での動作を検証（ACTIVE, INACTIVE, null, 不正値）
  - 例外処理のテスト（IllegalArgumentException）
  - _要件: 1.1, 5.1, 5.2, 5.3_

- [ ] 5.3 UserControllerの統合テスト作成
  - UserControllerTestクラスにgetAdminUsersエンドポイントのテスト追加
  - @WithMockUser(roles = "PMO")でPMOロールでのアクセステスト
  - @WithMockUser(roles = "DEVELOPER")で権限不足（403）のテスト
  - 認証なしでのアクセス（401）のテスト
  - MockMvcを使用してHTTPリクエスト/レスポンスを検証
  - _要件: 2.1, 2.2, 2.3_

- [ ] 5.4 エンドツーエンドテストの実装
  - 実際のデータベースとSpring Bootアプリケーションを起動
  - JWTトークンを使用した認証フローのテスト
  - /api/users/adminエンドポイントへの実際のHTTPリクエスト
  - レスポンスのJSON構造とデータ内容の検証
  - パフォーマンス測定（レスポンスタイムが200ms以内であることを確認）
  - _要件: 1.1, 2.1, 3.1, 3.2_

## フェーズ6: 統合とカバレッジ確認

- [ ] 6.1 全体統合とリファクタリング
  - 全てのコンポーネントが正しく連携することを確認
  - 重複コードの除去とメソッド抽出によるリファクタリング
  - ログ出力の一貫性確認（フォーマット、レベル）
  - エラーメッセージの日本語化と統一
  - _要件: 全要件_

- [ ] 6.2 テストカバレッジの確認と改善
  - `./gradlew test`でテスト実行
  - `./gradlew jacocoTestReport`でカバレッジレポート生成
  - 80%以上のカバレッジを達成していることを確認
  - カバレッジが不足している箇所に追加テストを作成
  - `./gradlew check`で最終的なビルド検証
  - _要件: 全要件_