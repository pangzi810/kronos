# Implementation Plan

## Phase 1: Backend API実装

- [ ] 1. データベースインデックスとマイグレーション作成
  - Flyway Migration V21を作成してインデックス追加
  - `idx_work_records_user_date_range`インデックスをwork_recordsテーブルに追加
  - 日本の祝日マスタテーブル（japanese_holidays）の作成（オプション）
  - テストデータベースでマイグレーション動作確認
  - _Requirements: REQ-2_

- [ ] 2. ドメインサービス実装（TDD）
- [ ] 2.1 WorkRecordStatusServiceのテスト作成
  - `calculateStatus`メソッドのユニットテスト作成
  - `findMissingDates`メソッドのテストケース作成
  - 境界値テスト：月初、月末、週末、祝日のケース
  - _Requirements: REQ-2, REQ-3_

- [ ] 2.2 WorkRecordStatusService実装
  - WorkRecordとWorkRecordApprovalからステータス計算ロジック実装
  - 指定期間内の未入力日を特定するロジック実装
  - 週末・祝日判定ロジックの組み込み
  - _Requirements: REQ-2, REQ-3_

- [ ] 3. アプリケーションサービス拡張（TDD）
- [ ] 3.1 WorkRecordApplicationServiceのテスト作成
  - `getMissingDatesForMonth`メソッドの統合テスト作成
  - モックを使用したリポジトリとの連携テスト
  - エラーケースのテスト：不正な日付範囲、権限なし
  - _Requirements: REQ-2_

- [ ] 3.2 WorkRecordApplicationService実装
  - `getMissingDatesForMonth`メソッド実装
  - WorkRecordRepositoryとの連携処理
  - 日付範囲バリデーション（最大3ヶ月制限）実装
  - _Requirements: REQ-2_

- [ ] 4. REST APIエンドポイント実装（TDD）
- [ ] 4.1 WorkRecordControllerのテスト作成
  - GET `/api/work-records/missing-dates`エンドポイントのテスト
  - 認証テスト、権限テスト、パラメータバリデーションテスト
  - レスポンス形式の検証テスト
  - _Requirements: REQ-2, REQ-4_

- [ ] 4.2 WorkRecordController拡張
  - `getMissingDates`エンドポイントメソッド実装
  - RequestパラメータとResponseDTO作成
  - エラーハンドリングとHTTPステータスコード設定
  - _Requirements: REQ-2, REQ-4_

## Phase 2: Frontend基本実装

- [ ] 5. Vuexストア拡張とAPIクライアント
- [ ] 5.1 workRecordストアのテスト作成
  - `fetchMissingDates`アクションのユニットテスト
  - ステート更新のテスト
  - エラーハンドリングのテスト
  - _Requirements: REQ-2_

- [ ] 5.2 workRecordストア拡張実装
  - missingDatesステート追加
  - `fetchMissingDates`アクション実装
  - APIクライアント（services/api.ts）にmissing-datesエンドポイント追加
  - _Requirements: REQ-2_

- [ ] 6. カレンダーウィジェットコンポーネント作成
- [ ] 6.1 CalendarWidgetコンポーネントのテスト作成
  - コンポーネントマウントテスト
  - プロップス検証テスト
  - イベントエミットテスト
  - _Requirements: REQ-1_

- [ ] 6.2 CalendarWidget.vue実装
  - Vuetifyカレンダーコンポーネントをベースに実装
  - year, month, missingDatesプロップス定義
  - カレンダーグリッドレイアウト作成
  - 月切り替えナビゲーション実装
  - _Requirements: REQ-1_

- [ ] 7. カレンダーセルコンポーネント作成
- [ ] 7.1 CalendarCellコンポーネントのテスト作成
  - 色分け表示のテスト（未入力=赤）
  - クリックイベントのテスト
  - 週末・祝日表示のテスト
  - _Requirements: REQ-2, REQ-3_

- [ ] 7.2 CalendarCell.vue実装
  - date, isMissing, isWeekend, isHolidayプロップス定義
  - 条件付きスタイリング（赤色: #FF4444）実装
  - クリックハンドラー実装
  - ホバー効果実装
  - _Requirements: REQ-2, REQ-3, REQ-4_

