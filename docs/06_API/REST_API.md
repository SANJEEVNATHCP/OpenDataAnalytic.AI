# REST API Documentation

## Overview

The OpenDataAnalytics.AI REST API provides programmatic access to all platform functionality. The API is designed to be RESTful, following standard HTTP conventions and returning JSON responses.

## API Fundamentals

### Base URL

- **Development**: `http://localhost:8000/api/v1`
- **Production**: `https://api.OpenDataAnalytics.AI.dev/v1`
- **Self-Hosted**: `https://your-domain.com/api/v1`

### API Version

Current version: **v1**

- Versioning strategy: URL-based (`/api/v1`, `/api/v2`, etc.)
- Backward compatibility maintained for 2 major versions
- Deprecation notice provided 6 months in advance

### Rate Limiting

```
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9999
X-RateLimit-Reset: 1640995200
```

- **Limit**: 10,000 requests per hour per API key
- **Burst**: 100 requests per 10 seconds
- **Retry**: Use exponential backoff

## Authentication

### Authentication Methods

#### 1. Bearer Token (JWT)

```bash
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  https://api.OpenDataAnalytics.AI.dev/v1/projects
```

**Obtaining a Token**:
```bash
POST /api/v1/auth/token
Content-Type: application/json

{
  "username": "user@example.com",
  "password": "password"
}

Response:
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

#### 2. API Key

```bash
curl -H "X-API-Key: YOUR_API_KEY" \
  https://api.OpenDataAnalytics.AI.dev/v1/projects
```

#### 3. OAuth2

```bash
Authorization: Bearer [oauth2_token]
```

### Token Management

- **Token Expiration**: 1 hour for access tokens
- **Refresh Token**: 30 days
- **Refresh Endpoint**: `POST /api/v1/auth/refresh`

```bash
POST /api/v1/auth/refresh
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Response:
{
  "access_token": "...",
  "expires_in": 3600
}
```

## Request/Response Format

### Request Headers

```
Content-Type: application/json
Authorization: Bearer [token]
X-Request-ID: [optional UUID for tracing]
User-Agent: [optional client info]
```

### Response Headers

```
Content-Type: application/json
X-Request-ID: [UUID for tracing]
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9999
X-RateLimit-Reset: [Unix timestamp]
```

### Success Response (2xx)

```json
{
  "status": "success",
  "data": {
    "id": "proj_123",
    "name": "My Project",
    "created_at": "2024-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "req_abc123",
    "timestamp": "2024-01-15T10:30:05Z"
  }
}
```

### Error Response

```json
{
  "status": "error",
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request was invalid",
    "details": {
      "field": "name",
      "reason": "Name is required"
    }
  },
  "meta": {
    "request_id": "req_abc123",
    "timestamp": "2024-01-15T10:30:05Z"
  }
}
```

## API Endpoints

### Projects

#### List Projects
```
GET /api/v1/projects
```

**Query Parameters**:
- `limit`: Maximum results (default: 20, max: 100)
- `offset`: Pagination offset (default: 0)
- `sort`: Sort field (default: created_at)
- `order`: asc or desc (default: desc)

**Response**:
```json
{
  "status": "success",
  "data": [
    {
      "id": "proj_123",
      "name": "Sales Analysis",
      "description": "Q4 sales data analysis",
      "owner_id": "user_456",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-20T14:15:00Z"
    }
  ],
  "meta": {
    "total": 42,
    "limit": 20,
    "offset": 0
  }
}
```

#### Create Project
```
POST /api/v1/projects
Content-Type: application/json

{
  "name": "New Analysis",
  "description": "Project description"
}
```

**Response**: 201 Created

#### Get Project
```
GET /api/v1/projects/{project_id}
```

#### Update Project
```
PUT /api/v1/projects/{project_id}
```

#### Delete Project
```
DELETE /api/v1/projects/{project_id}
```

### Datasets

#### List Datasets
```
GET /api/v1/projects/{project_id}/datasets
```

#### Upload Dataset
```
POST /api/v1/projects/{project_id}/datasets
Content-Type: multipart/form-data

