# 実装タスク

## フェーズ1: バックエンドAPI拡張

- [ ] 1. WorkRecordsResponseレスポンスDTOの作成とテスト
  - WorkRecordsResponseApplicationServiceTestクラスを作成し、レスポンスDTOのテストケースを記述
  - WorkRecordsResponse.javaをcom.devhour.application.dtoパッケージに実装
  - WorkRecordのリストとWorkRecordApprovalを含むDTOとして構成
  - 既存のWorkRecordエンティティとWorkRecordApprovalエンティティを直接使用
  - _要件: REQ-3.1, REQ-3.2_

- [ ] 2. WorkRecordApplicationServiceの拡張とテスト実装
  - WorkRecordApplicationServiceTestに新しいメソッドgetWorkRecordsWithApprovalStatusのテストケースを追加
  - 承認済み、未承認、承認データなしの3パターンのテストケースを作成
  - WorkRecordApplicationServiceにgetWorkRecordsWithApprovalStatusメソッドを実装
  - WorkRecordRepositoryとWorkRecordApprovalRepositoryの両方を使用してデータを取得
  - タスク1で作成したWorkRecordsResponseを使用してレスポンスを構築
  - _要件: REQ-3.3, REQ-3.4_

- [ ] 3. WorkRecordControllerの新規エンドポイント追加とテスト
  - WorkRecordControllerTestに新しいエンドポイントGET /user/{userId}/{date}のテストケースを追加
  - 認証済みユーザーのアクセス制御テストを含める
  - WorkRecordControllerにgetWorkRecordsByDateメソッドを実装
  - タスク2で実装したサービスメソッドを呼び出し
  - @GetMapping("/user/{userId}/{date}")でエンドポイントを公開
  - _要件: REQ-3.5, REQ-6.1_

- [ ] 4. 承認済みデータ更新制限のバックエンドバリデーション実装
  - WorkRecordApplicationServiceTestに承認済みデータ更新拒否のテストケースを追加
  - BusinessExceptionがスローされることを検証
  - WorkRecordApplicationServiceのsaveWorkRecordsメソッドを修正
  - 更新前にWorkRecordApprovalRepositoryで承認ステータスをチェック
  - 承認済み（APPROVED）の場合はBusinessExceptionをスロー
  - _要件: REQ-6.2, REQ-6.3, REQ-6.4_

## フェーズ2: フロントエンドコンポーネント開発

- [ ] 5. TypeScript型定義の追加
  - frontend/src/types/api.tsにWorkRecordApproval型を追加
  - WorkRecordsResponse型を定義（workRecordsとworkRecordApprovalを含む）
  - ApprovalStatus型をenum（'PENDING' | 'APPROVED' | 'REJECTED'）として定義
  - 既存のWorkRecord型定義を確認し、必要に応じて調整
  - _要件: REQ-1.1_

- [ ] 6. developerApiサービスの拡張とテスト
  - frontend/src/services/__tests__/developerApi.test.tsを作成
  - getWorkRecordsByDateメソッドのテストケースを記述
  - frontend/src/services/developerApi.tsにgetWorkRecordsByDateメソッドを追加
  - GET /api/work-records/user/{userId}/{date}を呼び出し
  - タスク5で定義したWorkRecordsResponse型を返す
  - _要件: REQ-3.1, REQ-3.2_

- [ ] 7. useWorkRecordsWithApprovalコンポジット関数の実装とテスト
  - frontend/src/composables/__tests__/useWorkRecordsWithApproval.test.tsを作成
  - 承認済み/未承認/データなしの各状態のテストケースを記述
  - frontend/src/composables/useWorkRecordsWithApproval.tsを実装
  - タスク6のAPIメソッドを使用してデータを取得
  - workRecords、workRecordApproval、isApprovedの計算プロパティを提供
  - 日付変更時の自動リフレッシュ機能を実装
  - _要件: REQ-1.2, REQ-5.1_

