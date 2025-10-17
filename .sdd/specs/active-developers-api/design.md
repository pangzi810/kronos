# 技術設計書

## 概要
製品開発工数管理ツールにおいて、アクティブな開発者一覧を取得するREST APIエンドポイント `/api/users/active/developers` の技術設計。既存のDDD（ドメイン駆動設計）アーキテクチャおよびSpring Boot + MyBatis技術スタックとの統合を重視し、JWT認証、パフォーマンス最適化、包括的なエラーハンドリングを実装する。

## 要件マッピング

### 設計コンポーネントトレーサビリティ
各設計コンポーネントは特定の要件に対応：

- **UserController.getActiveDevelopers()** → 要件1.1-1.4: APIエンドポイント提供
- **UserApplicationService.findActiveDevelopers()** → 要件2.1-2.4: アクティブ開発者判定ロジック
- **UserRepository.findActiveDevelopers()** → 要件2.1-2.4: データアクセス層での開発者フィルタリング
- **Spring Security統合** → 要件3.1-3.4: JWT認証・認可
- **ActiveDeveloperResponse DTO** → 要件4.1-4.4: レスポンス構造標準化
- **Global Exception Handler** → 要件5.1-5.4: エラーハンドリング
- **MyBatisマッパー最適化** → 要件6.1-6.4: パフォーマンス最適化
- **アクセスログ・メトリクス** → 要件7.1-7.4: 運用監視対応

### ユーザーストーリーカバレッジ
- **PMO向け**: リソース管理効率化 → UserController + 応答時間最適化
- **システム管理者向け**: 正確なフィルタリング → UserApplicationServiceでの業務ロジック実装
- **セキュリティ管理者向け**: アクセス制御 → Spring Security JWT統合
- **API利用者向け**: 構造化データ → 標準化されたJSON レスポンス
- **システム運用者向け**: エラー診断 → 包括的例外処理とログ記録
- **システム利用者向け**: 高速応答 → データベースクエリ最適化

## アーキテクチャ

### システムアーキテクチャ図
```mermaid
graph TB
    A[クライアント] --> B[Spring Security<br/>JWT Filter]
    B --> C[UserController<br/>/api/users/active/developers]
    C --> D[UserApplicationService<br/>findActiveDevelopers()]
    D --> E[UserRepository<br/>findActiveDevelopers()]
    E --> F[UserMapper<br/>MyBatis]
    F --> G[MySQL Database<br/>users table]
    
    C --> H[ActiveDeveloperResponse DTO]
    C --> I[Global Exception Handler]
    C --> J[Access Logging]
    
    style B fill:#e1f5fe
    style C fill:#f3e5f5
    style D fill:#e8f5e8
    style E fill:#fff3e0
    style F fill:#fff3e0
```

### 技術スタック
**既存アーキテクチャとの完全統合**：

- **言語**: Java 17
- **フレームワーク**: Spring Boot 3.5.4（既存）
- **セキュリティ**: Spring Security + JWT（既存のJwtUtil、JwtAuthenticationFilter活用）
- **アーキテクチャパターン**: DDD（Domain-Driven Design）
- **データアクセス**: MyBatis 3.0.5（既存のアノテーションベースマッパー）
- **データベース**: MySQL 8.0（既存のusersテーブル活用）
- **コード生成**: Lombok（既存パターン準拠）

### アーキテクチャ決定根拠
**既存システムとの一貫性を重視した技術選択**：

- **Why Spring Boot REST**: 既存のUserControllerパターンとの統合により開発効率と保守性を確保
- **Why DDD継続**: 既存のレイヤー構造（Presentation→Application→Domain→Infrastructure）との整合性
- **Why MyBatis**: 既存のUserMapperパターンを活用し、SQLクエリの最適化とパフォーマンス制御が可能
- **Why JWT統合**: 実装済みのJwtAuthenticationFilterとの統合により、追加認証インフラ不要

## データフロー

