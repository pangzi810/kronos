# Requirements Document

## Introduction

本機能は、SailPoint Identity Security Cloud (IdentityNow)を人事情報のマスターデータソースとして活用し、組織の社員情報（部署、役職、マネージャー、従業員番号等）を取得して専用の人事マスタテーブル（hr_master）として保持する統合機能です。SailPointは人事システムと連携した正確な組織情報を保持しているため、これをソースとすることで、システム内で最新の人事データを参照可能にします。人事マスタテーブルは認証用のユーザーテーブルとは独立して管理され、メールアドレスを通じて関連付けられます。認証はOkta SSOが担当し、SailPointは純粋に人事情報の取得源として利用します。

## Requirements

### Requirement 1: SailPoint API接続と認証
**User Story:** システム管理者として、SailPoint Identity Security CloudのAPIに安全に接続し、人事情報の読み取り権限で認証を行いたい。これにより、組織の人事データにアクセスできる。

#### Acceptance Criteria

1. WHEN システムがSailPoint APIへの接続を開始する THEN システムはOAuth 2.0 Client Credentialsフローを使用して認証を実行する
2. IF OAuth認証に必要な資格情報（Client ID、Client Secret、Token URL）が設定されている THEN システムはアクセストークンを取得する
3. WHEN アクセストークンの有効期限が切れそうになる THEN システムは自動的にトークンをリフレッシュする
4. IF SailPoint APIへの接続が失敗した THEN システムはエラーをログに記録し、管理者に通知する
5. WHERE 環境変数にSailPoint接続情報が設定されている場合 システムはそれらを安全に読み込んで使用する
6. WHEN API認証情報が無効または期限切れの場合 THEN システムは適切なエラーメッセージを返し、処理を中断する

### Requirement 2: 人事情報の取得
**User Story:** システム管理者として、SailPointから全社員の人事情報（組織情報、役職、マネージャー関係等）を取得したい。これにより、最新の人事データをシステムに反映できる。

#### Acceptance Criteria

1. WHEN 同期処理が実行される THEN システムはSailPoint Identity API（/v3/identities）を呼び出してアイデンティティ一覧を取得する
2. WHEN SailPointがページネーションで結果を返す THEN システムは全ページを順次取得して完全なアイデンティティ一覧を構築する
3. IF APIレート制限（100リクエスト/秒）に達した THEN システムは指数バックオフでリトライする
4. WHILE アイデンティティ情報を取得中 システムは進捗状況をログに記録する
5. WHEN 取得する人事情報には以下のフィールドが含まれる THEN システムはこれらを人事データとして処理する：
   - email (メールアドレス) - ユーザー識別用
   - firstName (名)
   - lastName (姓)
   - displayName (表示名)
   - attributes.employeeNumber (従業員番号)
   - attributes.title (役職)
   - attributes.location (勤務地)
   - attributes.costCenter (コストセンター)

   【主務情報】
   - attributes.primaryDivision (主務事業部)
   - attributes.primaryDepartment (主務部署)
   - attributes.primaryGroup (主務グループ)
   - attributes.primaryRole (主務役割/職責)

   【兼務1情報】
   - attributes.secondary1Division (兼務1事業部)
   - attributes.secondary1Department (兼務1部署)
   - attributes.secondary1Group (兼務1グループ)
   - attributes.secondary1Role (兼務1役割/職責)

   【兼務2情報】
   - attributes.secondary2Division (兼務2事業部)
   - attributes.secondary2Department (兼務2部署)
   - attributes.secondary2Group (兼務2グループ)
   - attributes.secondary2Role (兼務2役割/職責)

   【兼務3情報】
   - attributes.secondary3Division (兼務3事業部)
   - attributes.secondary3Department (兼務3部署)
   - attributes.secondary3Group (兼務3グループ)
   - attributes.secondary3Role (兼務3役割/職責)

   【兼務4情報】
   - attributes.secondary4Division (兼務4事業部)
   - attributes.secondary4Department (兼務4部署)
   - attributes.secondary4Group (兼務4グループ)
   - attributes.secondary4Role (兼務4役割/職責)

   【兼務5情報】
   - attributes.secondary5Division (兼務5事業部)
   - attributes.secondary5Department (兼務5部署)
   - attributes.secondary5Group (兼務5グループ)
   - attributes.secondary5Role (兼務5役割/職責)

   - managerRef (マネージャー参照)
   - identityStatus (在籍状態)
