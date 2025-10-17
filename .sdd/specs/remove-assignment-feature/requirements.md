# 要件定義書

## はじめに
本仕様は、開発工数管理システムからプロジェクトアサイン機能（ProjectAssignment）を完全に廃止し、システムアーキテクチャを簡素化することを目的とします。ProjectAssignmentエンティティおよび関連するすべてのロジック、リポジトリ、サービス、バリデーター、マッパー、コントローラー、UIコンポーネントを削除します。これにより、ユーザーとプロジェクトの関係性管理が不要となり、全てのユーザーが全てのプロジェクトに対して工数を記録できるようになります。システムの複雑性を大幅に削減し、メンテナンス性を向上させます。

## 要件

### 要件1: アサイン機能の削除
**ユーザーストーリー:** システム管理者として、複雑なアサイン管理機能を削除し、システムをシンプルに保ちたい

#### 受け入れ条件
1. WHEN ユーザーがプロジェクト一覧を表示する THEN システムは全ての有効なプロジェクトを表示しなければならない
2. WHEN ユーザーが工数入力画面にアクセスする THEN システムはアサインの有無に関係なく全てのプロジェクトを選択可能にしなければならない
3. IF プロジェクトアサイン関連のAPIが呼び出された THEN システムは適切なエラーメッセージを返さなければならない
4. WHEN 管理者がユーザー管理画面にアクセスする THEN システムはプロジェクトアサイン関連の機能を表示してはならない

### 要件2: データベースの簡素化
**ユーザーストーリー:** データベース管理者として、不要なテーブルと制約を削除し、スキーマをクリーンに保ちたい

#### 受け入れ条件
1. WHEN データベースマイグレーションが実行される THEN システムはproject_assignmentsテーブルを安全に削除しなければならない
2. IF 既存のアサインデータが存在する THEN システムは履歴保存のためのバックアップテーブルを作成しなければならない
3. WHEN 関連する外部キー制約が存在する THEN システムは依存関係を適切に処理しなければならない
4. WHERE インデックスがproject_assignmentsに関連している THEN システムは不要なインデックスを削除しなければならない

### 要件3: バックエンドAPIの更新
**ユーザーストーリー:** API開発者として、ProjectAssignment関連のすべてのコードを削除し、APIをシンプルに保ちたい

#### 受け入れ条件
1. WHEN クライアントがプロジェクトアサイン関連のAPIエンドポイントにアクセスする THEN システムは404エラーを返さなければならない
2. IF ProjectAssignmentエンティティが参照されている THEN システムはコンパイルエラーが発生しないように完全に削除しなければならない
3. WHEN ProjectAssignmentValidatorが呼び出される場面がある THEN システムは該当するバリデーションロジックをすべて削除しなければならない
4. WHILE アサイン関連のサービスクラスが存在する THE SYSTEM SHALL 関連するすべてのクラスとインターフェースを削除しなければならない
5. WHERE ProjectAssignmentRepositoryが使用されている THE SYSTEM SHALL リポジトリインターフェースと実装クラスを完全に削除しなければならない
6. IF ProjectAssignmentMapperが存在する THEN システムはMyBatisマッパーインターフェースを削除しなければならない
7. WHEN ProjectAllocationServiceがProjectAssignmentを参照している THEN システムは該当するメソッドとロジックを削除しなければならない

### 要件4: フロントエンドUIの簡素化
**ユーザーストーリー:** エンドユーザーとして、シンプルで直感的なUIで工数を入力したい

#### 受け入れ条件
1. WHEN ユーザーがAssignmentViewにアクセスしようとする THEN システムは適切なリダイレクトまたはエラーページを表示しなければならない
2. IF プロジェクトアサイン関連のコンポーネントが存在する THEN システムは該当するVueコンポーネントを削除しなければならない
3. WHEN useProjectAssignmentコンポーザブルが使用されている THEN システムは代替のロジックを提供するか削除しなければならない
4. WHERE メニューやナビゲーションにアサイン関連のリンクがある THE SYSTEM SHALL 該当する項目を削除しなければならない

### 要件5: 工数入力プロセスの変更
**ユーザーストーリー:** 開発者として、アサインに関係なく任意のプロジェクトに工数を記録したい

#### 受け入れ条件
1. WHEN ユーザーが工数入力画面を開く THEN システムは全ての有効なプロジェクトをドロップダウンに表示しなければならない
2. IF ユーザーが任意のプロジェクトを選択する THEN システムはアサインチェックを行わずに工数を記録できなければならない
3. WHEN 工数データが保存される THEN システムはproject_assignmentsテーブルを参照せずに処理を完了しなければならない
4. WHILE ユーザーが工数を入力している THE SYSTEM SHALL プロジェクトの選択制限を適用してはならない

