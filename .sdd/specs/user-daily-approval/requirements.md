# Requirements Document

## Introduction
本機能は、工数承認の単位を現在の「申請者/案件/日付」の3次元から「申請者/日付」の2次元に変更することで、承認者の作業効率を大幅に改善し、承認プロセスを簡素化します。承認者は特定の申請者の特定日のすべての案件工数を一括で承認できるようになり、案件ごとの個別承認が不要となります。

## Requirements

### Requirement 1: 承認単位の統合
**User Story:** As a 承認者, I want 申請者の特定日のすべての案件工数を一括で承認する, so that 案件ごとに個別承認する手間を削減できる

#### Acceptance Criteria
1. WHEN 承認者が承認待ち一覧画面を開く THEN システムは申請者/日付ごとに全案件のカテゴリ別工数合計を表示すること
2. WHERE 承認待ち一覧画面において THE SYSTEM SHALL その日の全案件の工数をカテゴリ別に集計した合計値を表形式で表示すること
3. WHEN 承認者が一覧画面上で承認ボタンをクリックする THEN システムは同一申請者の同一日付のすべての案件工数を一括で承認しWorkRecordApprovalの承認ステータスをAPPROVEDに更新すること
4. IF 申請者の特定日に複数の案件の工数が存在する THEN システムはそれらをすべて同じ承認ステータスで管理すること
5. WHEN 承認者が一覧画面上で差し戻しボタンをクリックする THEN システムは却下理由の入力を求めWorkRecordApprovalの承認ステータスをREJECTEDに更新し却下理由を保存すること

### Requirement 2: 承認対象の集約表示
**User Story:** As a 承認者, I want 申請者の日次工数を案件横断で確認する, so that 総合的な判断で承認可否を決定できる

#### Acceptance Criteria
1. WHEN 承認者が承認待ち一覧を開く THEN システムは申請者と日付の組み合わせでグループ化された一覧を表示すること
2. WHERE 一覧画面の各行において THE SYSTEM SHALL その日の全案件を通したカテゴリ別工数の合計を表示すること
3. WHEN 承認者が一覧画面で申請者/日付の行を展開する THEN システムは案件ごとのカテゴリ別工数内訳を同一画面内に表示すること
4. IF 複数の案件で同じカテゴリの工数が存在する THEN システムはそれらを合算してカテゴリ別合計として表示すること
5. WHERE 一覧画面において THE SYSTEM SHALL 全カテゴリの日次合計工数を各行に明確に表示すること

### Requirement 3: データモデルの最適化
**User Story:** As a システム管理者, I want 承認ステータスを申請者/日付単位で管理する, so that 新しい承認単位に対応できる

#### Acceptance Criteria
1. WHERE データベースにおいて THE SYSTEM SHALL WorkRecordApprovalテーブルを作成し申請者ID、日付、承認ステータス、承認者ID、承認日時、却下理由を管理すること
2. WHEN 新しい工数が登録される THEN システムはWorkRecordApprovalテーブルに申請者/日付の組み合わせで承認レコードを作成し承認ステータスをPENDINGに設定すること
3. IF 同一申請者/同一日付の工数が既に存在する THEN システムはWorkRecordApprovalの既存ステータスを維持すること
4. WHEN 承認処理が実行される THEN システムはWorkRecordApprovalテーブルの該当レコードの承認ステータスをAPPROVEDに更新し承認者IDと承認日時を記録すること
5. WHEN 差し戻し処理が実行される THEN システムはWorkRecordApprovalテーブルの該当レコードの承認ステータスをREJECTEDに更新し承認者IDと承認日時と却下理由を記録すること
6. IF WorkRecordApprovalで申請者/日付が承認済み（APPROVED）ステータスである THEN システムはその日付への新規工数入力を拒否すること
7. IF WorkRecordApprovalで申請者/日付が承認待ち（PENDING）または差し戻し（REJECTED）ステータスである THEN システムはその日付への工数入力を許可すること
8. WHEN システムがマイグレーションを実行する THEN 既存のWorkRecordテーブルの承認情報をWorkRecordApprovalテーブルに移行すること

### Requirement 4: 承認権限の検証
**User Story:** As a システム, I want 申請者/日付単位で承認権限を検証する, so that 適切な承認者のみが承認できる

#### Acceptance Criteria
1. WHEN 承認APIが呼び出される THEN システムは承認者と申請者の上下関係を検証すること
2. IF 承認者が申請者の上司でない THEN システムは承認要求を拒否すること
3. WHEN 承認権限チェックが実行される THEN システムは申請者/日付の組み合わせで権限を検証すること
4. IF 不正な承認要求が検出される THEN システムはHTTP 403エラーを返却すること
5. WHERE 権限検証において THE SYSTEM SHALL SupervisorRelationshipテーブルを参照すること

