# Requirements Document

## はじめに

このドキュメントは、既存の Development Hour Management Tool にOkta認証システムを統合するための要件を定義します。現在のJWTベースの認証システムとロールベースアクセス制御（RBAC）を、OktaのOpenID Connect (OIDC) によるシングルサインオン（SSO）とOAuth2スコープベースアクセス制御に完全に置き換えることで、セキュリティの強化、中央集権的なID管理、より細かい権限制御、そして組織全体の認証統合を実現します。

### 技術選定の理由

**OAuth2/OIDC を採用する理由：**
- **モダンアーキテクチャとの親和性**: Spring Boot + Vue 3のRESTful API設計に最適
- **スコープベース権限の自然な実装**: OAuth2のスコープ概念をそのまま3階層権限管理に活用
- **JSON/JWT形式**: 既存のJWT実装との互換性が高く、移行が容易
- **将来の拡張性**: モバイルアプリやSPA対応が容易
- **実装のシンプルさ**: SAMLと比較してシンプルで開発・保守が容易

## Requirements

### Requirement 1: Okta OIDC認証フロー

**User Story:** 開発者として、OktaのOpenID Connect (OIDC)を使用してシステムにログインしたい。これにより、複数のパスワードを管理する必要がなくなり、組織の他のシステムと同じ認証情報を使用できるようになる。

#### Acceptance Criteria

1. WHEN ユーザーがログインページにアクセスした THEN システムは「Oktaでログイン」オプションを表示しなければならない
2. WHEN ユーザーが「Oktaでログイン」を選択した THEN システムはOkta認証プロバイダーへOIDC Authorization Code Flowでリダイレクトしなければならない
3. IF Oktaで認証が成功した THEN システムは認可コードを受け取り、バックチャンネルでID TokenとAccess Tokenを取得しなければならない
4. WHEN ID TokenとAccess Tokenを取得した THEN システムはJWT形式のトークンを検証し、ユーザーをアプリケーションのホームページへリダイレクトしなければならない
5. IF Oktaで認証が失敗した THEN システムはエラーメッセージを表示し、ログインページへリダイレクトしなければならない
6. WHERE OAuth2/OIDCが使用される THE SYSTEM SHALL PKCE (Proof Key for Code Exchange) フローを実装してセキュリティを強化しなければならない
7. WHEN トークンを受信した THEN システムはissuer、audience、signature、有効期限を検証しなければならない

### Requirement 2: ユーザープロビジョニングと同期

**User Story:** システム管理者として、Oktaでユーザーが作成・更新・削除された時に、システム内のユーザー情報が自動的に同期されることを望む。これにより、ユーザー管理の一元化と運用負荷の削減が実現できる。

#### Acceptance Criteria

1. WHEN Oktaから初めてログインするユーザーが存在する THEN システムは自動的にローカルユーザーレコードを作成しなければならない
2. IF Oktaユーザーのプロファイル情報（名前、メールアドレス）が更新された THEN システムはログイン時にローカルユーザー情報を同期しなければならない
3. WHEN Oktaでユーザーが無効化された AND そのユーザーがログインを試みた THEN システムはアクセスを拒否しなければならない
4. WHERE ユーザーがOktaから自動プロビジョニングされた THE SYSTEM SHALL Oktaで定義されたスコープのみを使用し、ローカルロールは割り当てない
5. IF SCIM 2.0プロトコルが設定されている THEN システムはリアルタイムのユーザープロビジョニング/デプロビジョニングをサポートしなければならない
6. WHEN 既存のローカルユーザーが初めてOktaでログインした THEN システムはメールアドレスに基づいてアカウントをリンクしなければならない

### Requirement 3: OAuth2スコープベースの権限管理（3階層構造）

**User Story:** システム管理者として、既存のロールベース認可を3階層構造のOAuth2スコープベース認可に置き換えたい。これにより、リソース・アクション・範囲の組み合わせで細かい権限制御と、将来的な権限拡張の柔軟性を確保できる。

#### Acceptance Criteria

