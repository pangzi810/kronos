# Project Structure

## Root Directory Organization

```
kronos/
├── kronos-be/                  # Spring Boot backend application
├── kronos-fe/                  # Vue 3 frontend application
├── .sdd/                      # Spec-driven development artifacts
│   ├── specs/                 # Feature specifications
│   └── steering/              # Project steering documents
├── docker-compose.yml         # Multi-container orchestration
├── README.md                  # Project documentation
├── CLAUDE.md                  # AI assistant guidelines
└── .gitignore                # Version control exclusions
```

## Backend Structure (`/kronos-be`)

### Source Code Organization (`src/main/java/com/devhour`)

#### Domain Layer (`domain/`)
**Purpose**: Core business logic and domain models

```
domain/
├── model/
│   ├── entity/               # Domain entities (aggregate roots)
│   │   ├── Project.java
│   │   ├── User.java
│   │   ├── WorkRecord.java
│   │   ├── WorkCategory.java
│   │   ├── ApproverRelationship.java
│   │   ├── WorkRecordApproval.java
│   │   ├── ApprovalHistory.java
│   │   ├── JqlQuery.java
│   │   ├── FieldMapping.java
│   │   ├── SyncHistory.java
│   │   ├── SyncHistoryDetail.java
│   └── valueobject/          # Immutable value objects
│       ├── ProjectStatus.java
│       ├── ApprovalStatus.java
│       ├── CategoryCode.java
│       ├── CategoryHours.java
│       └── DisplayOrder.java
├── repository/               # Repository interfaces (contracts)
├── service/                  # Domain services (cross-entity logic)
│   └── JiraSyncDomainService.java
├── exception/                # Domain-specific exceptions
└── event/                    # Domain events
```

#### Application Layer (`application/`)
**Purpose**: Use case orchestration and application services

```
application/
├── service/                  # Application services (use cases)
│   ├── ProjectApplicationService.java
│   ├── UserApplicationService.java
│   ├── WorkRecordApplicationService.java
│   ├── AuthService.java
│   ├── DailyApprovalApplicationService.java
│   ├── ApproverApplicationService.java
│   ├── JiraSyncApplicationService.java
│   └── OktaUserSyncService.java
└── dto/                      # Data transfer objects
    ├── AuthResponse.java
    ├── WorkHoursSummaryResponse.java
    └── ActiveDeveloperResponse.java
```

#### Infrastructure Layer (`infrastructure/`)
**Purpose**: Technical implementations and external integrations

```
infrastructure/
├── repository/               # Repository implementations
│   └── *RepositoryImpl.java files
├── mapper/                   # MyBatis mapper interfaces
│   ├── UserMapper.java
│   ├── ProjectMapper.java
│   └── WorkRecordMapper.java
├── security/                 # Security implementations
│   ├── JwtUtil.java
│   ├── JwtAuthenticationFilter.java
│   ├── CustomUserDetailsService.java
│   ├── OktaSecurityConfig.java
│   └── OktaScopeConverter.java
├── typehandler/             # MyBatis type handlers
├── aspect/                  # AOP aspects (logging, monitoring)
├── kafka/                   # Event streaming
├── jira/                    # JIRA integration (implemented)
│   ├── JiraClient.java
│   ├── dto/JiraIssueSearchResponse.java
│   └── exception/Jira*.java
├── scheduler/               # Scheduled task infrastructure
│   └── JiraSyncScheduler.java
└── velocity/                # Velocity template processing
```

#### Presentation Layer (`presentation/`)
**Purpose**: REST API endpoints and web layer

```
presentation/
├── controller/              # REST controllers
│   ├── AuthController.java
│   ├── ProjectController.java
│   ├── UserController.java
│   └── WorkRecordController.java
├── request/                 # Request DTOs with validation
│   ├── LoginRequest.java
│   ├── ProjectCreateRequest.java
│   └── WorkRecordCreateRequest.java
├── response/                # Response DTOs
├── dto/                     # Additional DTOs
└── handler/                 # Exception handlers
    └── GlobalExceptionHandler.java
```

#### Configuration (`config/`)
**Purpose**: Spring Boot configuration classes

```
config/
├── SecurityConfig.java      # Spring Security configuration
├── MyBatisConfig.java      # MyBatis configuration
├── AspectConfig.java       # AOP configuration
├── AuditLogConfig.java     # Audit logging setup
├── SchedulingConfig.java   # Scheduled tasks configuration
├── JiraConfiguration.java  # JIRA integration configuration
└── JiraClientConfiguration.java # JIRA client configuration
```

### Resources (`src/main/resources`)

