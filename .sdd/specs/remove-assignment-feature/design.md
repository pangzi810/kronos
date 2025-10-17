# 技術設計書

## 1. 概要

### 1.1 目的
プロジェクトアサイン機能（ProjectAssignment）を完全に廃止し、システムアーキテクチャを簡素化します。これにより、ユーザーとプロジェクトの関係性管理が不要となり、全てのユーザーが全ての有効なプロジェクトに対して工数を記録できるようになります。

### 1.2 設計目標
- **簡素化**: 複雑なアサインロジックを削除し、システムをシンプルに保つ
- **保守性向上**: コードベースのサイズを削減し、メンテナンスを容易にする
- **後方互換性**: 既存の工数データの整合性を保ちながら移行する
- **安全な移行**: データ損失なしでスムーズに機能を削除する

## 2. システムアーキテクチャの変更

### 2.1 レイヤーごとの影響

#### Domain Layer
**削除対象エンティティ・サービス：**
- `ProjectAssignment` エンティティ
- `ProjectAssignmentRepository` インターフェース
- `ProjectAssignmentValidator` ドメインサービス
- `SubordinateProjectAssignmentService` ドメインサービス
- `ProjectAssignmentEvent` イベントクラス
- `ProjectAssignmentEventRepository` インターフェース
- `ProjectAllocationService` ドメインサービス

#### Application Layer
**削除対象サービス：**
- `ProjectAssignmentApplicationService` アプリケーションサービス

**更新対象サービス：**
- `WorkRecordApplicationService`: アサインチェックロジックの削除（現在依存なし）

#### Infrastructure Layer
**削除対象実装：**
- `ProjectAssignmentRepositoryImpl` リポジトリ実装
- `ProjectAssignmentEventRepositoryImpl` イベントリポジトリ実装
- `ProjectAssignmentMapper` MyBatisマッパー
- `ProjectAssignmentEventPublisher` Kafkaパブリッシャー

#### Presentation Layer
**削除対象コンポーネント：**
- `ProjectAssignmentController` RESTコントローラー
- `ProjectAssignmentCreateRequest` リクエストDTO

### 2.2 依存関係の解決

**ProjectAllocationService依存の削除：**
```java
// 削除前: ProjectAssignmentApplicationService
private final ProjectAllocationService projectAllocationService;

// 削除後: 該当クラス自体を削除
```

## 3. データベース設計

### 3.1 テーブル削除

**Flyway Migration V34__drop_project_assignments_table.sql:**
```sql
-- バックアップテーブルの作成（履歴保存用）
CREATE TABLE IF NOT EXISTS project_assignments_backup AS
SELECT * FROM project_assignments;

-- 外部キー制約の削除
ALTER TABLE work_records 
DROP FOREIGN KEY IF EXISTS fk_work_records_project_assignments;

-- インデックスの削除
DROP INDEX IF EXISTS idx_assignments_project_id ON project_assignments;
DROP INDEX IF EXISTS idx_assignments_user_id ON project_assignments;
DROP INDEX IF EXISTS idx_assignments_is_active ON project_assignments;
DROP INDEX IF EXISTS idx_assignments_project_active ON project_assignments;
DROP INDEX IF EXISTS idx_project_assignments_duplicate_check ON project_assignments;
DROP INDEX IF EXISTS idx_project_assignments_user_active ON project_assignments;
DROP INDEX IF EXISTS idx_project_assignments_project_user_active ON project_assignments;

-- テーブルの削除
DROP TABLE project_assignments;

-- バックアップテーブルにコメント追加
ALTER TABLE project_assignments_backup 
COMMENT = 'プロジェクトアサインメント履歴（廃止機能のバックアップ）';
```

### 3.2 関連テーブルの更新

work_recordsテーブルに外部キー制約がある場合は削除が必要です（現在の構造では直接の依存はない）。

## 4. バックエンド実装設計

### 4.1 削除対象ファイル一覧

**Domainレイヤー:**
```
backend/src/main/java/com/devhour/domain/
├── model/entity/ProjectAssignment.java
├── repository/
│   ├── ProjectAssignmentRepository.java
│   └── ProjectAssignmentEventRepository.java
├── service/
│   ├── ProjectAssignmentValidator.java
│   ├── SubordinateProjectAssignmentService.java
│   └── ProjectAllocationService.java
└── event/ProjectAssignmentEvent.java
```

**Applicationレイヤー:**
```
backend/src/main/java/com/devhour/application/
└── service/ProjectAssignmentApplicationService.java
```

**Infrastructureレイヤー:**
```
backend/src/main/java/com/devhour/infrastructure/
├── repository/
│   ├── ProjectAssignmentRepositoryImpl.java
│   └── ProjectAssignmentEventRepositoryImpl.java
├── mapper/ProjectAssignmentMapper.java
└── kafka/ProjectAssignmentEventPublisher.java
```