### プライマリユーザーフロー：アクティブ開発者一覧取得

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Spring Security
    participant UC as UserController
    participant UAS as UserApplicationService
    participant UR as UserRepository
    participant UM as UserMapper
    participant DB as MySQL Database
    
    C->>S: GET /api/users/active/developers<br/>Authorization: Bearer {JWT}
    S->>S: JWTトークン検証
    alt JWT有効
        S->>UC: 認証済みリクエスト転送
        UC->>UAS: findActiveDevelopers()
        UAS->>UR: findActiveDevelopers()
        UR->>UM: selectActiveDevelopers()
        UM->>DB: SELECT id,name,email FROM users<br/>WHERE is_active=true AND role='DEVELOPER'
        DB-->>UM: クエリ結果
        UM-->>UR: User オブジェクトリスト
        UR-->>UAS: ドメインオブジェクト
        UAS-->>UC: ActiveDeveloper リスト
        UC->>UC: ActiveDeveloperResponse DTO変換
        UC-->>C: HTTP 200 + JSON レスポンス
    else JWT無効
        S-->>C: HTTP 401 Unauthorized
    end
```

### エラーハンドリングフロー

```mermaid
sequenceDiagram
    participant C as Client
    participant UC as UserController
    participant EH as GlobalExceptionHandler
    participant L as Logger
    
    UC->>UC: ビジネスロジック実行
    alt データベースエラー
        UC->>EH: DataAccessException
        EH->>L: エラーログ記録
        EH-->>C: HTTP 500 + エラーレスポンス
    else 予期しない例外
        UC->>EH: RuntimeException
        EH->>L: 詳細ログ記録
        EH-->>C: HTTP 500 + 汎用エラーレスポンス
    end
```

## コンポーネントとインターフェース

### バックエンドサービス＆メソッドシグネチャ

#### Presentation Layer
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
@Slf4j
public class UserController {
    
    /**
     * アクティブな開発者一覧を取得する
     * 認証済みユーザーのみアクセス可能
     */
    @GetMapping("/active/developers")
    @PreAuthorize("hasRole('USER')")
    public ResponseEntity<ActiveDeveloperResponse> getActiveDevelopers() {
        // 実装詳細は後述
    }
}
```

#### Application Layer
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserApplicationService {
    
    /**
     * アクティブな開発者一覧を取得し、業務ロジックを適用
     */
    public List<User> findActiveDevelopers() {
        // ドメインロジック：開発者ロール＋アクティブ状態のフィルタリング
    }
}
```

#### Domain Layer
```java
public interface UserRepository {
    
    /**
     * アクティブかつ開発者ロールのユーザーを取得
     * @return アクティブな開発者のリスト
     */
    List<User> findActiveDevelopers();
}
```

#### Infrastructure Layer
```java
@Mapper
public interface UserMapper {
    
    /**
     * アクティブな開発者をデータベースから取得
     * パフォーマンス最適化：必要なフィールドのみ選択
     */
    @Select("SELECT id, name, email FROM users " +
            "WHERE is_active = true AND role = 'DEVELOPER' " +
            "ORDER BY name ASC")
    List<User> selectActiveDevelopers();
}
```

### API エンドポイント仕様

| Method | Route | Purpose | Auth | Status Codes | Response Time |
|--------|-------|---------|------|--------------|---------------|
| GET | /api/users/active/developers | アクティブ開発者一覧取得 | JWT Required | 200, 401, 403, 500 | < 2秒 |

#### 詳細API仕様

**リクエスト例**：
```http
GET /api/users/active/developers
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

**成功レスポンス（HTTP 200）**：
```json
{
  "developers": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "name": "田中太郎",
      "email": "tanaka@example.com"
    },
    {
      "id": "987fcdeb-51a2-43d1-b789-123456789abc",
      "name": "佐藤花子",
      "email": "sato@example.com"
    }
  ],
  "totalCount": 2
}
```