```
resources/
├── db/migration/           # Flyway migration scripts (consolidated)
│   └── V1__init.sql      # Complete schema definition with all tables and initial data
├── application.properties  # Default configuration with Okta OAuth2 settings
├── application-docker.properties
├── application-production.properties
├── .env.development        # Development environment variables (Okta issuer, JWK Set URI, client ID)
└── logback-spring.xml     # Logging configuration
```

### Test Structure

#### Backend Tests (`src/test/java/com/devhour`)
```
test/
├── unit/                   # Unit tests by layer
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
├── integration/            # Integration tests
│   ├── *IntegrationTest.java
│   └── *E2ETest.java
└── performance/            # Performance tests
    └── *PerformanceTest.java
```

#### Frontend Tests (`src/__tests__`)
```
__tests__/
├── views/                  # View component tests
│   └── CallbackView.test.ts
└── main.test.ts           # Application initialization tests
```

## Frontend Structure (`/kronos-fe`)

### Source Code (`src/`)

```
src/
├── components/             # Reusable Vue components
│   ├── admin/             # Admin-specific components
│   ├── approval/          # Approval workflow components
│   ├── jira/              # JIRA integration components
│   │   ├── JiraConnectionSettings.vue
│   │   ├── JqlQueryManager.vue
│   │   ├── ManualSyncTrigger.vue
│   │   ├── ResponseTemplateEditor.vue
│   │   ├── ResponseTemplateManager.vue
│   │   ├── SyncHistoryViewer.vue
│   │   └── SyncStatusMonitor.vue
│   ├── common/            # Common reusable components
│   │   └── DatePicker.vue # Date picker with status visualization and month-change events
│   ├── project/           # Project management components
│   ├── shared/            # Shared/common components
│   ├── user/              # User management components
│   └── work-hours/        # Work hours tracking components
│       ├── CalendarWidget.vue
│       ├── CalendarCell.vue
│       ├── WorkHoursTable.vue  # Work hours input table with autocomplete and validation
│       └── WorkHoursSummary.vue
├── views/                 # Page-level components (routes)
│   ├── AnalysisView.vue
│   ├── ApprovalView.vue
│   ├── JiraSettingsView.vue
│   ├── JqlQueryView.vue
│   ├── LoginView.vue
│   ├── ProjectsView.vue
│   ├── ResponseTemplateView.vue
│   ├── ApproverSettingView.vue
│   ├── SyncHistoryView.vue
│   ├── UsersView.vue
│   └── WorkRecordView.vue
├── composables/           # Vue Composition API utilities
│   ├── useApiError.ts
│   ├── useApproval.ts
│   ├── useErrorNotification.ts
│   ├── useJiraSync.ts
│   ├── useJqlQuery.ts
│   ├── useProject.ts
│   ├── useResponseTemplate.ts
│   ├── useSubordinateSelection.ts
│   ├── useSubordinateSelectionManager.ts
│   ├── useUtils.ts
│   └── useWorkRecord.ts
├── services/              # API service layer
│   ├── core/              # Core service infrastructure
│   │   ├── api.cache.ts
│   │   ├── api.client.ts
│   │   ├── base.service.ts
│   │   └── error.handler.ts
│   ├── domains/           # Domain-specific services
│   │   ├── approval.service.ts
│   │   ├── auth.service.ts
│   │   ├── developer.service.ts
│   │   ├── jira-sync.service.ts
│   │   ├── jira.service.ts
│   │   ├── project.service.ts
│   │   ├── subordinate.service.ts
│   │   ├── supervisor.service.ts
│   │   ├── user.service.ts
│   │   ├── work-category.service.ts
│   │   └── work-record.service.ts
│   └── types/             # Service type definitions
│       ├── auth.types.ts
│       ├── common.types.ts
│       ├── jira.types.ts
│       ├── project.types.ts
│       ├── subordinate.types.ts
│       ├── supervisor.types.ts
│       ├── user.types.ts
│       └── work-record.types.ts
├── stores/                # Pinia state management
│   ├── auth.ts            # Authentication state with 1-hour user caching
│   ├── dateStatuses.ts    # Multi-month date statuses cache (LRU, 6 months max, 5-min TTL)
│   ├── projects.ts
│   └── workRecords.ts
├── types/                 # TypeScript type definitions
│   ├── api.ts
│   ├── models.ts
│   └── components.ts
├── utils/                 # Utility functions
│   └── date.ts            # JST (Japan Standard Time) date utilities
├── router/                # Vue Router configuration
│   └── index.ts
├── config/                # Application configuration
│   └── okta.config.ts    # Okta OAuth2 configuration (uses environment variables)
├── i18n/                  # Internationalization
│   ├── index.ts
│   ├── locales/
│   │   ├── ja.json       # Japanese translations
│   │   └── zh.json       # Chinese translations
├── plugins/               # Vue plugins
├── assets/                # Static assets
│   ├── styles/
│   └── images/
├── App.vue                # Root component
└── main.ts                # Application entry point
```

