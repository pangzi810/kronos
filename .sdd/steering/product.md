# Product Overview

## Product Description
Development Hour Management Tool (製品開発工数管理ツール) - A comprehensive web application system for PMOs and developers to efficiently manage and track project development hours, resource allocation, and work approval workflows in software development organizations.

## Core Features

### Project Management
- **Project Creation and Configuration**: Manage software development projects with start/end dates, status tracking, and budget allocation
- **Direct Project Access**: All users can access all projects directly without assignment requirements
- **Work Hour Tracking**: Record daily work hours across multiple projects with category-based time allocation

### Work Hour Management
- **Multi-Project Time Entry**: Developers can record hours across multiple projects
- **Category-Based Tracking**: Allocate hours to specific work categories (development, testing, meetings, etc.)
- **Missing Hours Indicator**: Visual indicators for dates with incomplete work hour entries
- **Input Restriction**: Prevent modification of approved work hours with visual feedback

### Approval Workflow
- **Hierarchical Approval System**: Approvers review and approve work hours
- **Daily Approval Process**: Day-by-day approval capability for fine-grained control
- **Batch Approval**: Approve multiple days/users simultaneously for efficiency
- **Rejection with Feedback**: Reject entries with comments for correction

### Team Management
- **Approver Relationships**: Hierarchical approval structure support
- **Active Developer Tracking**: Monitor currently active developers across projects
- **Team Organization**: Support for hierarchical team structures

### Administrative Features
- **User Management**: Create, update, deactivate users with role assignments
- **Admin User Access**: Special administrative endpoints for system management
- **Deleted User Handling**: Track and manage deactivated user data
- **Audit Logging**: Comprehensive audit trail for compliance and tracking

### Authentication & Security
- **Okta SSO Integration**: ✅ **Fully Implemented** - Complete OAuth2/OpenID Connect single sign-on
  - OAuth2 Resource Server configuration for token validation
  - Scope-based authorization mapping to application roles
  - Automatic user synchronization from Okta to local database
  - Secure token validation with Okta issuer
  - User status management (ACTIVE, INACTIVE, SUSPENDED)
  - Last login tracking for user activity monitoring
  - Password-less authentication (local passwords removed)
  - Unique Okta user mapping with database constraints
- **Access Control**: ✅ **Implemented** - User-based permissions and administrative functions
- **Audit Logging**: System-wide activity tracking and compliance

### External Integration
- **JIRA Project Sync**: ✅ **Fully Implemented** - Complete automatic synchronization with JIRA for project information
  - Distributed scheduler with ShedLock for concurrent execution prevention
  - Comprehensive demo data and test infrastructure (V25 migration)
  - Application service with 100% test success rate
  - Security-integrated REST API endpoints
  - Complete frontend implementation with Vue 3 components
- **JQL Query Management**: ✅ **Implemented** - Configure multiple JQL queries for targeted data retrieval
- **Velocity Template Transformation**: ✅ **Implemented** - Transform JIRA responses using customizable templates
- **Scheduled Synchronization**: ✅ **Implemented** - Hourly automatic sync with distributed locking
- **JIRA Frontend Interface**: ✅ **Implemented** - Full Vue 3 UI for JIRA sync management and monitoring

## Target Use Cases

### Primary Use Cases
1. **Software Development Teams**: Track development effort across multiple concurrent projects
2. **PMO Organizations**: Monitor resource allocation and project progress across teams
3. **Consulting Firms**: Track billable hours for client projects
4. **IT Departments**: Manage internal project resources and time allocation

### Specific Scenarios
- **Multi-Project Environment**: Developers working on 3-5 concurrent projects
- **Matrix Organizations**: Teams working across multiple departments/projects
- **Compliance Requirements**: Organizations needing detailed time tracking audit trails
- **Remote Teams**: Distributed teams requiring centralized hour tracking

## Key Value Propositions

### Business Value
- **Resource Optimization**: Identify over/under-allocated resources across projects
- **Cost Control**: Track actual vs. planned hours for budget management
- **Compliance**: Maintain auditable records of work hours and approvals
- **Decision Support**: Data-driven insights for project planning and staffing

### Technical Excellence
- **Domain-Driven Design**: Clean architecture ensuring maintainability and scalability
- **Test-First Development**: 80% minimum test coverage for reliability
- **Performance**: Optimized queries and caching for fast response times
- **Security**: Enterprise-grade Okta SSO authentication with role-based access control and activity tracking

### User Experience
- **Intuitive Interface**: Vue 3 + Vuetify for modern, responsive UI
- **Multi-Language Support**: Japanese and Chinese language support
- **Real-Time Feedback**: Toast notifications for immediate user feedback
- **Efficient Workflows**: Batch operations and keyboard shortcuts for power users

### Operational Benefits
- **Easy Deployment**: Docker containerization for simplified deployment
- **Automated Migrations**: Flyway database migrations for seamless updates
- **Comprehensive Monitoring**: Logging and metrics for operational visibility
- **External Integration Ready**: Extensible architecture for third-party integrations

## Product Maturity

### Current Status
- **Backend**: Fully implemented with comprehensive test coverage
- **Frontend**: Substantial implementation with core features complete
  - Build optimization with code splitting for improved performance
  - Enhanced i18n support with latest Vue i18n v12
  - Refactored components for better maintainability (WorkHoursTable)
  - Client-side caching for performance (multi-month date statuses, user authentication)
  - JST timezone utilities for consistent date handling
- **Database**: Consolidated migration system (V1__init.sql) with all schema definitions
- **API**: RESTful API with OpenAPI documentation
- **Testing**: Unit, integration, and performance tests in place
- **JIRA Integration**: Fully implemented with complete frontend and backend integration
- **Okta SSO Integration**: Fully implemented with complete OAuth2 authentication, user synchronization, and password-less operation

### Upcoming Enhancements
- **GitHub Commit Integration**: ⏳ **In Development** - Automatic synchronization of GitHub commit logs with work hour records
  - GitHub App authentication for secure repository access
  - Automatic commit-to-work-record mapping by date and user
  - Commit activity visualization in work hour tracking interface
  - Project-level commit reporting and productivity analytics
- **Advanced Analytics**: Enhanced reporting and visualization capabilities
- **Mobile Support**: Responsive design optimization for mobile devices
- **Additional Integrations**: Slack notifications, calendar synchronization
- **Enhanced JIRA Features**: Advanced sync filtering and custom field mapping

## Success Metrics
- **User Adoption**: Number of active users and projects managed
- **Data Quality**: Percentage of complete and approved time entries
- **Performance**: Response time under 200ms for 95% of requests
- **Reliability**: 99.9% uptime with zero data loss
- **User Satisfaction**: Reduced time for hour entry and approval processes