6. WHERE アイデンティティのprocessingStateがactiveまたはdisabledの場合 システムは同期対象として処理する

### Requirement 3: 人事マスタテーブルへのデータ同期
**User Story:** 開発者として、SailPointから取得した人事情報を専用の人事マスタテーブル（hr_master）に同期したい。これにより、認証情報とは独立して最新の組織構造と人事データを維持できる。

#### Acceptance Criteria

1. WHEN 人事マスタテーブル（hr_master）が存在しない THEN システムは以下の構造で新規テーブルを作成する
2. WHEN SailPointの人事データに存在する社員がhr_masterテーブルに存在しない THEN システムは新規レコードを作成する
3. WHEN SailPointの人事情報とhr_masterテーブルのデータが異なる THEN システムは該当レコードを更新する
4. IF hr_masterテーブルの社員がSailPointに存在しない THEN システムはis_activeフラグをfalseに設定し、レコードは物理削除しない
5. WHEN 人事情報を同期する THEN システムは以下のフィールドをhr_masterテーブルにマッピングする：
   【基本情報】
   - employee_number ← attributes.employeeNumber
   - title ← attributes.title
   - location ← attributes.location
   - cost_center ← attributes.costCenter

   【主務配属】
   - primary_division ← attributes.primaryDivision
   - primary_department ← attributes.primaryDepartment
   - primary_group ← attributes.primaryGroup
   - primary_role ← attributes.primaryRole

   【兼務1配属】
   - secondary1_division ← attributes.secondary1Division
   - secondary1_department ← attributes.secondary1Department
   - secondary1_group ← attributes.secondary1Group
   - secondary1_role ← attributes.secondary1Role

   【兼務2配属】
   - secondary2_division ← attributes.secondary2Division
   - secondary2_department ← attributes.secondary2Department
   - secondary2_group ← attributes.secondary2Group
   - secondary2_role ← attributes.secondary2Role

   【兼務3配属】
   - secondary3_division ← attributes.secondary3Division
   - secondary3_department ← attributes.secondary3Department
   - secondary3_group ← attributes.secondary3Group
   - secondary3_role ← attributes.secondary3Role

   【兼務4配属】
   - secondary4_division ← attributes.secondary4Division
   - secondary4_department ← attributes.secondary4Department
   - secondary4_group ← attributes.secondary4Group
   - secondary4_role ← attributes.secondary4Role

   【兼務5配属】
   - secondary5_division ← attributes.secondary5Division
   - secondary5_department ← attributes.secondary5Department
   - secondary5_group ← attributes.secondary5Group
   - secondary5_role ← attributes.secondary5Role

   【管理情報】
   - manager_id ← managerRef.id (存在する場合)
   - manager_email ← managerRef.email (存在する場合)
   - hr_status ← identityStatus (人事ステータス)
   - is_active ← true/false (SailPointでの存在有無)
   - hr_data_updated_at ← 同期実行時刻

   【識別情報】
   - email ← email (主キー、ユーザーテーブルとの関連用)
   - first_name ← firstName
   - last_name ← lastName
   - display_name ← displayName
   - sailpoint_id ← id (SailPoint内部ID)
6. WHERE emailアドレスを主キーとしてhr_masterテーブルを管理し THEN システムはユーザーテーブルとメールアドレスで関連付ける
7. WHEN データベーストランザクション中にエラーが発生した THEN システムはロールバックしてデータ整合性を保つ

### Requirement 4: 自動同期スケジューリング
**User Story:** システム管理者として、SailPoint同期を毎時自動実行し、その状況を監視したい。これにより、人事データをほぼリアルタイムで最新状態に保てる。

#### Acceptance Criteria

1. WHEN スケジュール同期が設定されている THEN システムは毎時（毎時0分）に自動実行する
2. IF 同期処理が既に実行中 THEN システムは新しいスケジュール実行をスキップし、次回実行を待つ
3. WHILE 同期処理が実行中 システムは以下の情報を記録する：
   - 処理開始時刻
   - 取得アイデンティティ数
   - 新規作成レコード数（hr_master）
   - 更新レコード数（hr_master）
   - 非アクティブ化数（SailPointに存在しない社員）
   - エラー発生数