1. WHEN OktaからAccess Tokenを受信した THEN システムはJWTペイロード内の`scp`または`scope`クレームから`{resource}:{action}:{scope}`形式のスコープ（例：`work-hours:read:own`、`projects:write:managed`）を抽出しなければならない
2. IF APIエンドポイントが呼び出された THEN システムはエンドポイントが要求するスコープとユーザーが持つスコープを比較して認可判断を行わなければならない
3. WHERE 既存のロールベースのチェックが存在する THE SYSTEM SHALL それらを3階層スコープベースのチェックに置き換えなければならない
4. WHEN Oktaでスコープが更新された THEN システムは次回のトークンリフレッシュまたは再ログイン時に新しいスコープを反映しなければならない
5. IF ユーザーが必要なスコープを持たない THEN システムはHTTP 403 Forbiddenレスポンスと不足しているスコープの情報を返さなければならない
6. WHILE システムが稼働している THE SYSTEM SHALL 以下の3階層スコープ体系を実装しなければならない：

   **工数管理スコープ (work-hours)**
   - `work-hours:read:own` - 自分の工数を閲覧
   - `work-hours:read:team` - チームメンバーの工数を閲覧
   - `work-hours:read:all` - 全ユーザーの工数を閲覧
   - `work-hours:write:own` - 自分の工数を入力・編集
   - `work-hours:write:team` - チームメンバーの工数を代理入力
   - `work-hours:approve:team` - チームメンバーの工数を承認
   - `work-hours:approve:all` - 全ユーザーの工数を承認

   **プロジェクト管理スコープ (projects)**
   - `projects:read:assigned` - 自分がアサインされたプロジェクトを閲覧
   - `projects:read:managed` - 自分が管理するプロジェクトを閲覧
   - `projects:read:all` - 全プロジェクトを閲覧
   - `projects:write:managed` - 管理プロジェクトを編集
   - `projects:write:all` - 全プロジェクトを編集
   - `projects:create:own` - プロジェクトを作成（自分が管理者）
   - `projects:assign:managed` - 管理プロジェクトにメンバーをアサイン
   - `projects:assign:all` - 全プロジェクトにメンバーをアサイン

   **ユーザー管理スコープ (users)**
   - `users:read:own` - 自分のプロファイルを閲覧
   - `users:read:team` - チームメンバーの情報を閲覧
   - `users:read:all` - 全ユーザーの情報を閲覧
   - `users:write:own` - 自分のプロファイルを編集
   - `users:write:all` - 全ユーザーの情報を編集
   - `users:create:standard` - 標準権限のユーザーを作成
   - `users:delete:all` - ユーザーを削除

   **レポートスコープ (reports)**
   - `reports:read:own` - 自分の実績レポートを閲覧
   - `reports:read:team` - チームレポートを閲覧
   - `reports:read:all` - 全体レポートを閲覧
   - `reports:export:team` - チームデータをエクスポート
   - `reports:export:all` - 全データをエクスポート

   **システム管理スコープ (system)**
   - `system:read:config` - システム設定を閲覧
   - `system:write:config` - システム設定を変更
   - `system:manage:integration` - 外部連携（JIRA/Okta）を管理

7. IF ワイルドカードスコープが使用される（例：`work-hours:*:own`、`*:read:all`） THEN システムは該当するすべての権限を付与しなければならない

### Requirement 4: セッション管理とログアウト

**User Story:** ユーザーとして、Oktaのグローバルログアウト機能を使用して、すべての連携アプリケーションから同時にログアウトしたい。これにより、セキュリティが向上する。

#### Acceptance Criteria

1. WHEN ユーザーがアプリケーションからログアウトした THEN システムはOktaのログアウトエンドポイントを呼び出し、グローバルログアウトを実行しなければならない
2. IF Oktaセッションが期限切れになった THEN システムは再認証を要求しなければならない
3. WHILE ユーザーがOkta経由でログインしている THE SYSTEM SHALL Oktaのセッションタイムアウト設定を遵守しなければならない
4. WHEN シングルログアウト（SLO）リクエストをOktaから受信した THEN システムはローカルセッションを無効化しなければならない
5. IF `ENABLE_SESSION_SYNC`が有効な場合 THEN システムは定期的にOktaセッションの有効性を確認しなければならない
6. WHERE リフレッシュトークンが利用可能な場合 THE SYSTEM SHALL 自動的にアクセストークンを更新しなければならない

