# Database Design

## Executive Summary

OpenDataAnalytics.AI uses SQLite as the default database with support for PostgreSQL in enterprise deployments. The database design is normalized, scalable, and supports complex analytics queries with audit trails and version control.

## Database Architecture

### Supported Databases

| Database | Use Case | Support |
|----------|----------|---------|
| **SQLite** | Desktop, single-user | ✅ Default |
| **PostgreSQL** | Self-hosted, cloud | ✅ Recommended |
| **MySQL** | Alternative SQL | ⏳ Planned |
| **Oracle** | Enterprise | ⏳ Planned |

### Connection Strategy

```python
# Desktop/Local
DATABASE_URL = "sqlite:///OpenDataAnalytics.AI.db"

# Self-hosted
DATABASE_URL = "postgresql://user:password@localhost/OpenDataAnalytics.AI"

# Cloud
DATABASE_URL = "postgresql://user:password@cloud-host/OpenDataAnalytics.AI"
```

## Database Schema Overview

### Core Tables

```
┌─────────────────────────────────────────────────┐
│                  Users                          │
│  id | email | password | created_at | ...      │
└────────────────┬────────────────────────────────┘
                 │
                 ├─→ Projects
                 ├─→ API Keys
                 └─→ Audit Logs
                 
┌─────────────────────────────────────────────────┐
│              Projects                           │
│  id | name | owner_id | created_at | ...       │
└────────────────┬────────────────────────────────┘
                 │
                 ├─→ Datasets
                 ├─→ Analyses
                 ├─→ Visualizations
                 ├─→ Dashboards
                 ├─→ Reports
                 └─→ Collaborators
                 
┌─────────────────────────────────────────────────┐
│             Datasets                            │
│  id | name | project_id | data | ...           │
└────────────────┬────────────────────────────────┘
                 │
                 ├─→ Dataset Columns
                 ├─→ Dataset Profile
                 ├─→ Data Quality Issues
                 └─→ Data Versions
```

## Table Definitions

### 1. Users Table

```sql
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    avatar_url VARCHAR(500),
    role VARCHAR(50) DEFAULT 'user',
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    email_verified BOOLEAN DEFAULT FALSE,
    mfa_enabled BOOLEAN DEFAULT FALSE,
    deleted_at TIMESTAMP NULL
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_users_status ON users(status);
```

**Fields**:
- `id`: UUID primary key
- `email`: Unique email address
- `password_hash`: Bcrypt hashed password
- `role`: User role (user, admin, analyst)
- `status`: Account status (active, inactive, suspended)
- `mfa_enabled`: Two-factor authentication flag
- `deleted_at`: Soft delete support

### 2. Projects Table

```sql
CREATE TABLE projects (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_id VARCHAR(36) NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    visibility VARCHAR(50) DEFAULT 'private',
    tags JSON,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    archived_at TIMESTAMP NULL,
    FOREIGN KEY (owner_id) REFERENCES users(id)
);

CREATE INDEX idx_projects_owner_id ON projects(owner_id);
CREATE INDEX idx_projects_created_at ON projects(created_at);
CREATE INDEX idx_projects_status ON projects(status);
```

### 3. Datasets Table

```sql
CREATE TABLE datasets (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    file_path VARCHAR(500),
    file_size BIGINT,
    file_format VARCHAR(50),
    row_count BIGINT,
    column_count INT,
    quality_score FLOAT,
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (created_by) REFERENCES users(id)
);

CREATE INDEX idx_datasets_project_id ON datasets(project_id);
CREATE INDEX idx_datasets_status ON datasets(status);
CREATE INDEX idx_datasets_created_at ON datasets(created_at);
```

### 4. Dataset Columns Table

```sql
CREATE TABLE dataset_columns (
    id VARCHAR(36) PRIMARY KEY,
    dataset_id VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    data_type VARCHAR(50),
    position INT,
    null_count INT DEFAULT 0,
    unique_count INT,
    min_value VARCHAR(100),
    max_value VARCHAR(100),
    mean_value FLOAT,
    median_value FLOAT,
    std_dev FLOAT,
    quality_score FLOAT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (dataset_id) REFERENCES datasets(id)
);

CREATE INDEX idx_dataset_columns_dataset_id ON dataset_columns(dataset_id);
```

### 5. Analyses Table