4. WHEN 同期処理が完了した THEN システムは処理サマリーをログに記録し、異常があれば管理者に通知する
5. WHERE 分散環境で複数のインスタンスが存在する場合 システムはShedLockを使用して重複実行を防ぐ
6. IF 同期処理がスケジュール時刻から一定時間（デフォルト：1時間）経過しても開始されない THEN システムはアラートを発生させる

### Requirement 5: 認証システムとの役割分担とテーブル分離
**User Story:** 開発者として、人事マスタ（hr_master）とユーザー認証テーブル（users）を完全に分離し、それぞれの役割を適切に管理したい。これにより、責務の分離と安定した運用を実現できる。

#### Acceptance Criteria

1. WHEN SailPointから人事情報を同期する THEN システムはhr_masterテーブルのみを更新し、usersテーブルには一切触れない
2. IF Oktaでユーザーが新規作成された AND hr_masterに該当社員が存在する THEN システムはemailで関連付けて人事情報を参照可能にする
3. WHERE ユーザーがOktaで認証される場合 システムはhr_masterテーブルの存在に関わらず正常に動作する
4. WHEN 人事情報とOkta認証情報を参照する場合 THEN システムはJOINまたは別クエリでemailを使って関連付ける
5. IF SailPoint同期が失敗または遅延した場合でも THEN Okta認証とusersテーブルは影響を受けない
6. WHEN hr_masterテーブルとusersテーブルのデータ不整合がある場合 THEN システムは各テーブルを独立したエンティティとして扱う

### Requirement 6: エラーハンドリングと監視
**User Story:** システム管理者として、SailPoint同期のエラーを検知し、問題を迅速に解決したい。これにより、システムの信頼性を維持できる。

#### Acceptance Criteria

1. WHEN SailPoint APIがエラーレスポンスを返す THEN システムはHTTPステータスコードとエラー詳細を記録する
2. IF ネットワークエラーが発生した THEN システムは最大3回リトライし、それでも失敗した場合はアラートを発生させる
3. WHERE 部分的な同期失敗が発生した場合 システムは成功した部分を保持し、失敗したアイデンティティのリストを記録する
4. WHEN APIレスポンスが予期しない形式の場合 THEN システムは詳細なエラーログを記録し、処理をスキップする
5. IF 同期処理の実行時間が設定された閾値（デフォルト30分）を超えた THEN システムはタイムアウトして処理を中断する
6. WHILE エラーリカバリ処理中 システムは詳細な診断情報をログに出力する

### Requirement 7: セキュリティとコンプライアンス
**User Story:** セキュリティ管理者として、SailPoint統合が組織のセキュリティポリシーに準拠し、機密情報を適切に保護したい。これにより、コンプライアンス要件を満たせる。

#### Acceptance Criteria

1. WHEN SailPoint API認証情報を保存する THEN システムは環境変数または暗号化されたシークレット管理システムを使用する
2. WHILE SailPoint APIと通信中 システムは全ての通信をHTTPS/TLS 1.2以上で暗号化する
3. WHERE センシティブ情報（パスワード、トークン）がログ出力される可能性がある場合 システムはそれらをマスクまたは除外する
4. WHEN 同期APIエンドポイントにアクセスする THEN システムは管理者権限を持つユーザーのみを許可する
5. IF 監査要件が設定されている THEN システムは全ての同期操作を監査ログに記録する
6. WHEN 個人情報を含むデータを処理する THEN システムはGDPR/個人情報保護法に準拠した方法で処理する

### Requirement 8: 人事マスタテーブルの構造定義
**User Story:** データベース管理者として、人事情報を格納する専用のhr_masterテーブルを適切に設計し、将来の拡張にも対応できる構造を定義したい。これにより、人事データの独立性と拡張性を確保できる。

#### Acceptance Criteria

