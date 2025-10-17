# 製品開発工数管理ツール (Development Hour Management Tool)

PMOと開発者が連携してプロジェクトの工数を効率的に管理・追跡するWebアプリケーションシステム。

## 🏗️ アーキテクチャ

- **フロントエンド**: Vue 3.5.18 + TypeScript 5.8 + Vuetify 3.9.4
- **バックエンド**: Java 17+ + Spring Boot 3.5.4 (DDD設計)
- **データベース**: MySQL 8.0 + HikariCP接続プール
- **データアクセス**: MyBatis 3.0.5 (アノテーションベース)
- **マイグレーション**: Flyway (V1-V19)
- **認証**: JWT + Spring Security
- **開発手法**: Test-First Development (TDD)
- **状態管理**: Pinia
- **国際化**: Vue i18n (日本語・中国語対応)

## 🚀 クイックスタート

### 前提条件
- Node.js 20.19.0+ または 22.12.0+
- Java 17+
- Docker & Docker Compose

### 開発環境セットアップ

1. **プロジェクトクローン**
```bash
git clone <repository-url>
cd dev-hour-management
```

2. **データベースの起動**
```bash
docker-compose up -d mysql
```

3. **バックエンド起動**
```bash
cd backend
./gradlew bootRun
```

4. **フロントエンド起動**
```bash
cd frontend
npm install
npm run dev
```

### Docker での起動
```bash
docker-compose up --build
```

## 📁 プロジェクト構造

```
├── backend/                 # Spring Boot Java アプリケーション (DDD設計)
│   ├── src/main/java/
│   │   └── com/devhour/
│   │       ├── domain/      # ドメイン層 (エンティティ・値オブジェクト)
│   │       ├── application/ # アプリケーション層 (ユースケース)
│   │       ├── infrastructure/ # インフラ層 (MyBatisマッパー・リポジトリ実装)
│   │       ├── presentation/  # プレゼンテーション層 (REST API)
│   │       └── config/      # 設定クラス
│   ├── src/main/resources/
│   │   └── db/migration/    # Flyway マイグレーションスクリプト
│   └── src/test/java/       # テストクラス
├── frontend/               # Vue 3 アプリケーション (大幅実装完了)
│   ├── src/
│   │   ├── components/     # Vue コンポーネント
│   │   │   ├── admin/      # 管理者機能コンポーネント
│   │   │   ├── approvals/  # 承認ワークフローコンポーネント
│   │   │   └── subordinates/ # 部下管理コンポーネント
│   │   ├── views/         # ページコンポーネント
│   │   ├── services/      # API サービス
│   │   ├── stores/        # Pinia状態管理
│   │   ├── composables/   # Vue Composition API
│   │   ├── i18n/          # 国際化設定・翻訳ファイル
│   │   └── types/         # TypeScript 型定義
│   └── tests/             # テストファイル
└── docker-compose.yml     # 開発環境 Docker 設定
```

## 🧪 テスト実行

**バックエンドテスト**
```bash
cd backend
./gradlew test
```

**フロントエンドテスト**
```bash
cd frontend
npm run test:unit      # Vitestユニットテスト
npm run test:e2e       # Playwright E2Eテスト
npm run type-check     # TypeScript型チェック
npm run lint           # ESLint
npm run format         # Prettier
```

**テストカバレッジ**
```bash
cd backend
./gradlew check        # 80%カバレッジ検証付きテスト実行
```

## 📊 主要機能

### 実装状況
- **バックエンド**: ✅ 完全実装 (DDD設計、79%テストカバレッジ)
- **フロントエンド**: ✅ 大幅実装完了 (Vue 3 + TypeScript)
- **認証システム**: ✅ JWT認証完全実装
- **承認ワークフロー**: ✅ 上司承認システム完全実装
- **多言語対応**: ✅ 日本語・中国語対応
- **データベース**: ✅ 19のFlywayマイグレーション完了

### PMO機能
- ✅ プロジェクト作成・編集・削除
- ✅ 進捗・工数レポート閲覧
- ✅ 上司関係管理
- ✅ 一括承認操作