{
  "file": <file>,
  "name": "dataset_name",
  "description": "optional description"
}
```

**Supported Formats**:
- CSV
- Excel (.xlsx, .xls)
- JSON
- Parquet
- SQLite

#### Get Dataset
```
GET /api/v1/projects/{project_id}/datasets/{dataset_id}
```

#### Profile Dataset
```
POST /api/v1/projects/{project_id}/datasets/{dataset_id}/profile
```

**Response**:
```json
{
  "status": "success",
  "data": {
    "dataset_id": "ds_123",
    "row_count": 10000,
    "column_count": 15,
    "quality_score": 0.92,
    "columns": [
      {
        "name": "customer_id",
        "type": "integer",
        "null_count": 0,
        "unique_count": 9500
      }
    ],
    "issues": [
      {
        "column": "email",
        "type": "invalid_format",
        "count": 50
      }
    ]
  }
}
```

### Analysis

#### Create Analysis
```
POST /api/v1/projects/{project_id}/analyses
Content-Type: application/json

{
  "name": "Q4 Sales Trends",
  "dataset_id": "ds_123",
  "query": "SELECT * FROM sales WHERE date > '2024-10-01'"
}
```

#### Natural Language Query
```
POST /api/v1/projects/{project_id}/query
Content-Type: application/json

{
  "question": "What are the top 10 products by revenue?",
  "dataset_id": "ds_123"
}
```

**Response**:
```json
{
  "status": "success",
  "data": {
    "query_id": "q_789",
    "generated_sql": "SELECT product_name, SUM(revenue) as total_revenue FROM sales GROUP BY product_name ORDER BY total_revenue DESC LIMIT 10",
    "results": [
      {
        "product_name": "Product A",
        "total_revenue": 125000
      }
    ],
    "explanation": "This query aggregates revenue by product..."
  }
}
```

#### Get Visualization Recommendations
```
POST /api/v1/projects/{project_id}/visualizations/recommend
Content-Type: application/json

{
  "dataset_id": "ds_123",
  "columns": ["date", "revenue", "category"],
  "analysis_type": "trend"
}
```

**Response**:
```json
{
  "status": "success",
  "data": {
    "recommendations": [
      {
        "type": "line",
        "x_axis": "date",
        "y_axis": "revenue",
        "color": "category",
        "confidence": 0.95,
        "reasoning": "Line chart best shows trend over time"
      },
      {
        "type": "bar",
        "x_axis": "category",
        "y_axis": "revenue",
        "confidence": 0.87
      }
    ]
  }
}
```

### Visualizations

#### Create Visualization
```
POST /api/v1/projects/{project_id}/visualizations
Content-Type: application/json

{
  "name": "Revenue Trend",
  "type": "line",
  "query": "SELECT date, revenue FROM sales ORDER BY date",
  "x_axis": "date",
  "y_axis": "revenue"
}
```

#### List Visualizations
```
GET /api/v1/projects/{project_id}/visualizations
```

#### Export Visualization
```
GET /api/v1/projects/{project_id}/visualizations/{viz_id}/export?format=png
```

**Formats**: png, svg, pdf, html

### Dashboards

#### Create Dashboard
```
POST /api/v1/projects/{project_id}/dashboards
```

#### Add Widget
```
POST /api/v1/projects/{project_id}/dashboards/{dashboard_id}/widgets
```

#### Remove Widget
```
DELETE /api/v1/projects/{project_id}/dashboards/{dashboard_id}/widgets/{widget_id}
```

### Reports

#### Generate Report
```
POST /api/v1/projects/{project_id}/reports
Content-Type: application/json

{
  "name": "Q4 Executive Summary",
  "title": "Q4 2024 Business Review",
  "description": "Quarterly business metrics and insights",
  "visualizations": ["viz_123", "viz_456"],
  "insights": ["insight_789"]
}
```

**Response**: 201 Created
```json
{
  "status": "success",
  "data": {
    "report_id": "rpt_123",
    "status": "generating",
    "estimated_time": "2 minutes",
    "download_url": "/api/v1/projects/{project_id}/reports/{report_id}/download"
  }
}
```

#### Get Report Status
```
GET /api/v1/projects/{project_id}/reports/{report_id}
```

#### Download Report
```
GET /api/v1/projects/{project_id}/reports/{report_id}/download?format=pdf
```

### AI Agents

#### Profile Dataset (AI)
```
POST /api/v1/ai/profile
Content-Type: application/json

{
  "dataset_id": "ds_123"
}
```

#### Clean Data (AI)
```
POST /api/v1/ai/clean
Content-Type: application/json

{
  "dataset_id": "ds_123",
  "operations": ["remove_duplicates", "fill_nulls"]
}
```

#### Generate Insights (AI)
```
POST /api/v1/ai/insights
Content-Type: application/json

{
  "dataset_id": "ds_123"
}
```

### Collaboration

#### Share Project
```
POST /api/v1/projects/{project_id}/share
Content-Type: application/json