- [ ] 8. 未入力日数インジケーター作成
- [ ] 8.1 MissingDatesIndicatorコンポーネントのテスト
  - カウント表示のテスト
  - ゼロ件時の表示テスト
  - _Requirements: REQ-2_

- [ ] 8.2 MissingDatesIndicator.vue実装
  - countプロップス定義
  - 視覚的に目立つバッジスタイル実装
  - アイコン付き表示実装
  - _Requirements: REQ-2_

## Phase 3: 統合とナビゲーション

- [ ] 9. WorkRecordViewへの統合
- [ ] 9.1 WorkRecordView統合テスト作成
  - カレンダーウィジェット表示テスト
  - ストアとの連携テスト
  - ナビゲーションテスト
  - _Requirements: REQ-1, REQ-4_

- [ ] 9.2 WorkRecordView.vue更新
  - CalendarWidgetコンポーネントをインポート・配置
  - ページロード時のfetchMissingDates呼び出し
  - カレンダーを工数入力フォーム上部に配置
  - _Requirements: REQ-1, REQ-4_

- [ ] 10. 日付クリックナビゲーション実装
- [ ] 10.1 日付ナビゲーションのテスト作成
  - 日付クリック時のルーティングテスト
  - 選択日付の入力フォーム反映テスト
  - ブラウザ戻るボタンのテスト
  - _Requirements: REQ-4_

- [ ] 10.2 日付ナビゲーション機能実装
  - CalendarCellクリック時のイベントハンドラー実装
  - Vue Routerでクエリパラメータ（?date=YYYY-MM-DD）追加
  - WorkRecordViewで選択日付の入力フォーム表示
  - _Requirements: REQ-4_

## Phase 4: リアルタイム更新

- [ ] 11. リアルタイム状態更新機能
- [ ] 11.1 リアルタイム更新のテスト作成
  - 工数保存後のカレンダー更新テスト
  - イベントバスのテスト
  - 更新アニメーションのテスト
  - _Requirements: REQ-5_

- [ ] 11.2 リアルタイム更新実装
  - 工数保存成功時のstatus-changedイベント発火
  - CalendarWidgetでのイベントリスニング
  - updateCellColorメソッド実装（未入力→承認待ちへの色変更）
  - CSSトランジション追加（0.3秒のフェード効果）
  - _Requirements: REQ-5_

## Phase 5: パフォーマンス最適化とテスト

- [ ] 12. パフォーマンス最適化
- [ ] 12.1 フロントエンド最適化
  - CalendarWidgetの遅延ローディング実装（defineAsyncComponent）
  - computedプロパティでmissingDatesCountメモ化
  - 大量日付表示時の仮想スクロール実装（必要に応じて）
  - _Requirements: REQ-1_

- [ ] 12.2 バックエンド最適化
  - SQLクエリのEXPLAIN分析
  - インデックス効果の検証（50ms以内の応答確認）
  - N+1クエリ問題のチェックと解決
  - _Requirements: REQ-2_

- [ ] 13. E2Eテスト実装
- [ ] 13.1 カレンダー表示E2Eテスト
  - Playwrightでカレンダー初期表示テスト
  - 月切り替えナビゲーションテスト
  - 未入力日の赤色表示確認
  - _Requirements: REQ-1, REQ-2_

- [ ] 13.2 工数入力フローE2Eテスト
  - カレンダーから日付選択→入力→保存の一連フロー
  - 保存後のカレンダー色更新確認
  - エラーケースのテスト（ネットワークエラー等）
  - _Requirements: REQ-4, REQ-5_

## Phase 6: 最終統合

- [ ] 14. 全体統合とスモークテスト
- [ ] 14.1 統合テスト実装
  - Backend APIとFrontendの完全統合テスト
  - 月初から現在日までの全日付チェック
  - 権限別アクセステスト（開発者、PMO）
  - _Requirements: REQ-1, REQ-2, REQ-3, REQ-4, REQ-5_

- [ ] 14.2 最終動作確認
  - 開発環境での完全動作確認
  - パフォーマンス目標達成確認（初期表示2秒以内、更新500ms以内）
  - ブラウザ互換性確認（Chrome, Firefox, Safari, Edge）
  - _Requirements: REQ-1, REQ-2, REQ-3, REQ-4, REQ-5_