### 開発者機能
- ✅ 日次工数記録
- ✅ 工数履歴確認
- ✅ プロジェクト進捗確認
- ✅ 工数入力状況の可視化

### 管理者機能
- ✅ ユーザー管理（作成・更新・削除）
- ✅ 上司関係設定
- ✅ ロールベースアクセス制御
- ✅ アクティブ開発者一覧
- ✅ 管理者ユーザー一覧
- ✅ 削除済みユーザー一覧

### 承認機能
- ✅ ユーザー・日付ベースの承認システム
- ✅ 承認履歴追跡
- ✅ 承認済み日付の入力制限
- ✅ 一括承認・却下機能
- ✅ コメント付き承認・却下

## 🔒 セキュリティ

- JWT ベースの認証
- ロールベースアクセス制御
- HTTPS 通信
- SQL インジェクション対策 (MyBatis パラメータクエリ)
- XSS 対策
- CORS 設定

## 🚀 デプロイメント

本プロジェクトは Docker を使用したコンテナデプロイメントに対応しています。

```bash
docker-compose -f docker-compose.prod.yml up -d
```

## 📝 API ドキュメント

開発サーバー起動後、以下でAPIドキュメントを確認できます:
- Swagger UI: http://localhost:8080/swagger-ui.html

## 🤝 開発ガイドライン

### DDD (Domain-Driven Design) アプローチ
- ドメインモデリングから開始
- ビジネスロジックをドメイン層に集約
- 依存関係の逆転によりドメインを中心とした設計

### Test-First Development
- TDD (Test-Driven Development) サイクルの採用
- Red-Green-Refactor サイクルの実践
- 実装前にテストを作成
- JaCoCo 80%テストカバレッジ要件（現在79%達成）

### 品質管理
- コミット前に必ずテストを実行（`./gradlew check`）
- PRには適切なテストケースを含める
- コードレビューを必須とする
- セキュリティベストプラクティスを遵守
- CI/CD パイプラインでの品質ゲート

### Spec-Driven Development (SDD)
- `.sdd/specs/`にフィーチャー仕様を管理
- `.sdd/steering/`にプロジェクト指針文書を管理
- 仕様駆動開発によるタスク管理
- 要件→設計→タスクの段階的開発プロセス

## 🌐 多言語対応

- **日本語**: プライマリ言語
- **中国語**: セカンダリ言語
- **英語**: 開発言語
- Vue i18nによる動的言語切り替え

## 📈 現在の開発状況

### 完了済み仕様
- ✅ **基本機能**: プロジェクト管理、ユーザー管理、工数記録
- ✅ **認証・認可**: JWT認証、ロールベースアクセス制御
- ✅ **承認ワークフロー**: 上司承認システム、承認履歴
- ✅ **ユーザー・日付承認**: 簡素化された承認単位
- ✅ **アクティブ開発者API**: `/users/active-developers`
- ✅ **管理者ユーザーAPI**: `/users/admin`
- ✅ **削除済みユーザーAPI**: `/users/deleted`
- ✅ **承認済み日付入力制限**: 承認後の修正防止

### 開発中・予定仕様
- 📋 **工数入力状況表示**: カレンダービューでの入力状況可視化

## 🛠️ 技術的特徴

### Domain-Driven Design (DDD)
- ドメイン層中心の設計
- エンティティ、値オブジェクト、ドメインサービス
- レポジトリパターンによるデータアクセス抽象化
- イベント駆動アーキテクチャ（Kafka統合）

### 最新技術スタック
- **Spring Boot 3.5.4**: 最新Spring Bootフレームワーク
- **Vue 3.5.18**: Composition API活用
- **TypeScript 5.8**: 型安全性確保
- **Vuetify 3.9.4**: Material Design UI
- **Pinia**: 現代的状態管理
- **Vite 7.0.6**: 高速ビルドツール

### パフォーマンス・品質
- MyBatisアノテーションベースマッピング
- JaCoCo テストカバレッジ監視
- Spring AOP クロスカッティング処理
- Flyway データベースマイグレーション管理