**エラーレスポンス（HTTP 500）**：
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "内部サーバーエラーが発生しました",
    "timestamp": "2025-01-26T15:30:00Z"
  }
}
```

## データモデル

### ドメインエンティティ

1. **User**: 既存のUserエンティティを活用
   - ユーザー管理の中核エンティティ
   - isActive フィールドでアクティブ状態管理
   - role フィールドで開発者・PMO区分

### エンティティ関係図（関連部分）

```mermaid
erDiagram
    USER {
        string id PK
        string name
        string email
        boolean is_active
        string role
        datetime created_at
        datetime updated_at
    }
    
    USER ||--o{ PROJECT_ASSIGNMENT : "assigned_to"
    
    PROJECT_ASSIGNMENT {
        string id PK
        string user_id FK
        string project_id FK
        string project_role
        boolean is_active
    }
```

### DTOモデル定義

#### Response DTO
```java
/**
 * アクティブ開発者APIレスポンス
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ActiveDeveloperResponse {
    
    @JsonProperty("developers")
    private List<DeveloperInfo> developers;
    
    @JsonProperty("totalCount")
    private Integer totalCount;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class DeveloperInfo {
        
        @JsonProperty("id")
        private String id;
        
        @JsonProperty("name")
        private String name;
        
        @JsonProperty("email")
        private String email;
    }
}
```

#### Error Response DTO
```java
/**
 * 標準エラーレスポンス
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ErrorResponse {
    
    @JsonProperty("error")
    private ErrorDetail error;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ErrorDetail {
        
        @JsonProperty("code")
        private String code;
        
        @JsonProperty("message")
        private String message;
        
        @JsonProperty("timestamp")
        private Instant timestamp;
    }
}
```

### データベーススキーマ
**既存のusersテーブルを活用**：

```sql
-- 既存テーブル構造（参考）
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('PMO', 'DEVELOPER') NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- パフォーマンス最適化用インデックス
CREATE INDEX idx_users_active_role ON users(is_active, role);
```

### マイグレーション戦略
- **既存テーブル活用**: 新規マイグレーション不要
- **インデックス追加**: パフォーマンス最適化のためのコンポジットインデックス
- **後方互換性**: 既存機能への影響なし

## エラーハンドリング

### 包括的エラーハンドリング戦略

#### Global Exception Handler
```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    /**
     * データベースアクセスエラー
     */
    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<ErrorResponse> handleDataAccessException(DataAccessException ex) {
        log.error("Database access error occurred", ex);
        
        ErrorResponse errorResponse = ErrorResponse.builder()
            .error(ErrorResponse.ErrorDetail.builder()
                .code("DATABASE_ERROR")
                .message("データベースアクセスエラーが発生しました")
                .timestamp(Instant.now())
                .build())
            .build();
            
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(errorResponse);
    }
    
    /**
     * 予期しないエラー
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        log.error("Unexpected error occurred", ex);
        
        ErrorResponse errorResponse = ErrorResponse.builder()
            .error(ErrorResponse.ErrorDetail.builder()
                .code("INTERNAL_SERVER_ERROR")
                .message("内部サーバーエラーが発生しました")
                .timestamp(Instant.now())
                .build())
            .build();
            
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(errorResponse);
    }
}
```

### ログ記録戦略
```java
// アクセスログ
@GetMapping("/active/developers")
public ResponseEntity<ActiveDeveloperResponse> getActiveDevelopers() {
    log.info("Active developers API called by user: {}", SecurityContextHolder.getContext().getAuthentication().getName());
    
    try {
        // ビジネスロジック実行
        long startTime = System.currentTimeMillis();
        List<User> activeDevelopers = userApplicationService.findActiveDevelopers();
        long executionTime = System.currentTimeMillis() - startTime;
        
        log.info("Active developers retrieved: count={}, executionTime={}ms", 
                 activeDevelopers.size(), executionTime);
                 
        // レスポンス構築
        return ResponseEntity.ok(buildResponse(activeDevelopers));
        
    } catch (Exception ex) {
        log.error("Error retrieving active developers", ex);
        throw ex; // Global Exception Handlerで処理
    }
}
```

## セキュリティ考慮事項

### 認証・認可

#### JWT認証フロー
```mermaid
sequenceDiagram
    participant Client
    participant JwtAuthFilter
    participant JwtUtil
    participant UserController
    
    Client->>JwtAuthFilter: HTTPリクエスト + JWTトークン
    JwtAuthFilter->>JwtUtil: validateToken(token)
    alt トークン有効
        JwtUtil-->>JwtAuthFilter: ユーザー情報
        JwtAuthFilter->>UserController: 認証済みリクエスト
        UserController-->>Client: APIレスポンス
    else トークン無効
        JwtAuthFilter-->>Client: HTTP 401 Unauthorized
    end
```

#### 認可マトリックス

| Role | Endpoint Access | Notes |
|------|----------------|-------|
| PMO | ✅ Allowed | リソース管理者として開発者一覧が必要 |
| DEVELOPER | ✅ Allowed | プロジェクトチーム情報として参照可能 |
| 未認証 | ❌ Denied | HTTP 401 |

#### Security Configuration
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/users/active/developers")
                .authenticated() // 認証済みユーザーのみアクセス可能
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

### データ保護

#### 入力検証
```java
// リクエストパラメーター検証（現在のエンドポイントはパラメーターなし）
// 将来の拡張（ページネーション等）に備えた検証戦略

@GetMapping("/active/developers")
public ResponseEntity<ActiveDeveloperResponse> getActiveDevelopers(
    @RequestParam(defaultValue = "0") @Min(0) Integer offset,
    @RequestParam(defaultValue = "100") @Max(1000) Integer limit
) {
    // 実装詳細
}
```

#### 機密データ保護
- **パスワード除外**: User エンティティからパスワードフィールドを除外
- **JWT秘密鍵保護**: 環境変数による秘密鍵管理
- **ログ機密性**: 個人情報のログ出力を制限

### セキュリティベストプラクティス

#### OWASP Top 10 対策
1. **Injection Prevention**: MyBatisパラメーター化クエリ
2. **Authentication**: JWT強制認証
3. **Sensitive Data**: パスワード・秘密鍵除外
4. **Access Control**: Spring Security認可
5. **Security Misconfiguration**: セキュリティヘッダー設定
6. **Vulnerable Components**: 依存関係脆弱性管理
7. **Insufficient Logging**: 包括的アクセスログ
8. **API Security**: レート制限・入力検証

#### セキュリティヘッダー設定
```java
@Configuration
public class SecurityHeadersConfig {
    
    @Bean
    public SecurityFilterChain securityHeaders(HttpSecurity http) throws Exception {
        return http
            .headers(headers -> headers
                .frameOptions(HeadersConfigurer.FrameOptionsConfig::deny)
                .contentTypeOptions(HeadersConfigurer.ContentTypeOptionsConfig::and)
                .httpStrictTransportSecurity(HeadersConfigurer.HstsConfig::and)
            )
            .build();
    }
}
```

## パフォーマンス＆スケーラビリティ

### パフォーマンス目標

| メトリック | 目標値 | 測定方法 |
|-----------|--------|----------|
| レスポンス時間 (p95) | < 2秒 | Spring Boot Actuator + Micrometer |
| レスポンス時間 (p99) | < 5秒 | APM監視 |
| スループット | > 100 req/sec | 負荷テスト |
| データベースクエリ (p99) | < 50ms | MyBatis SQL ログ |
| 同時接続ユーザー | > 100 users | システム容量テスト |

### キャッシュ戦略

#### アプリケーションレベルキャッシュ
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class UserApplicationService {
    
    /**
     * 開発者一覧をキャッシュして高速化
     * TTL: 5分（開発者情報の更新頻度を考慮）
     */
    @Cacheable(value = "activeDevelopers", key = "'all'")
    public List<User> findActiveDevelopers() {
        log.info("Cache miss - fetching active developers from database");
        return userRepository.findActiveDevelopers();
    }
    
    /**
     * ユーザー情報更新時のキャッシュ無効化
     */
    @CacheEvict(value = "activeDevelopers", allEntries = true)
    public void evictActiveDevelopersCache() {
        log.info("Active developers cache evicted");
    }
}
```

#### キャッシュ設定
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(Duration.ofMinutes(5))
            .recordStats()
        );
        return cacheManager;
    }
}
```