**Presentationレイヤー:**
```
backend/src/main/java/com/devhour/presentation/
├── controller/ProjectAssignmentController.java
└── dto/request/ProjectAssignmentCreateRequest.java
```

**テストファイル:**
```
backend/src/test/java/com/devhour/
├── domain/
│   ├── model/entity/ProjectAssignmentTest.java
│   ├── service/
│   │   ├── ProjectAssignmentValidatorTest.java
│   │   └── SubordinateProjectAssignmentServiceTest.java
│   └── event/ProjectAssignmentEventTest.java
├── application/service/
│   ├── ProjectAssignmentApplicationServiceTest.java
│   └── ProjectAssignmentApplicationServiceEventTest.java
├── infrastructure/
│   ├── repository/
│   │   ├── ProjectAssignmentRepositoryImplTest.java
│   │   └── ProjectAssignmentEventRepositoryImplTest.java
│   ├── mapper/ProjectAssignmentMapperTest.java
│   └── kafka/ProjectAssignmentEventPublisherTest.java
└── presentation/controller/ProjectAssignmentControllerTest.java
```

### 4.2 APIエンドポイントの削除

**削除対象エンドポイント:**
- `POST /api/project-assignments` - プロジェクトアサインの作成
- `GET /api/project-assignments` - アサイン一覧の取得
- `GET /api/project-assignments/{id}` - 特定のアサインの取得
- `DELETE /api/project-assignments/{id}` - アサインの削除
- `GET /api/project-assignments/user/{userId}` - ユーザーのアサイン取得
- `GET /api/project-assignments/project/{projectId}` - プロジェクトのアサイン取得

### 4.3 工数入力ロジックの変更

**WorkRecordApplicationService の変更:**
```java
// 削除前のアサインチェック（もし存在する場合）
private void validateProjectAssignment(String userId, String projectId) {
    // アサインの確認ロジック
}

// 削除後（プロジェクトの有効性のみ確認）
private void validateProject(String projectId) {
    Project project = projectRepository.findById(projectId)
        .orElseThrow(() -> new IllegalArgumentException("プロジェクトが存在しません"));
    
    if (!project.isActive()) {
        throw new IllegalStateException("無効なプロジェクトです");
    }
}
```

## 5. フロントエンド実装設計

### 5.1 削除対象ファイル一覧

**Vueコンポーネント:**
```
frontend/src/
├── views/
│   └── AssignmentView.vue
├── components/assignment/
│   ├── AssignmentProjectToDeveloperDialog.vue
│   ├── AssignmentSubordinateList.vue
│   ├── AssignmentDeveloperDetailDialog.vue
│   ├── AssignmentDeveloperToProjectDialog.vue
│   └── AssignmentProjectList.vue
├── composables/
│   └── useProjectAssignment.ts
└── services/domains/
    └── project-assignment.service.ts
```

**テストファイル:**
```
frontend/src/
├── views/__tests__/
│   └── AssignmentView.spec.ts
└── components/assignment/__tests__/
    └── AssignmentProjectToDeveloperDialog.spec.ts
```

### 5.2 ルーティングの更新

**router/index.ts の変更:**
```typescript
// 削除前
{
  path: '/assignments',
  name: 'assignments',
  component: () => import('../views/AssignmentView.vue'),
  meta: { requiresAuth: true }
}

// 削除後: このルート定義を完全に削除
```

### 5.3 メニューナビゲーションの更新

**App.vue または NavigationMenu コンポーネントの変更:**
```vue
<!-- 削除前 -->
<v-list-item to="/assignments" prepend-icon="mdi-account-multiple">
  <v-list-item-title>プロジェクトアサイン</v-list-item-title>
</v-list-item>

<!-- 削除後: この項目を削除 -->
```

### 5.4 工数入力画面の変更

**WorkRecordView.vue の変更:**
```typescript
// 削除前: アサインされたプロジェクトのみ取得
const assignedProjects = await projectAssignmentService.getUserProjects(userId);

// 削除後: 全ての有効なプロジェクトを取得
const allProjects = await projectService.getActiveProjects();
```

## 6. APIレスポンスの変更

### 6.1 プロジェクト一覧API

**変更前のレスポンス:**
```json
{
  "projects": [
    {
      "id": "project-1",
      "name": "プロジェクトA",
      "isAssigned": true
    }
  ]
}
```

**変更後のレスポンス:**
```json
{
  "projects": [
    {
      "id": "project-1",
      "name": "プロジェクトA"
      // isAssigned フィールドを削除
    }
  ]
}
```

## 7. テスト戦略

### 7.1 削除するテストケース