### Requirement 5: 承認画面のUI最適化
**User Story:** As a 承認者, I want 申請者/日付単位で効率的に承認作業を行う, so that 承認作業の時間を短縮できる

#### Acceptance Criteria
1. WHEN 承認者が承認画面にアクセスする THEN システムは申請者/日付単位でグループ化された承認待ち一覧を表示すること
2. WHERE 一覧画面において THE SYSTEM SHALL 各行に承認/却下ボタンを配置し詳細画面への遷移なしに承認処理を可能にすること
3. IF 承認者が複数の項目を選択する THEN システムは一括承認ボタンを有効化すること
4. WHEN 承認/却下が一覧画面から実行される THEN システムはインラインでコメント入力フィールドを表示すること
5. WHERE 承認画面において THE SYSTEM SHALL 承認済み/未承認のステータスを視覚的に区別すること

### Requirement 6: 申請者向け承認状況表示
**User Story:** As a 申請者, I want 自分の工数の承認状態を日単位で確認する, so that 承認状況を把握できる

#### Acceptance Criteria
1. WHEN 申請者がカレンダービューを開く THEN システムはWorkRecordApprovalテーブルから日次の承認状態を取得し色分けして表示すること
2. WHERE カレンダービューにおいて THE SYSTEM SHALL 承認済み（緑）、承認待ち（黄）、差し戻し（赤）で色分けすること
3. WHEN 申請者が承認済み（APPROVED）の日付の工数入力画面にアクセスする THEN システムは編集不可メッセージを表示し入力を無効化すること
4. IF WorkRecordApprovalで差し戻し（REJECTED）ステータスである THEN システムは却下理由を画面に表示し工数の再入力を許可すること
5. WHERE 承認状況画面において THE SYSTEM SHALL 差し戻しされた日付を赤色でハイライト表示し却下理由をツールチップで表示すること

### Requirement 7: 承認API の実装
**User Story:** As a フロントエンド開発者, I want 申請者/日付単位で承認操作を行うAPIを利用する, so that 新しい承認フローに対応できる

#### Acceptance Criteria
1. WHEN POST /api/approvals/daily が呼び出される THEN システムはWorkRecordApprovalテーブルを更新し申請者/日付単位で承認処理を実行すること
2. IF WorkRecordApprovalで承認済み（APPROVED）ステータスの日付に対して工数登録APIが呼び出される THEN システムはHTTP 409エラーを返却すること
3. IF WorkRecordApprovalで承認待ち（PENDING）または差し戻し（REJECTED）ステータスの日付に対して工数登録APIが呼び出される THEN システムは工数入力を許可し正常に処理すること
4. WHEN GET /api/approvals/pending が改修される THEN システムはWorkRecordApprovalテーブルから承認待ちステータスのレコードを取得し申請者/日付でグループ化して返却すること
5. WHERE APIレスポンスにおいて THE SYSTEM SHALL WorkRecordApprovalの承認情報と関連するWorkRecordのカテゴリ別工数合計および案件ごとの内訳を含めること
6. WHEN 一括承認APIが呼び出される THEN システムはWorkRecordApprovalテーブルの複数レコードを同時に更新し承認者IDと承認日時を記録すること

### Requirement 8: データ移行戦略
**User Story:** As a システム管理者, I want 既存の承認データを新形式に移行する, so that システムの継続性を保てる

#### Acceptance Criteria
1. WHEN 移行スクリプトが実行される THEN システムは既存のWorkRecordテーブルの承認情報からWorkRecordApprovalテーブルにデータを移行すること
2. WHERE 移行処理において THE SYSTEM SHALL 申請者ID/日付でグループ化し最初の承認者IDと承認日時を記録すること
3. IF 同一申請者/同一日付で複数の承認ステータスが存在する THEN システムは最も制限的なステータス（未承認＞却下＞承認の優先順位）を適用すること
4. WHILE 移行処理が実行中 THE SYSTEM SHALL WorkRecordApprovalテーブルへの挿入件数と既存データとの整合性チェック結果をログに記録すること
5. IF 移行エラーが発生する THEN システムはWorkRecordApprovalテーブルの変更をロールバックし元の状態を維持すること

### Requirement 9: 承認状況レポート
**User Story:** As a マネージャー, I want 申請者/日付単位での承認状況を確認する, so that チーム全体の工数入力・承認状況を把握できる

#### Acceptance Criteria
1. WHEN マネージャーがレポート画面を開く THEN システムは申請者別の承認状況サマリーを表示すること
2. WHERE レポート画面において THE SYSTEM SHALL 未承認日数と承認済み日数を集計して表示すること
3. IF 承認が遅延している項目が存在する THEN システムはアラートアイコンを表示すること
4. WHILE レポートが表示されている間 THE SYSTEM SHALL 30秒ごとに最新データを自動更新すること