### Requirement 5: セキュリティ要件

**User Story:** セキュリティ管理者として、Okta OIDC統合が組織のセキュリティ基準を満たし、監査可能であることを確認したい。

#### Acceptance Criteria

1. WHEN Okta OIDC認証が使用される THEN システムはHTTPS通信のみを使用しなければならない
2. WHERE OAuth2/OIDCが使用される THE SYSTEM SHALL PKCE (Proof Key for Code Exchange) フローを実装しなければならない
3. WHEN JWTトークンを検証する THEN システムは以下を確認しなければならない：
   - 署名の検証（RS256アルゴリズム使用）
   - issuerの一致確認
   - audienceの一致確認
   - 有効期限（exp）の確認
   - 発行時刻（iat）の妥当性確認
4. WHEN 認証イベントが発生した THEN システムは詳細な監査ログを記録しなければならない（タイムスタンプ、ユーザーID、IPアドレス、成功/失敗、使用されたスコープ）
5. IF 不審なログイン試行が検出された（例：短時間での複数の失敗） THEN システムはアラートを生成しなければならない
6. WHILE トークンを保存している THE SYSTEM SHALL メモリ内のみで管理し、ローカルストレージには暗号化された形式でのみ保管しなければならない
7. WHEN Oktaクライアントシークレットが設定される THEN システムは環境変数または安全な設定管理システムから読み込まなければならない
8. IF トークンのリフレッシュが必要な場合 THEN システムはRefresh Token Rotation を使用して安全にトークンを更新しなければならない

### Requirement 6: モニタリングと運用

**User Story:** 運用チームとして、Okta統合の健全性を監視し、問題を迅速に診断できるようにしたい。

#### Acceptance Criteria

1. WHEN Okta APIが利用不可能な場合 THEN システムは適切なエラーメッセージを表示し、フォールバック動作を実行しなければならない
2. WHERE Spring Actuatorエンドポイントが設定されている THE SYSTEM SHALL Okta接続状態のヘルスチェックを提供しなければならない
3. IF Okta API呼び出しが失敗した THEN システムは指数バックオフを使用してリトライしなければならない
4. WHILE システムが稼働している THE SYSTEM SHALL Okta認証のメトリクス（成功率、レスポンス時間、エラー率）を収集しなければならない
5. WHEN Okta設定が変更された THEN システムは再起動なしで設定をリロードできなければならない
6. IF Oktaレート制限に達した THEN システムはグレースフルに処理し、管理者にアラートを送信しなければならない

### Requirement 7: フロントエンド統合

**User Story:** フロントエンド開発者として、Vue 3アプリケーションでOkta OIDCをシームレスに統合したい。

#### Acceptance Criteria

1. WHEN Vue Routerがナビゲーションガードを実行する THEN システムはOkta認証状態を確認しなければならない
2. IF ユーザーが未認証で保護されたルートにアクセスしようとした THEN システムはOktaログインページへリダイレクトしなければならない
3. WHEN ルートが特定のスコープを要求する AND ユーザーがそのスコープを持たない THEN システムは権限不足エラーページを表示しなければならない
4. WHERE Piniaストアが使用されている THE SYSTEM SHALL 以下を管理しなければならない：
   - ID Token（ユーザー情報）
   - Access Token（API認可用）
   - OAuth2スコープリスト
   - トークン有効期限
5. WHEN トークンがリフレッシュされた THEN システムはAxiosインターセプターで自動的に新しいAccess Tokenを使用しなければならない
6. IF Okta Vue SDKが実装される THEN システムは`@okta/okta-vue`を使用してOIDCフローを処理しなければならない
7. WHILE ユーザーがログインしている THE SYSTEM SHALL ヘッダーにOktaプロファイル情報とアクティブなスコープを表示しなければならない
8. WHEN ログアウトボタンがクリックされた THEN システムはOktaのログアウトエンドポイントを呼び出し、ローカルトークンを削除しなければならない