```sql
CREATE TABLE analyses (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    dataset_id VARCHAR(36),
    name VARCHAR(255) NOT NULL,
    query TEXT,
    query_type VARCHAR(50),
    status VARCHAR(50) DEFAULT 'draft',
    result_count INT,
    execution_time INT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (dataset_id) REFERENCES datasets(id),
    FOREIGN KEY (created_by) REFERENCES users(id)
);

CREATE INDEX idx_analyses_project_id ON analyses(project_id);
CREATE INDEX idx_analyses_status ON analyses(status);
```

### 6. Visualizations Table

```sql
CREATE TABLE visualizations (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    analysis_id VARCHAR(36),
    name VARCHAR(255) NOT NULL,
    chart_type VARCHAR(50),
    x_axis VARCHAR(255),
    y_axis VARCHAR(255),
    color_by VARCHAR(255),
    config JSON,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (analysis_id) REFERENCES analyses(id),
    FOREIGN KEY (created_by) REFERENCES users(id)
);

CREATE INDEX idx_visualizations_project_id ON visualizations(project_id);
CREATE INDEX idx_visualizations_analysis_id ON visualizations(analysis_id);
```

### 7. Dashboards Table

```sql
CREATE TABLE dashboards (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    layout JSON,
    theme VARCHAR(50),
    refresh_interval INT,
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (created_by) REFERENCES users(id)
);

CREATE INDEX idx_dashboards_project_id ON dashboards(project_id);
```

### 8. Dashboard Widgets Table

```sql
CREATE TABLE dashboard_widgets (
    id VARCHAR(36) PRIMARY KEY,
    dashboard_id VARCHAR(36) NOT NULL,
    visualization_id VARCHAR(36),
    position_x INT,
    position_y INT,
    width INT,
    height INT,
    config JSON,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (dashboard_id) REFERENCES dashboards(id),
    FOREIGN KEY (visualization_id) REFERENCES visualizations(id)
);

CREATE INDEX idx_dashboard_widgets_dashboard_id ON dashboard_widgets(dashboard_id);
```

### 9. Reports Table

```sql
CREATE TABLE reports (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    title VARCHAR(255),
    description TEXT,
    visualizations JSON,
    insights JSON,
    status VARCHAR(50) DEFAULT 'draft',
    generated_at TIMESTAMP,
    scheduled_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (created_by) REFERENCES users(id)
);

CREATE INDEX idx_reports_project_id ON reports(project_id);
CREATE INDEX idx_reports_status ON reports(status);
```

### 10. Collaborators Table

```sql
CREATE TABLE collaborators (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'viewer',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    UNIQUE(project_id, user_id)
);

CREATE INDEX idx_collaborators_project_id ON collaborators(project_id);
CREATE INDEX idx_collaborators_user_id ON collaborators(user_id);
```

### 11. Comments Table

```sql
CREATE TABLE comments (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36) NOT NULL,
    entity_type VARCHAR(50),
    entity_id VARCHAR(36),
    text TEXT NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    resolved BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_comments_project_id ON comments(project_id);
CREATE INDEX idx_comments_entity ON comments(entity_type, entity_id);
```

### 12. Audit Logs Table

```sql
CREATE TABLE audit_logs (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36),
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50),
    entity_id VARCHAR(36),
    old_value TEXT,
    new_value TEXT,
    ip_address VARCHAR(50),
    user_agent VARCHAR(500),
    status VARCHAR(50),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
```

### 13. API Keys Table

```sql
CREATE TABLE api_keys (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    key_hash VARCHAR(255) NOT NULL UNIQUE,
    prefix VARCHAR(20),
    last_used TIMESTAMP,
    expires_at TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_api_keys_user_id ON api_keys(user_id);
CREATE INDEX idx_api_keys_key_hash ON api_keys(key_hash);
```

### 14. Data Quality Issues Table

```sql
CREATE TABLE data_quality_issues (
    id VARCHAR(36) PRIMARY KEY,
    dataset_id VARCHAR(36) NOT NULL,
    column_id VARCHAR(36),
    issue_type VARCHAR(50),
    severity VARCHAR(50),
    description TEXT,
    affected_rows INT,
    resolved BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    resolved_at TIMESTAMP,
    FOREIGN KEY (dataset_id) REFERENCES datasets(id),
    FOREIGN KEY (column_id) REFERENCES dataset_columns(id)
);

CREATE INDEX idx_dqi_dataset_id ON data_quality_issues(dataset_id);
CREATE INDEX idx_dqi_resolved ON data_quality_issues(resolved);
```

