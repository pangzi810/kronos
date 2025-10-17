# Technology Stack

## Architecture
**Domain-Driven Design (DDD)** with clear separation of concerns across four layers:
- **Domain Layer**: Business logic and domain models
- **Application Layer**: Use case orchestration and DTOs  
- **Infrastructure Layer**: Data persistence and external integrations
- **Presentation Layer**: REST API endpoints and request/response handling

## Backend

### Core Technologies
- **Language**: Java 17+
- **Framework**: Spring Boot 3.5.4
- **Build Tool**: Gradle (Groovy DSL)
- **Data Access**: MyBatis 3.0.5 (annotation-based mappers)
- **Database**: MySQL 8.0 with HikariCP connection pooling
- **Migration**: Flyway 11.11.0 for database versioning

### Security & Authentication
- **Authentication**: Okta OAuth2/OpenID Connect SSO (password-less)
- **OAuth2 Resource Server**: Spring Security OAuth2 Resource Server
- **Okta Integration**: 
  - OAuth2 token validation with Okta issuer
  - Scope-based authorization for user permissions
  - Automatic user synchronization service (OktaUserSyncService)
  - User status management (ACTIVE, INACTIVE, SUSPENDED enum)
  - Last login tracking for activity monitoring
  - Unique Okta user mapping with database constraints
- **Session Management**: Stateless OAuth2 tokens
- **Audit Logging**: Comprehensive activity tracking

### Additional Backend Libraries
- **Lombok**: Reduce boilerplate code
- **Spring AOP**: Aspect-oriented programming for cross-cutting concerns
- **Spring Kafka**: Event-driven architecture support
- **SpringDoc OpenAPI**: 2.2.0 for API documentation (Swagger UI)
- **Spring Actuator**: Application monitoring and health checks
- **Spring Validation**: Request validation
- **Apache Velocity**: Template engine for JIRA response transformation
- **JsonPath (Jayway)**: JSON parsing for JIRA API responses
- **ShedLock**: Distributed scheduler lock management
- **JIRA REST API**: External system integration

### Testing (Backend)
- **Unit Testing**: JUnit 5 + Mockito
- **Integration Testing**: @SpringBootTest with TestContainers
- **Database Testing**: H2 in-memory database for tests
- **Coverage Tool**: JaCoCo with 80% minimum requirement
- **Test Containers**: MySQL test containers for integration tests

## Frontend

### Core Technologies
- **Framework**: Vue 3.5.18 with Composition API
- **Language**: TypeScript 5.8
- **Build Tool**: Vite 7.0.6
- **Package Manager**: npm (Node.js 20.19.0+ or 22.12.0+)
- **UI Framework**: Vuetify 3.9.4 (Material Design components)

### State Management & Routing
- **State Management**: Pinia 3.0.3
  - User authentication with 1-hour cache (auth store)
  - Multi-month date statuses with LRU cache (dateStatuses store, 6 months max, 5-min TTL)
- **Router**: Vue Router 4.5.1
- **Composables**: VueUse 13.7.0 for utility functions

### Frontend Libraries
- **HTTP Client**: Axios 1.11.0
- **Internationalization**: Vue i18n 12.0.0-alpha.3 (Japanese/Chinese support)
- **Date Handling**: date-fns 4.1.0
  - Custom JST (Japan Standard Time) utilities for timezone-consistent date operations
- **JSON Display**: vue-json-pretty 2.5.0
- **Notifications**: Vue Toastification 2.0.0-rc.5
- **Icons**: MDI Font 7.4.47
- **Utilities**: Lodash-es 4.17.21

### Frontend Testing
- **Unit Testing**: Vitest 3.2.4 with Vue Test Utils
  - Test files in `src/__tests__/` directory
  - Component and view testing with CallbackView tests
  - Main application initialization tests
- **E2E Testing**: Playwright 1.54.2
- **Type Checking**: vue-tsc 3.0.4
- **Linting**: ESLint 9.31.0 with Vue plugin
- **Formatting**: Prettier 3.6.2

## Development Environment

### Required Tools
- **Java**: JDK 17 or higher
- **Node.js**: 20.19.0+ or 22.12.0+
- **Docker**: For MySQL and containerized deployment
- **Docker Compose**: For multi-container orchestration
- **Git**: Version control

### IDE Recommendations
- **Backend**: IntelliJ IDEA with Spring Boot plugin
- **Frontend**: Visual Studio Code with Vue/TypeScript extensions
- **Database**: MySQL Workbench or DBeaver

## Common Commands

### Backend Development
```bash
# Start backend server
cd backend
./gradlew bootRun

# Run tests with coverage
./gradlew test
./gradlew check  # Verify 80% coverage

# Build application
./gradlew build

# Database migration
./gradlew flywayMigrate
./gradlew flywayClean  # Clean database (dev only)
```

### Frontend Development
```bash
# Install dependencies
cd frontend
npm install

# Development server
npm run dev

# Production build
npm run build

# Testing
npm run test:unit     # Unit tests
npm run test:e2e      # E2E tests
npm run type-check    # TypeScript validation
npm run lint          # ESLint
npm run format        # Prettier
```

### Docker Operations
```bash
# Start all services
docker-compose up --build

# Start database only
docker-compose up -d mysql

# Stop all services
docker-compose down

# Remove volumes
docker-compose down -v
```

## Environment Variables