### 要件6: 工数入力権限の簡素化
**ユーザーストーリー:** ユーザーとして、有効なプロジェクトであれば制限なく工数を入力したい

#### 受け入れ条件
1. WHEN ユーザーが工数入力画面にアクセスする THEN システムはロールに関係なくすべてのユーザーに工数入力を許可しなければならない
2. IF プロジェクトが有効（ACTIVE）状態である THEN システムはすべてのユーザーに該当プロジェクトへの工数入力を許可しなければならない
3. WHERE プロジェクトアサインチェックが存在する THE SYSTEM SHALL 該当するチェックロジックを完全に削除しなければならない
4. WHEN 工数入力の権限チェックが実行される THEN システムはユーザーの認証状態のみを確認し、ロールチェックを行ってはならない

### 要件7: テストコードの更新
**ユーザーストーリー:** QAエンジニアとして、アサイン機能削除後もテストカバレッジを維持したい

#### 受け入れ条件
1. WHEN ProjectAssignment関連のテストが実行される THEN システムは該当するテストケースを削除しなければならない
2. IF 統合テストでアサイン機能に依存している箇所がある THEN システムは代替のテストシナリオを実装しなければならない
3. WHILE テストスイートが実行される THE SYSTEM SHALL アサイン関連のモックやスタブを使用してはならない
4. WHERE E2Eテストでアサイン操作が含まれている THE SYSTEM SHALL 該当するテストステップを削除または更新しなければならない

### 要件8: ドキュメントとコメントの更新
**ユーザーストーリー:** 開発者として、正確で最新のドキュメントを参照したい

#### 受け入れ条件
1. WHEN README.mdやCLAUDE.mdが参照される THEN システムはアサイン機能に関する記述を削除しなければならない
2. IF APIドキュメント（Swagger）にアサイン関連のエンドポイントがある THEN システムは該当する定義を削除しなければならない
3. WHERE コード内のコメントやJavaDocでアサインに言及している THE SYSTEM SHALL 該当するコメントを削除または更新しなければならない
4. WHEN ユーザーマニュアルが更新される THEN システムはアサイン機能に関するセクションを削除しなければならない

### 要件9: 後方互換性の考慮
**ユーザーストーリー:** システム運用者として、既存データの整合性を保ちながら移行したい

#### 受け入れ条件
1. WHEN 既存の工数データが存在する THEN システムはアサイン情報なしでもデータを正しく表示しなければならない
2. IF 履歴データの参照が必要な場合 THEN システムはバックアップテーブルから過去のアサイン情報を取得できなければならない
3. WHILE データ移行が実行される THE SYSTEM SHALL トランザクションを使用してデータの一貫性を保証しなければならない
4. WHERE レポート機能がアサイン情報を使用している THE SYSTEM SHALL アサインに依存しない形でレポートを生成しなければならない

### 要件10: ProjectAssignment関連ファイルの完全削除
**ユーザーストーリー:** 開発者として、ProjectAssignment関連のすべてのファイルとロジックを削除し、クリーンなコードベースを維持したい

#### 受け入れ条件
1. WHEN backend/src/main/java/com/devhour/domain/model/entity/ProjectAssignment.javaが存在する THEN システムは該当ファイルを完全に削除しなければならない
2. IF backend/src/main/java/com/devhour/domain/repository/ProjectAssignmentRepository.javaが存在する THEN システムは該当インターフェースを削除しなければならない
3. WHERE backend/src/main/java/com/devhour/infrastructure/repository/ProjectAssignmentRepositoryImpl.javaが存在する THE SYSTEM SHALL 実装クラスを削除しなければならない
4. WHEN backend/src/main/java/com/devhour/infrastructure/mapper/ProjectAssignmentMapper.javaが存在する THEN システムはMyBatisマッパーを削除しなければならない
5. IF backend/src/main/java/com/devhour/domain/service/ProjectAssignmentValidator.javaが存在する THEN システムはバリデータークラスを削除しなければならない
6. WHERE frontend/src/components/assignment/フォルダが存在する THE SYSTEM SHALL すべてのアサイン関連コンポーネントを削除しなければならない
7. WHEN frontend/src/views/AssignmentView.vueが存在する THEN システムは該当ビューコンポーネントを削除しなければならない
8. IF frontend/src/composables/useProjectAssignment.tsが存在する THEN システムはコンポーザブルを削除しなければならない
9. WHERE frontend/src/services/domains/project-assignment.service.tsが存在する THE SYSTEM SHALL サービスファイルを削除しなければならない