### スケーラビリティアプローチ

#### 水平スケーリング
- **アプリケーションサーバー**: 複数インスタンス対応
- **データベース**: 読み取り専用レプリカ活用
- **ロードバランサー**: 複数インスタンス間でのリクエスト分散

#### データベース最適化
```java
// コンポジットインデックス活用
@Select("SELECT id, name, email FROM users " +
        "WHERE is_active = true AND role = 'DEVELOPER' " +
        "ORDER BY name ASC " +
        "LIMIT #{limit} OFFSET #{offset}")
List<User> selectActiveDevelopersWithPagination(
    @Param("limit") int limit, 
    @Param("offset") int offset
);
```

#### 接続プール最適化
```yaml
# application.yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 20000
      idle-timeout: 300000
      max-lifetime: 1200000
```

## テスト戦略

### テストカバレッジ要件
- **ユニットテスト**: ≥80% コードカバレッジ（JaCoCo強制）
- **統合テスト**: 全APIエンドポイント
- **E2Eテスト**: 主要ユーザージャーニー
- **パフォーマンステスト**: 2倍想定負荷でのテスト

### テストアプローチ

#### 1. ユニットテスト
```java
@ExtendWith(MockitoExtension.class)
class UserApplicationServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserApplicationService userApplicationService;
    
    @Test
    @DisplayName("アクティブな開発者一覧を正常に取得できること")
    void testFindActiveDevelopers_Success() {
        // Given
        List<User> mockUsers = createMockActiveDevelopers();
        when(userRepository.findActiveDevelopers()).thenReturn(mockUsers);
        
        // When
        List<User> result = userApplicationService.findActiveDevelopers();
        
        // Then
        assertThat(result).hasSize(2);
        assertThat(result.get(0).getRole()).isEqualTo("DEVELOPER");
        assertThat(result.get(0).isActive()).isTrue();
        
        verify(userRepository, times(1)).findActiveDevelopers();
    }
    
    private List<User> createMockActiveDevelopers() {
        return Arrays.asList(
            User.builder()
                .id("user1")
                .name("田中太郎")
                .email("tanaka@example.com")
                .role("DEVELOPER")
                .isActive(true)
                .build(),
            User.builder()
                .id("user2")
                .name("佐藤花子")
                .email("sato@example.com")
                .role("DEVELOPER")
                .isActive(true)
                .build()
        );
    }
}
```

