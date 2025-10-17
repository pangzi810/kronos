# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Development Hour Management Tool (製品開発工数管理ツール) - A Spring Boot application using Domain-Driven Design (DDD) architecture for tracking and managing development effort and resource allocation. The backend is fully implemented with comprehensive testing, while the frontend is planned for future development.

## Essential Commands

### Build and Run
```bash
# Backend development
cd backend
./gradlew bootRun

# Full environment with Docker
docker-compose up --build

# Database only
docker-compose up -d mysql
```

### Testing and Coverage (CRITICAL)
```bash
# Run tests with coverage report
cd backend
./gradlew test

# Verify 80% minimum coverage requirement
./gradlew check

# Run specific test class
./gradlew test --tests "*UserApplicationServiceTest"

# Run tests with debug output
./gradlew test --info

# Coverage report location
open backend/build/reports/jacoco/test/html/index.html
```

### Frontend (Vue 3 + TypeScript)
```bash
cd frontend
npm install
npm run dev       # Development server
npm run build     # Production build
npm run type-check # Type checking
npm run lint      # ESLint
npm run format    # Prettier formatting
```

### API Testing
```bash
cd backend/test-scripts
./test.sh  # Comprehensive API workflow test
```

## Architecture Overview

### Domain-Driven Design Layers

1. **Domain Layer** (`com.devhour.domain.*`)
   - **Entities**: Rich domain models with business logic (Project, User, WorkRecord, WorkCategory)
   - **Value Objects**: Immutable objects (ProjectStatus, CategoryCode, CategoryName, CategoryHours, DisplayOrder)
   - **Domain Services**: Cross-entity business logic
   - **Repository Interfaces**: Domain contracts for data persistence

2. **Application Layer** (`com.devhour.application.*`)
   - **Application Services**: Use case orchestration (ProjectApplicationService, UserApplicationService, etc.)
   - **DTOs**: Data transfer objects (AuthResponse, WorkHoursSummaryResponse)

3. **Infrastructure Layer** (`com.devhour.infrastructure.*`)
   - **Repository Implementations**: MyBatis-based data access
   - **Mappers**: MyBatis SQL mappers with annotations
   - **Type Handlers**: Custom converters for value objects
   - **Security**: JWT authentication (JwtUtil, JwtAuthenticationFilter)

4. **Presentation Layer** (`com.devhour.presentation.*`)
   - **Controllers**: REST API endpoints
   - **Request DTOs**: API request validation objects
   - **Exception Handler**: Global exception handling

### Key Technical Decisions

- **Data Access**: MyBatis with annotation-based mappers (no XML)
- **Authentication**: JWT tokens with Spring Security
- **Database Migration**: Flyway (V1-V6 migrations completed)
- **Testing**: JUnit 5 + Mockito with 80% JaCoCo coverage requirement
- **Build Tool**: Gradle with Kotlin DSL

### Database Schema

Database migrations implemented:
- V1: users table
- V2: projects table  
- V3: work_records table
- V5: work_categories table with initial data
- V7-V34: Additional features and enhancements

Note: ProjectAssignment functionality has been removed - all users can access all projects directly.

## Development Guidelines

### Test-First Development (MANDATORY)
1. Write tests before implementation
2. Achieve 80% minimum coverage
3. Run `./gradlew check` to verify coverage
4. No task is complete without passing coverage verification

### Repository Pattern Implementation
- Implement only basic CRUD operations
- Use `save` method for both create and update
- Prefer soft delete (logical deletion)
- **DO NOT** implement aggregation methods (count, sum, average) unless explicitly requested

### Code Style Conventions
- Entity getters: `getId()` format (not `getProjectId()`)
- Value Objects: `value()` method for records, `getValue()` for classes
- Follow existing patterns in codebase
- Use Lombok to reduce boilerplate

## Frontend Architecture