- ProjectAssignment関連の全ての単体テスト
- ProjectAssignmentApplicationServiceの統合テスト
- アサイン重複チェックのテスト
- アサイン権限チェックのテスト

### 7.2 更新するテストケース

**WorkRecordApplicationServiceTest:**
```java
@Test
void testCreateWorkRecordWithoutAssignmentCheck() {
    // アサインチェックなしで工数記録が作成できることを確認
    // プロジェクトの有効性のみチェック
}
```

### 7.3 新規テストケース

**全プロジェクトアクセステスト:**
```java
@Test
void testUserCanAccessAllActiveProjects() {
    // 任意のユーザーが全ての有効なプロジェクトにアクセスできることを確認
}
```

## 8. 移行計画

### 8.1 段階的移行アプローチ

**Phase 1: 準備（1日）**
- バックアップスクリプトの準備
- 移行手順書の作成
- ロールバック計画の策定

**Phase 2: バックエンド削除（2日）**
- ProjectAssignment関連クラスの削除
- コンパイルエラーの解決
- 単体テストの更新

**Phase 3: フロントエンド削除（2日）**
- Vue コンポーネントの削除
- ルーティングとナビゲーションの更新
- E2Eテストの更新

**Phase 4: データベース移行（1日）**
- バックアップテーブルの作成
- project_assignmentsテーブルの削除
- インデックスのクリーンアップ

**Phase 5: 統合テスト（1日）**
- 全体的な動作確認
- パフォーマンステスト
- セキュリティチェック

### 8.2 ロールバック計画

```sql
-- ロールバックSQL
CREATE TABLE project_assignments AS
SELECT * FROM project_assignments_backup;

-- インデックスの再作成
CREATE INDEX idx_assignments_project_id ON project_assignments(project_id);
-- 他のインデックスも同様に再作成
```

## 9. リスク分析と対策

### 9.1 識別されたリスク

| リスク | 影響度 | 発生確率 | 対策 |
|--------|--------|----------|------|
| 既存の工数データへの影響 | 高 | 低 | バックアップテーブルの作成、トランザクション使用 |
| 権限管理の混乱 | 中 | 中 | 明確なドキュメント作成、ユーザー教育 |
| APIクライアントの破損 | 高 | 中 | 段階的廃止、エラーハンドリング |
| パフォーマンス低下 | 低 | 低 | 事前のパフォーマンステスト実施 |

### 9.2 緊急時対応

1. **即座のロールバック**: バックアップテーブルからの復元
2. **部分的な機能停止**: 工数入力機能の一時停止
3. **代替ワークフロー**: 管理者による手動データ入力

## 10. セキュリティ考慮事項

### 10.1 アクセス制御の変更

**変更前:**
- プロジェクトアサインに基づくアクセス制御
- ロールベースの権限チェック

**変更後:**
- 認証状態のみでアクセス許可
- プロジェクトの有効性チェックのみ

### 10.2 監査ログ

全ての削除操作と変更をaudit_logsテーブルに記録：
```sql
INSERT INTO audit_logs (action, entity_type, details, user_id, timestamp)
VALUES ('DELETE', 'PROJECT_ASSIGNMENT_FEATURE', 'Feature removal migration', 'system', NOW());
```

## 11. ドキュメント更新

### 11.1 更新対象ドキュメント

- **README.md**: ProjectAssignment機能の記述を削除
- **CLAUDE.md**: アーキテクチャ説明からProjectAssignmentを削除
- **.sdd/steering/structure.md**: ファイル構造からProjectAssignment関連を削除
- **API仕様書（Swagger）**: ProjectAssignmentエンドポイントを削除

### 11.2 ユーザーマニュアル

- アサイン管理セクションを削除
- 工数入力手順を更新（全プロジェクト選択可能）

## 12. 検証基準

### 12.1 機能検証

- [ ] 全てのユーザーが有効なプロジェクトを選択できる
- [ ] 工数入力がアサインなしで正常に動作する
- [ ] 既存の工数データが正しく表示される
- [ ] ProjectAssignment APIが404を返す

### 12.2 技術検証

- [ ] コンパイルエラーがない
- [ ] 全てのテストが通過する
- [ ] テストカバレッジ80%以上を維持
- [ ] パフォーマンスの劣化がない

## 13. 実装優先順位

1. **最優先**: バックエンドのProjectAssignment削除
2. **高優先**: フロントエンドのアサイン画面削除
3. **中優先**: データベーススキーマの更新
4. **低優先**: ドキュメントの更新

## 14. 完了条件

- 全てのProjectAssignment関連コードが削除されている
- project_assignmentsテーブルが削除されている
- 全ユーザーが全プロジェクトに工数入力可能
- テストカバレッジ80%以上
- ドキュメントが更新されている