1. WHEN hr_masterテーブルを作成する THEN システムは以下の構造を定義する：
```sql
CREATE TABLE hr_master (
    -- 識別情報
    email VARCHAR(255) PRIMARY KEY,
    sailpoint_id VARCHAR(100) UNIQUE NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    display_name VARCHAR(255),

    -- 基本人事情報
    employee_number VARCHAR(50),
    title VARCHAR(255),
    location VARCHAR(255),
    cost_center VARCHAR(100),

    -- 主務配属
    primary_division VARCHAR(255),
    primary_department VARCHAR(255),
    primary_group VARCHAR(255),
    primary_role VARCHAR(255),

    -- 兼務1
    secondary1_division VARCHAR(255),
    secondary1_department VARCHAR(255),
    secondary1_group VARCHAR(255),
    secondary1_role VARCHAR(255),

    -- 兼務2
    secondary2_division VARCHAR(255),
    secondary2_department VARCHAR(255),
    secondary2_group VARCHAR(255),
    secondary2_role VARCHAR(255),

    -- 兼務3
    secondary3_division VARCHAR(255),
    secondary3_department VARCHAR(255),
    secondary3_group VARCHAR(255),
    secondary3_role VARCHAR(255),

    -- 兼務4
    secondary4_division VARCHAR(255),
    secondary4_department VARCHAR(255),
    secondary4_group VARCHAR(255),
    secondary4_role VARCHAR(255),

    -- 兼務5
    secondary5_division VARCHAR(255),
    secondary5_department VARCHAR(255),
    secondary5_group VARCHAR(255),
    secondary5_role VARCHAR(255),

    -- 管理情報
    manager_id VARCHAR(100),
    manager_email VARCHAR(255),
    hr_status VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,

    -- メタデータ
    hr_data_updated_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    -- インデックス
    INDEX idx_employee_number (employee_number),
    INDEX idx_manager_email (manager_email),
    INDEX idx_is_active (is_active),
    INDEX idx_hr_data_updated_at (hr_data_updated_at)
)
```
2. WHERE emailフィールドが主キーとして設定され THEN システムはusersテーブルとの結合に使用する
3. IF 将来的にカスタムフィールドが必要になった場合 THEN JSON型のcustom_attributesフィールドを追加可能にする
4. WHEN パフォーマンス要件を満たすため THEN システムは適切なインデックスを配置する
5. WHERE データ整合性を保つため THEN システムは外部キー制約を使用せず、アプリケーションレベルで整合性を管理する

### Requirement 9: Webhookエンドポイントによる人事変更の即時反映
**User Story:** システム管理者として、SailPointから人事変更イベント（組織変更、昇進、異動等）をWebhook経由で受信し、リアルタイムで人事情報を更新したい。これにより、組織変更を即座にシステムに反映できる。

#### Acceptance Criteria

1. WHEN システムがWebhookエンドポイントを公開する THEN `/api/webhooks/sailpoint/hr-events`のパスで人事イベントを受信可能にする
2. IF SailPointからWebhookリクエストが送信される THEN システムはHTTP POST メソッドでJSONペイロードを受信する
3. WHEN Webhookリクエストを受信する THEN システムは以下の人事関連イベントのみを処理する：
   - ATTRIBUTE_CHANGED - 部署、役職、勤務地等の変更
   - MANAGER_CHANGED - レポートライン（上司）の変更
   - IDENTITY_UPDATED - 従業員情報の更新
   - IDENTITY_LIFECYCLE_STATE_CHANGED - 在籍ステータスの変更（入社、退職、休職等）
4. WHERE Webhookリクエストの署名検証が必要な場合 システムはHMACベースの署名を検証してリクエストの真正性を確認する
5. WHEN 有効な人事変更イベントを受信した THEN システムは以下の処理を実行する：
   - イベントタイプの識別（人事関連のみ処理）
   - ペイロードから人事情報の抽出
   - hr_masterテーブルの該当レコードを更新（emailで識別）
   - 処理結果のログ記録
6. IF Webhookペイロードが不正または署名が無効 THEN システムは400 Bad Requestまたは401 Unauthorizedを返す
7. WHEN Webhook処理が成功した THEN システムは200 OKまたは202 Acceptedステータスコードを返す
8. IF Webhook処理中にエラーが発生した THEN システムは適切なエラーコード（500 Internal Server Error等）を返し、リトライ可能であることを示す
9. WHERE 同一イベントが重複して送信される場合 システムは冪等性を保証して重複処理を防ぐ
10. WHEN Webhookエンドポイントがダウンしている間のイベント THEN SailPointの再送機能と連携して欠落したイベントを補完する
11. IF リアルタイム同期と定期バッチ同期が競合する場合 THEN システムは楽観的ロックで競合を回避する
12. WHILE Webhookイベントを処理中 システムは処理時間を記録し、パフォーマンスメトリクスを収集する

---