### Vue 3 + TypeScript Structure
```
frontend/src/
├── components/        # Reusable Vue components (WorkHoursTable, ExampleErrorHandling)
├── composables/       # Vue composition API utilities (useErrorNotification)
├── plugins/           # Vue plugins (errorNotification)
├── services/          # API client services (api, developerApi, errorHandler)
├── types/             # TypeScript type definitions (api.ts)
├── views/             # Page-level components (WorkRecordsView)
└── router/            # Vue Router configuration
```

### Frontend Services
- **api.ts**: Base axios configuration with JWT authentication
- **developerApi.ts**: Developer-specific API endpoints
- **errorHandler.ts**: Centralized error handling with Toast notifications

## Spec-Driven Development

### Active Specifications
Located in `.sdd/specs/`:
- **development-hour-management**: Main product specification
- **duplicate-member-assignment-prevention**: Bugfix specification
- **active-developers-api**: API endpoint for retrieving active developers list
- **admin-users-list-api**: API endpoint for retrieving admin users list (/users/admin)
- **work-hours-approval**: Work hours approval workflow by supervisors
- **work-hours-input-status**: Work hours input status visualization for developers
- **approved-date-input-restriction**: Prevent input modification for approved dates with visual indication
- **subordinate-project-assignment**: Assign projects to subordinates from a list view
- **missing-work-hours-indicator**: Display missing work hour entries from month start to current date
- **jira-project-sync**: JIRA API integration for automatic project synchronization with JQL queries
- **okta-authentication**: Okta認証システムの統合 (SSO implementation)
- **sailpoint-user-sync**: SailPoint Identity Security Cloudからの社員情報同期機能

### Workflow Commands
```bash
# Initialize new spec
/sdd:spec-init "Project description"

# Generate requirements
/sdd:spec-requirements {feature-name}

# Generate design (with interactive approval)
/sdd:spec-design {feature-name}

# Generate tasks (with interactive approval)
/sdd:spec-tasks {feature-name}
```

## Environment Configuration

### Ports
- Backend: 8080
- MySQL: 3306
- Frontend: 3000

### Environment Variables
- `SPRING_DATASOURCE_URL`: MySQL connection
- `SPRING_DATASOURCE_USERNAME`: Database user
- `SPRING_DATASOURCE_PASSWORD`: Database password
- `JWT_SECRET`: JWT signing key

### API Documentation
- Swagger UI: http://localhost:8080/swagger-ui.html (when running)

## Important Constraints

1. **Coverage Requirement**: 80% minimum enforced by JaCoCo
2. **Test Execution Policy**: All tests must pass before code integration
3. **No Test Skipping**: @Disabled annotations require justification
4. **Security**: Never commit secrets or keys to repository

## Common Troubleshooting

### Database Connection Issues
```bash
# Check MySQL container
docker ps
docker logs devhour-mysql

# Reset database
docker-compose down -v
docker-compose up -d mysql
```

### Test Failures
```bash
# Run with debug output
./gradlew test --info

# Check specific test
./gradlew test --tests "ClassName.methodName"
```

### Coverage Failures
```bash
# Generate detailed report
./gradlew jacocoTestReport

# Open HTML report
open backend/build/reports/jacoco/test/html/index.html
```

## Current Development Focus

### Phase 9: External System Integration
- JIRA API integration for project synchronization
- Scheduled JQL query execution (hourly)
- Automatic project information updates
- Sync history and monitoring

## Important Notes

### Working Directory Context
- Backend commands should be run from the `backend/` directory
- Frontend commands should be run from the `frontend/` directory
- Docker commands should be run from the project root

### Test Execution Priority
- Always run tests after making changes
- Coverage verification (`./gradlew check`) is mandatory before considering a task complete
- Tests must pass with 80% minimum coverage

### Database Schema Updates
- Use Flyway migrations for all schema changes
- Migration files in `backend/src/main/resources/db/migration/`
- Follow naming convention: `V{number}__{description}.sql`