#### 2. 統合テスト
```java
@SpringBootTest
@AutoConfigureTestDatabase
@Transactional
class UserControllerIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @Test
    @DisplayName("認証済みユーザーがアクティブ開発者一覧を取得できること")
    void testGetActiveDevelopers_Authenticated_Success() {
        // Given
        String jwt = jwtUtil.generateToken("testuser");
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(jwt);
        HttpEntity<?> entity = new HttpEntity<>(headers);
        
        // When
        ResponseEntity<ActiveDeveloperResponse> response = restTemplate.exchange(
            "/api/users/active/developers",
            HttpMethod.GET,
            entity,
            ActiveDeveloperResponse.class
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getDevelopers()).isNotEmpty();
        assertThat(response.getBody().getTotalCount()).isGreaterThan(0);
    }
    
    @Test
    @DisplayName("未認証ユーザーは401エラーになること")
    void testGetActiveDevelopers_Unauthenticated_Returns401() {
        // When
        ResponseEntity<ErrorResponse> response = restTemplate.getForEntity(
            "/api/users/active/developers",
            ErrorResponse.class
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
    }
}
```

#### 3. パフォーマンステスト
```java
@Test
@DisplayName("レスポンス時間が2秒以内であること")
void testGetActiveDevelopers_PerformanceRequirement() {
    // Given
    String jwt = jwtUtil.generateToken("testuser");
    
    // When & Then
    assertTimeout(Duration.ofSeconds(2), () -> {
        ResponseEntity<ActiveDeveloperResponse> response = 
            restTemplate.exchange(/* APIコール */);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    });
}
```

### CI/CDパイプライン

```mermaid
graph LR
    A[Code Push] --> B[Lint & Format]
    B --> C[Unit Tests<br/>80% Coverage]
    C --> D[Integration Tests]
    D --> E[Build JAR]
    E --> F[Security Scan]
    F --> G[Performance Tests]
    G --> H[Deploy to Staging]
    H --> I[E2E Tests]
    I --> J[Deploy to Production]
    
    style C fill:#ffebee
    style G fill:#e8f5e8
    style I fill:#fff3e0
```

#### テスト自動化設定
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      
      - name: Run tests with coverage
        run: ./gradlew test jacocoTestReport
      
      - name: Verify coverage threshold
        run: ./gradlew jacocoTestCoverageVerification
      
      - name: Performance tests
        run: ./gradlew performanceTest
```

### テストデータ管理
```java
@TestConfiguration
public class TestDataConfig {
    
    @Primary
    @Bean
    public UserRepository testUserRepository() {
        return new TestUserRepository(); // テストデータ付きモック実装
    }
}

// テスト用データセット
public class TestDataBuilder {
    
    public static List<User> createActiveDevelopers(int count) {
        return IntStream.range(0, count)
            .mapToObj(i -> User.builder()
                .id(UUID.randomUUID().toString())
                .name("開発者" + i)
                .email(String.format("dev%d@test.com", i))
                .role("DEVELOPER")
                .isActive(true)
                .build())
            .collect(Collectors.toList());
    }
}
```

---
**ステータス**: 技術設計完了
**次のステップ**: `/sdd:spec-tasks active-developers-api` を実行してタスクフェーズに進む