- [ ] 8. ApprovalStatusBannerコンポーネントの作成とテスト
  - frontend/src/components/__tests__/ApprovalStatusBanner.test.tsを作成
  - 承認済み状態の表示、承認者名、承認日時の表示テストを記述
  - frontend/src/components/ApprovalStatusBanner.vueを実装
  - Vuetifyのv-alertコンポーネントを使用した警告バナー
  - タスク5で定義したWorkRecordApproval型をpropsとして受け取る
  - 承認日時のフォーマット処理と承認者名の表示を実装
  - _要件: REQ-2.6, REQ-2.7, REQ-2.8_

## フェーズ3: UI統合と入力制限実装

- [ ] 9. WorkHoursTableコンポーネントの修正とテスト - パート1（データ統合）
  - WorkHoursTable.test.tsに承認ステータス連携のテストケースを追加
  - WorkHoursTable.vueでタスク7のuseWorkRecordsWithApprovalを使用
  - 既存のデータ取得ロジックを新しいコンポジット関数に置き換え
  - workRecordApprovalデータをコンポーネント内で利用可能にする
  - _要件: REQ-1.3, REQ-1.4_

- [ ] 10. WorkHoursTableコンポーネントの修正とテスト - パート2（UI制限）
  - 承認済み状態での入力フィールド無効化のテストケースを追加
  - 保存ボタンの無効化テストを追加
  - タスク8のApprovalStatusBannerコンポーネントをテンプレートに追加
  - isApprovedフラグに基づいて全入力フィールドにdisabled属性を設定
  - 保存ボタンのdisabled条件にisApprovedを追加
  - _要件: REQ-1.5, REQ-2.1, REQ-2.2, REQ-2.3_

- [ ] 11. 承認済みスタイルとビジュアルフィードバックの実装
  - WorkHoursTable.vueのスタイルセクションに承認済み用CSSを追加
  - .hours-input--disabledクラスでグレーアウト表示を実装
  - カーソルをnot-allowedに設定し、背景色を#f5f5f5に変更
  - v-text-fieldにclass属性を動的バインディング
  - 保存ボタンのラベルを条件付きで「Read Only」に変更
  - _要件: REQ-2.4, REQ-2.5, REQ-4.1_

- [ ] 12. エラーハンドリングとトースト通知の実装
  - WorkHoursTable.vueにエラーハンドリングロジックを追加
  - 403エラー（承認済み更新試行）を特別に処理
  - useErrorNotificationコンポーザブルを使用してトースト通知を表示
  - 承認済みエラー時に承認ステータスを再取得する処理を追加
  - ネットワークエラーと承認エラーを区別して適切なメッセージを表示
  - _要件: REQ-4.2, REQ-4.3, REQ-4.4, REQ-6.5_

## フェーズ4: 統合テストとE2Eテスト

- [ ] 13. バックエンド統合テストの実装
  - ApprovalRestrictionIntegrationTest.javaを作成
  - 承認済みデータへの更新が403エラーを返すことを検証
  - 未承認データの正常更新が可能であることを確認
  - 承認ステータスAPIが正しいデータを返すことを検証
  - MockMvcを使用した完全なリクエスト/レスポンスフローのテスト
  - _要件: REQ-6.1, REQ-6.2, REQ-6.3, REQ-6.4_

- [ ] 14. フロントエンドE2Eテストの実装
  - frontend/tests/e2e/approvalRestriction.spec.tsを作成
  - Playwrightを使用した承認済み日付の入力制限シナリオテスト
  - 承認済みバナーの表示確認
  - 入力フィールドが無効化されていることの検証
  - 保存ボタンがクリックできないことの確認
  - 未承認日付での正常な入力が可能であることのテスト
  - _要件: REQ-1.5, REQ-2.1, REQ-2.6, REQ-4.1, REQ-5.3_

- [ ] 15. 完全システム統合の検証とリグレッションテスト
  - 全体的な動作フローの確認テストを実装
  - 既存の工数入力機能が影響を受けていないことを確認
  - 承認ワークフローとの連携が正常に動作することを検証
  - パフォーマンステストを実施（API応答時間の測定）
  - 全テストスイートを実行し、カバレッジが80%以上を維持していることを確認
  - _要件: REQ-5.2, REQ-5.4, REQ-5.5_