### Configuration Files

```
kronos-fe/
├── package.json           # Dependencies and scripts
├── tsconfig.json         # TypeScript configuration
├── vite.config.ts        # Vite build configuration with envDir: './tools/environments'
├── .eslintrc.js          # ESLint rules
├── .prettierrc           # Prettier formatting
├── playwright.config.ts  # E2E test configuration
└── tools/
    └── environments/      # Environment variable files
        ├── .env.development   # Development environment (Okta issuer, client ID)
        ├── .env.staging       # Staging environment (Okta issuer, client ID)
        └── .env.production    # Production environment (Okta issuer, client ID)
```

## Spec-Driven Development (`.sdd/`)

```
.sdd/
├── specs/                 # Feature specifications
│   ├── development-hour-management/
│   ├── work-hours-approval/
│   ├── jira-project-sync/
│   └── [feature-name]/
│       ├── spec.json      # Specification metadata
│       ├── requirements.md
│       ├── design.md
│       └── tasks.md
└── steering/              # Project steering documents
    ├── product.md         # Product overview
    ├── tech.md           # Technology stack
    └── structure.md      # This file
```

## Code Organization Patterns

### Domain-Driven Design Principles
- **Entities**: Rich domain models with business logic
- **Value Objects**: Immutable objects representing concepts
- **Repositories**: Abstractions for data persistence
- **Services**: Domain logic spanning multiple entities
- **Factories**: Complex object creation (static methods)

### Layered Architecture Rules
1. **Dependency Direction**: Outer layers depend on inner layers
2. **Domain Independence**: Domain layer has no external dependencies
3. **Repository Pattern**: Interfaces in domain, implementations in infrastructure
4. **DTO Boundaries**: DTOs for layer communication

### Package Structure Conventions
- **By Layer First**: Primary organization by architectural layer
- **By Feature Second**: Within layers, group by feature/domain
- **Consistent Naming**: `*Service`, `*Repository`, `*Controller`
- **Test Mirroring**: Test structure mirrors source structure

## File Naming Conventions

### Java Files
- **Entities**: `Project.java`, `User.java` (singular nouns)
- **Value Objects**: `ProjectStatus.java`, `CategoryHours.java`
- **Services**: `*Service.java` (e.g., `ProjectApplicationService.java`)
- **Repositories**: `*Repository.java` interface, `*RepositoryImpl.java` implementation
- **Controllers**: `*Controller.java` (e.g., `ProjectController.java`)
- **DTOs**: `*Request.java`, `*Response.java`, `*DTO.java`
- **Tests**: `*Test.java` (e.g., `ProjectTest.java`)

### Vue Files
- **Components**: PascalCase (e.g., `WorkHoursTable.vue`)
- **Views**: PascalCase with View suffix (e.g., `ProjectsView.vue`)
- **Composables**: camelCase with use prefix (e.g., `useProjects.ts`)
- **Services**: camelCase (e.g., `projectService.ts`)
- **Stores**: camelCase (e.g., `projects.ts`)
- **Types**: camelCase for files, PascalCase for interfaces

### SQL Files
- **Migrations**: `V{number}__{description}.sql`
- **Number**: Sequential, no gaps
- **Description**: snake_case (e.g., `V1__create_users_table.sql`)

## Import Organization

### Java Import Order
1. Java standard library (`java.*`, `javax.*`)
2. Third-party libraries (`org.springframework.*`, etc.)
3. Application packages (`com.devhour.*`)
4. Static imports
5. Blank line between groups

### TypeScript Import Order
1. External libraries (`vue`, `axios`, etc.)
2. Internal aliases (`@/components`, `@/services`)
3. Relative imports (`./`, `../`)
4. Type imports (`import type`)
5. Blank line between groups

## Key Architectural Principles

### Separation of Concerns
- **Single Responsibility**: Each class/component has one reason to change
- **Interface Segregation**: Narrow, focused interfaces
- **Dependency Inversion**: Depend on abstractions, not concretions

### Domain Integrity
- **Rich Domain Models**: Business logic in entities, not services
- **Immutable Value Objects**: Prevent invalid state
- **Aggregate Boundaries**: Consistency boundaries for transactions

### Testing Strategy
- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test component interactions
- **E2E Tests**: Test complete user workflows
- **Test Coverage**: Minimum 80% for backend

### Code Quality Standards
- **No Commented Code**: Remove, don't comment out
- **Meaningful Names**: Self-documenting code
- **Small Methods**: Single purpose, < 20 lines
- **Consistent Formatting**: Enforced by tools

### Security Practices
- **Input Validation**: Validate at boundaries
- **Parameterized Queries**: Prevent SQL injection
- **Authentication**: JWT tokens, no sessions
- **Authorization**: Role-based access control
- **Audit Logging**: Track sensitive operations