{
  "email": "colleague@example.com",
  "role": "editor"
}
```

**Roles**: viewer, commenter, editor, owner

#### Add Comment
```
POST /api/v1/projects/{project_id}/comments
Content-Type: application/json

{
  "entity_type": "visualization",
  "entity_id": "viz_123",
  "text": "Great visualization! Consider adding..."
}
```

### Users & Admin

#### Get Current User
```
GET /api/v1/auth/me
```

#### List Users (admin only)
```
GET /api/v1/admin/users
```

#### Create User (admin only)
```
POST /api/v1/admin/users
```

## Error Codes

### Authentication Errors

| Code | Status | Description |
|------|--------|-------------|
| UNAUTHORIZED | 401 | Missing or invalid token |
| FORBIDDEN | 403 | Insufficient permissions |
| INVALID_TOKEN | 401 | Token expired or invalid |
| INVALID_CREDENTIALS | 401 | Wrong username/password |

### Validation Errors

| Code | Status | Description |
|------|--------|-------------|
| INVALID_REQUEST | 400 | Invalid request format |
| MISSING_FIELD | 400 | Required field missing |
| INVALID_VALUE | 400 | Invalid field value |
| DUPLICATE_RECORD | 409 | Record already exists |

### Server Errors

| Code | Status | Description |
|------|--------|-------------|
| INTERNAL_ERROR | 500 | Internal server error |
| SERVICE_UNAVAILABLE | 503 | Service temporarily unavailable |
| TIMEOUT | 504 | Request timeout |

## Pagination

### Query Parameters
```
GET /api/v1/projects?limit=20&offset=40&sort=created_at&order=desc
```

### Response Meta
```json
{
  "meta": {
    "total": 150,
    "limit": 20,
    "offset": 40,
    "has_next": true,
    "has_previous": true
  }
}
```

## Filtering

### Query Parameters
```
GET /api/v1/projects?name=sales&status=active
```

### Operators
```
GET /api/v1/datasets?row_count[gte]=1000&created_at[gte]=2024-01-01
```

**Operators**: `eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `in`, `nin`, `contains`

## Webhooks

### Create Webhook
```
POST /api/v1/webhooks
Content-Type: application/json

{
  "url": "https://your-app.com/webhook",
  "events": ["analysis.completed", "report.generated"],
  "active": true
}
```

### Webhook Events
- `analysis.completed`
- `report.generated`
- `dataset.uploaded`
- `project.created`
- `project.shared`

### Webhook Payload
```json
{
  "event": "analysis.completed",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "project_id": "proj_123",
    "analysis_id": "a_456"
  }
}
```

## Async Operations

### Long-Running Operations

Some operations return immediately with a status:

```json
{
  "status": "success",
  "data": {
    "operation_id": "op_123",
    "status": "processing",
    "estimated_time": 120,
    "result_url": "/api/v1/operations/op_123/result"
  }
}
```

### Check Status
```
GET /api/v1/operations/{operation_id}
```

### Get Result
```
GET /api/v1/operations/{operation_id}/result
```

## SDK & Client Libraries

### Available SDKs
- Python: `pip install OpenDataAnalytics.AI-sdk`
- JavaScript/TypeScript: `npm install @OpenDataAnalytics.AI/sdk`
- Java: `com.OpenDataAnalytics.AI:OpenDataAnalytics.AI-sdk`
- Go: `github.com/OpenDataAnalytics.AI/go-sdk`

### Python Example
```python
from OpenDataAnalytics.AI import OpenDataAnalytics.AI

client = OpenDataAnalytics.AI(api_key="your_api_key")

# List projects
projects = client.projects.list()

# Upload dataset
dataset = client.datasets.upload(
    project_id="proj_123",
    file="data.csv"
)

# Get profile
profile = client.ai.profile(dataset_id="ds_123")
```

## Best Practices

1. **Use Pagination**: Always implement pagination for list endpoints
2. **Error Handling**: Always handle error responses gracefully
3. **Retry Logic**: Implement exponential backoff for retries
4. **Rate Limiting**: Monitor rate limit headers and implement backoff
5. **Caching**: Cache responses when appropriate
6. **API Keys**: Rotate API keys regularly
7. **Logging**: Log important API calls for debugging
8. **Testing**: Test API integration thoroughly

---

## Related Documents
- [Authentication](Authentication.md)
- [Authorization](Authorization.md)
- [Error Codes](Error_Codes.md)
- [Examples](Examples.md)

**Last Updated**: December 2024
