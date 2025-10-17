# Requirements Document

## Introduction
本機能は、システム管理者が論理削除（無効化）されたユーザーの一覧を取得できるREST APIエンドポイントを提供します。これにより、ユーザー管理における監査性とトレーサビリティが向上し、削除されたユーザーの履歴確認、復元判断、システム利用状況の分析が可能になります。

## Requirements

### Requirement 1: 削除済みユーザー一覧取得API
**User Story:** システム管理者として、削除（無効化）されたユーザーの一覧を確認したい。これにより、過去のユーザー情報の監査や、必要に応じた復元判断ができる。

#### Acceptance Criteria
1. WHEN クライアントが GET /api/users/deleted エンドポイントにリクエストを送信する THEN システムは isActive=false のユーザー一覧を返却する SHALL
2. IF リクエストユーザーがPMOロールを持つ THEN システムは削除済みユーザー一覧へのアクセスを許可する SHALL
3. IF リクエストユーザーがDEVELOPERロールのみを持つ THEN システムは403 Forbiddenエラーを返却する SHALL
4. IF リクエストユーザーが認証されていない THEN システムは401 Unauthorizedエラーを返却する SHALL
5. WHEN 削除済みユーザーが存在しない THEN システムは空の配列を含む200 OKレスポンスを返却する SHALL
6. WHILE システムが削除済みユーザー一覧を取得中 THE SYSTEM SHALL 更新日時（updatedAt）の降順でソートする

### Requirement 2: レスポンスデータ構造
**User Story:** システム管理者として、削除済みユーザーの詳細情報を構造化された形式で取得したい。これにより、ユーザーの削除時期や基本情報を把握できる。

#### Acceptance Criteria
1. WHEN システムが削除済みユーザー一覧を返却する THEN 各ユーザーオブジェクトは以下のフィールドを含む SHALL：
   - id（ユーザーID）
   - username（ユーザー名）
   - email（メールアドレス）
   - role（ロール：PMO/DEVELOPER）
   - fullName（フルネーム）
   - isActive（常にfalse）
   - createdAt（作成日時）
   - updatedAt（削除/無効化日時）
2. IF レスポンスにユーザーが含まれる THEN システムはパスワード関連の情報を除外する SHALL
3. WHEN レスポンスが生成される THEN システムはISO 8601形式で日時フィールドをフォーマットする SHALL
4. WHERE レスポンスのContent-Type THE SYSTEM SHALL application/jsonを設定する

### Requirement 3: ページネーションサポート
**User Story:** システム管理者として、大量の削除済みユーザーを効率的に閲覧したい。これにより、パフォーマンスを維持しながら必要な情報にアクセスできる。

#### Acceptance Criteria
1. WHEN クライアントがpageパラメータを指定する THEN システムは指定されたページ番号のデータを返却する SHALL（デフォルト: 0）
2. WHEN クライアントがsizeパラメータを指定する THEN システムは指定された件数のデータを返却する SHALL（デフォルト: 20、最大: 100）
3. IF sizeパラメータが100を超える THEN システムは400 Bad Requestエラーを返却する SHALL
4. WHEN ページネーションが適用される THEN レスポンスは以下のメタデータを含む SHALL：
   - totalElements（総件数）
   - totalPages（総ページ数）
   - currentPage（現在のページ番号）
   - pageSize（ページサイズ）
   - hasNext（次ページの有無）
   - hasPrevious（前ページの有無）

### Requirement 4: フィルタリング機能
**User Story:** システム管理者として、特定の条件で削除済みユーザーを絞り込みたい。これにより、目的のユーザー情報を素早く見つけることができる。

#### Acceptance Criteria
1. WHEN クライアントがroleパラメータ（PMO/DEVELOPER）を指定する THEN システムは指定されたロールの削除済みユーザーのみを返却する SHALL
2. WHEN クライアントがdeletedFromパラメータ（日付）を指定する THEN システムは指定日以降に削除されたユーザーのみを返却する SHALL
3. WHEN クライアントがdeletedToパラメータ（日付）を指定する THEN システムは指定日以前に削除されたユーザーのみを返却する SHALL
4. IF 無効な日付形式が指定された THEN システムは400 Bad Requestエラーを返却する SHALL
5. WHEN 複数のフィルタパラメータが指定される THEN システムはAND条件として適用する SHALL

### Requirement 5: 検索機能
**User Story:** システム管理者として、削除済みユーザーを名前やメールアドレスで検索したい。これにより、特定のユーザー情報を迅速に確認できる。

#### Acceptance Criteria
1. WHEN クライアントがsearchパラメータを指定する THEN システムは以下のフィールドで部分一致検索を実行する SHALL：
   - username
   - email
   - fullName
2. IF searchパラメータが3文字未満 THEN システムは400 Bad Requestエラーを返却する SHALL
3. WHEN 検索が実行される THEN システムは大文字小文字を区別しない検索を行う SHALL
4. WHERE 検索とフィルタリングの両方が指定される THE SYSTEM SHALL AND条件として組み合わせる

### Requirement 6: エラーハンドリング
**User Story:** システム管理者として、APIエラー時に適切なエラーメッセージを受け取りたい。これにより、問題の原因を理解し適切に対処できる。

#### Acceptance Criteria
1. WHEN データベース接続エラーが発生する THEN システムは503 Service Unavailableエラーを返却する SHALL
2. WHEN 予期しないエラーが発生する THEN システムは500 Internal Server Errorを返却する SHALL
3. IF エラーレスポンスが生成される THEN 以下の構造を含む SHALL：
   - timestamp（エラー発生日時）
   - status（HTTPステータスコード）
   - error（エラータイプ）
   - message（エラーメッセージ）
   - path（リクエストパス）
4. WHERE エラーメッセージ THE SYSTEM SHALL センシティブな情報を含まない

### Requirement 7: 監査ログ
**User Story:** システム管理者として、削除済みユーザー一覧へのアクセスを監査したい。これにより、情報へのアクセス履歴を追跡できる。

#### Acceptance Criteria
1. WHEN 削除済みユーザー一覧APIが呼び出される THEN システムは以下の情報をログに記録する SHALL：
   - アクセス日時
   - アクセスユーザーID
   - リクエストパラメータ
   - レスポンスステータス
   - 返却件数
2. IF アクセスが拒否された THEN システムはセキュリティログにも記録する SHALL
3. WHERE ログレベル THE SYSTEM SHALL INFOレベルで成功アクセスを、WARNレベルで失敗アクセスを記録する