### Backend Configuration
```properties
# Database
SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/devhour_db
SPRING_DATASOURCE_USERNAME=devhour_user
SPRING_DATASOURCE_PASSWORD=devhour_password

# Okta OAuth2 Configuration (Primary Authentication)
OKTA_OAUTH2_ISSUER=https://your-domain.okta.com/oauth2/default
OKTA_OAUTH2_CLIENT_ID=your-okta-client-id
OKTA_OAUTH2_CLIENT_SECRET=your-okta-client-secret
OKTA_OAUTH2_AUDIENCES=api://default

# JIRA Integration
JIRA_BASE_URL=https://your-domain.atlassian.net
JIRA_API_TOKEN=your-jira-api-token
JIRA_USERNAME=your-jira-username
JIRA_SYNC_ENABLED=true
JIRA_SYNC_CRON=0 0 * * * ?  # Hourly execution

# Server
SERVER_PORT=8080
SPRING_PROFILES_ACTIVE=development

# Logging
LOGGING_LEVEL_ROOT=INFO
LOGGING_LEVEL_COM_DEVHOUR=DEBUG
```

### Frontend Configuration
```env
# API Configuration
VITE_API_BASE_URL=http://localhost:8080/api
VITE_APP_TITLE=Development Hour Management

# Feature Flags
VITE_ENABLE_MOCK_DATA=false
VITE_ENABLE_DEBUG_MODE=false
```

### Docker Environment
```yaml
# MySQL Container
MYSQL_ROOT_PASSWORD=root_password
MYSQL_DATABASE=devhour_db
MYSQL_USER=devhour_user
MYSQL_PASSWORD=devhour_password
```

## Port Configuration

### Development Ports
- **Backend API**: 8080
- **Frontend Dev Server**: 5173 (Vite default)
- **MySQL Database**: 3306
- **Swagger UI**: 8080/swagger-ui.html

### Docker Ports
- **Backend Container**: 8080 → 8080
- **Frontend Container**: 80 → 3000
- **MySQL Container**: 3306 → 3306

## API Standards

### REST Conventions
- **Base Path**: `/api`
- **Versioning**: Not currently versioned
- **Authentication**: Bearer token in Authorization header
- **Content Type**: `application/json`
- **Date Format**: ISO 8601 (YYYY-MM-DD)

### Response Format
```json
{
  "data": {},        // Success response
  "error": {         // Error response
    "code": "ERROR_CODE",
    "message": "Error description",
    "details": {}
  }
}
```

## Database Schema

### Migration System
- **Consolidated Migration**: Single V1__init.sql containing complete schema definition
- **Previous System**: Previously 33 separate migrations (V1-V33) now consolidated
- **Benefits**: Simplified deployment and faster database initialization
- **Schema Coverage**: All tables, indexes, constraints, and initial data in one migration

### Key Tables
- `users`: User accounts and authentication
- `projects`: Project definitions with JIRA integration (jira_issue_key)
- `work_records`: Time entry records
- `work_categories`: Work type classifications
- `approver_relationships`: Organizational approval hierarchy
- `approval_histories`: Audit trail for approvals
- `work_record_approvals`: Approval status tracking
- `jql_queries`: JIRA JQL query configurations
- `response_template`: Velocity templates for JIRA response transformation
- `sync_histories`: JIRA synchronization execution logs
- `sync_history_details`: Detailed sync results per project
- `shedlock`: Distributed scheduler lock management

## Performance Considerations

### Backend Optimizations
- **Connection Pooling**: HikariCP for efficient database connections
- **Query Optimization**: Indexed columns for frequent queries
- **Caching**: Spring Cache abstraction (ready for Redis)
- **Batch Operations**: Bulk insert/update capabilities
- **Lazy Loading**: Efficient data fetching strategies

### Frontend Optimizations
- **Code Splitting**: Enhanced with manual chunk optimization for better performance
- **Tree Shaking**: Remove unused code with Vite's optimized build process
- **Compression**: Gzip/Brotli for production builds
- **Lazy Loading**: Vue Router lazy route loading
- **Component Architecture**: Refactored components (e.g., WorkHoursTable) for better performance
- **Bundle Optimization**: Optimized dependency chunking for faster initial load times
- **Client-Side Caching**:
  - LRU cache for multi-month date statuses (6 months, 5-min TTL)
  - User authentication cache (1-hour TTL)

## Security Measures

### Application Security
- **CORS Configuration**: Restricted origins
- **CSRF Protection**: Disabled for stateless JWT
- **XSS Prevention**: Input sanitization
- **SQL Injection**: Parameterized queries via MyBatis
- **Rate Limiting**: Ready for implementation

### Authentication Flow

#### Okta OAuth2 Authentication (SSO)
1. User initiates SSO login via Okta authorization endpoint
2. User authenticates with Okta (redirect flow)
3. Okta returns authorization code to redirect URI (`/callback`)
4. Client exchanges code for OAuth2 access token
5. Client includes Bearer token in Authorization header
6. Server validates token with Okta issuer
7. Server maps Okta scopes to application permissions
8. Automatic user synchronization updates/creates user in local database
9. User status and last login timestamp updated
10. Application proceeds with authorized access

## Monitoring & Logging

### Logging Configuration
- **Framework**: Logback with Spring Boot
- **Log Levels**: Configurable per package
- **Log Rotation**: Daily rotation with compression
- **Audit Logging**: Dedicated audit.log file

### Health Checks
- **Actuator Endpoints**: `/actuator/health`, `/actuator/info`
- **Database Health**: Connection pool monitoring
- **Application Metrics**: Ready for Prometheus integration