### 15. Dataset Versions Table

```sql
CREATE TABLE dataset_versions (
    id VARCHAR(36) PRIMARY KEY,
    dataset_id VARCHAR(36) NOT NULL,
    version_number INT,
    file_path VARCHAR(500),
    file_size BIGINT,
    changes JSON,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    FOREIGN KEY (dataset_id) REFERENCES datasets(id),
    FOREIGN KEY (created_by) REFERENCES users(id),
    UNIQUE(dataset_id, version_number)
);

CREATE INDEX idx_dataset_versions_dataset_id ON dataset_versions(dataset_id);
```

## Data Types

### SQLite Types
- `TEXT`: String data
- `INTEGER`: Whole numbers
- `REAL`: Floating-point numbers
- `BLOB`: Binary data
- `NULL`: Null value

### PostgreSQL Types
- `VARCHAR(n)`: Variable-length strings
- `INT`: 32-bit integers
- `BIGINT`: 64-bit integers
- `FLOAT`: Floating-point numbers
- `JSON`: JSON data
- `TIMESTAMP`: Date and time
- `BOOLEAN`: True/False
- `UUID`: Universally unique identifier

## Indexing Strategy

### Indexes for Performance

```sql
-- Foreign key indexes
CREATE INDEX idx_fk_project_owner ON projects(owner_id);
CREATE INDEX idx_fk_dataset_project ON datasets(project_id);

-- Timestamp indexes (for sorting, filtering)
CREATE INDEX idx_created_at ON projects(created_at);
CREATE INDEX idx_updated_at ON datasets(updated_at);

-- Status indexes (common filters)
CREATE INDEX idx_project_status ON projects(status);
CREATE INDEX idx_dataset_status ON datasets(status);

-- Audit trail indexes
CREATE INDEX idx_audit_user_time ON audit_logs(user_id, created_at);

-- Search indexes
CREATE INDEX idx_project_name ON projects(name);
CREATE INDEX idx_dataset_name ON datasets(name);
```

### Query Optimization

```sql
-- Composite indexes for common queries
CREATE INDEX idx_dataset_project_status 
    ON datasets(project_id, status);

CREATE INDEX idx_collaborator_project_user 
    ON collaborators(project_id, user_id);

CREATE INDEX idx_audit_entity_time 
    ON audit_logs(entity_type, entity_id, created_at);
```

## Relationships

### One-to-Many
- User → Projects (owner)
- Project → Datasets
- Project → Analyses
- Dataset → Dataset Columns

### Many-to-Many
- Project ↔ Users (Collaborators)
- Analysis → Visualizations
- Dashboard ↔ Visualizations (Widgets)

### Audit Trail
- Every change logged to audit_logs
- User action tracking
- Timestamp recording
- Change details stored

## Data Retention Policy

### Default Retention
- Active data: Indefinite
- Audit logs: 7 years
- Deleted data: 30-day recovery window
- Soft deletes: `deleted_at` timestamp

### GDPR Compliance
- Right to be forgotten: Delete user data
- Data portability: Export user data
- Access control: User data filtering
- Encryption: Sensitive data protection

## Backup Strategy

### Backup Schedule
- Daily: Full database backup
- Hourly: Incremental backup
- Real-time: Replication (production)

### Recovery
- Point-in-time recovery: 30 days
- Full backup recovery: 24 hours
- Incremental recovery: < 1 hour
- Test recovery: Monthly

## Migration Strategy

### Database Migrations

```python
# Using Alembic for migrations
alembic init migrations
alembic revision --autogenerate -m "Add audit table"
alembic upgrade head
```

### Version Control
- Track all schema changes
- Reversible migrations
- Tested before production
- Documentation included

## Performance Considerations

### Query Optimization
- Use appropriate indexes
- Avoid N+1 queries
- Batch operations
- Connection pooling

### Scaling
- Vertical scaling: Upgrade hardware
- Horizontal scaling: Read replicas
- Partitioning: Time-based partitioning
- Caching: Redis cache layer

---

## Related Documents
- [ER Diagram](ER_Diagram.md)
- [Table Definitions](Table_Definitions.md)
- [Indexes](Indexes.md)

**Last Updated**: December 2024
