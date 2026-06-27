# Testing Strategy

## Executive Summary

OpenDataAnalytics.AI implements a comprehensive testing strategy across unit, integration, AI, performance, and security testing to ensure reliability, quality, and safety. Target coverage is 85%+ for code and 100% for critical paths.

## Testing Philosophy

### Core Principles

1. **Shift Left**: Test early and often
2. **Automation**: Automate all repeatable tests
3. **Comprehensive**: Cover all layers and critical paths
4. **Realistic**: Test with production-like data
5. **Continuous**: Test throughout development
6. **Visible**: Make test results transparent

### Testing Pyramid

```
        ┌─────────────────┐
        │      UI Tests   │  (5%)
        │  (E2E, Manual)  │
        ├────────┬────────┤
        │ Integration Tests│ (20%)
        │  (API, DB, Sys) │
        ├─────┬──────┬────┤
        │   Unit Tests   │  (75%)
        │ (Functions,    │
        │  Components)   │
        └─────────────────┘
```

## Test Types

### 1. Unit Testing

**Purpose**: Test individual functions and components in isolation

**Tools**:
- **Frontend**: Vitest, React Testing Library
- **Backend**: pytest, unittest
- **Coverage Target**: 80%+

#### Frontend Unit Tests

```typescript
// Example: Component unit test
import { render, screen } from '@testing-library/react';
import { DatasetManager } from './DatasetManager';

describe('DatasetManager', () => {
  it('should display dataset list', () => {
    render(<DatasetManager />);
    expect(screen.getByText('My Datasets')).toBeInTheDocument();
  });

  it('should upload dataset', async () => {
    render(<DatasetManager />);
    const uploadBtn = screen.getByText('Upload');
    fireEvent.click(uploadBtn);
    // Verify upload flow
  });
});
```

**Execution**:
```bash
npm run test              # Run all tests
npm run test:watch       # Watch mode
npm run test:coverage    # With coverage
```

#### Backend Unit Tests

```python
# Example: Service unit test
import pytest
from backend.app.services import DatasetService

@pytest.fixture
def dataset_service():
    return DatasetService()

def test_validate_dataset_format(dataset_service):
    """Test dataset format validation."""
    result = dataset_service.validate_format("data.csv")
    assert result is True
    
    with pytest.raises(ValueError):
        dataset_service.validate_format("data.xyz")

def test_detect_column_types(dataset_service):
    """Test column type detection."""
    data = {"id": [1, 2, 3], "name": ["a", "b", "c"]}
    types = dataset_service.detect_types(data)
    assert types["id"] == "integer"
    assert types["name"] == "string"
```

**Execution**:
```bash
pytest                   # Run all tests
pytest -v               # Verbose
pytest --cov           # With coverage
pytest -k test_name    # Specific test
```

### 2. Integration Testing

**Purpose**: Test component interactions and system integration

**Tools**:
- **API Testing**: pytest, requests
- **Database Testing**: pytest fixtures
- **System Testing**: Custom test harnesses

#### API Integration Tests

```python
@pytest.fixture
def client():
    """Test client for API."""
    return TestClient(app)

def test_create_project_workflow(client):
    """Test complete project creation workflow."""
    # Create project
    response = client.post("/api/v1/projects", json={
        "name": "Test Project",
        "description": "Test description"
    })
    assert response.status_code == 201
    project_id = response.json()["data"]["id"]
    
    # Upload dataset
    with open("test_data.csv", "rb") as f:
        response = client.post(
            f"/api/v1/projects/{project_id}/datasets",
            files={"file": f}
        )
    assert response.status_code == 201
    dataset_id = response.json()["data"]["id"]
    
    # Profile dataset
    response = client.post(
        f"/api/v1/projects/{project_id}/datasets/{dataset_id}/profile"
    )
    assert response.status_code == 200
    assert "quality_score" in response.json()["data"]
```

#### Database Integration Tests

```python
def test_dataset_versioning(db_session):
    """Test dataset version control."""
    # Create dataset
    dataset = Dataset(name="test", project_id="proj_1")
    db_session.add(dataset)
    db_session.commit()
    
    # Create version
    version = DatasetVersion(
        dataset_id=dataset.id,
        version_number=1,
        file_path="/path/to/v1.csv"
    )
    db_session.add(version)
    db_session.commit()
    
    # Verify version
    versions = db_session.query(DatasetVersion).filter_by(
        dataset_id=dataset.id
    ).all()
    assert len(versions) == 1
    assert versions[0].version_number == 1
```

**Execution**:
```bash
pytest tests/integration/          # Integration tests
pytest tests/integration/test_api  # API tests
pytest tests/integration/test_db   # Database tests
```

### 3. AI Testing

**Purpose**: Test AI agents, prompts, and LLM integration

**Tools**:
- **Prompt Testing**: Custom validators
- **Output Validation**: Unit tests
- **Confidence Scoring**: Regression tests

#### AI Agent Testing

```python
class TestDataProfilingAI:
    """Test Data Profiling AI Agent."""
    
    @pytest.fixture
    def ai_agent(self):
        return DataProfilingAI(model="claude-3-haiku")
    
    def test_profile_generation(self, ai_agent):
        """Test basic profiling."""
        dataset = {
            "columns": ["id", "name", "age"],
            "data": [[1, "Alice", 30], [2, "Bob", 25]]
        }
        
        profile = ai_agent.profile(dataset)
        
        assert "quality_score" in profile
        assert "issues" in profile
        assert 0 <= profile["quality_score"] <= 1
    
    def test_confidence_scoring(self, ai_agent):
        """Test confidence scoring."""
        output = {"quality_score": 0.95, "issues": []}
        confidence = ai_agent.score_confidence(output)
        
        assert 0 <= confidence <= 1
        assert confidence > 0.8  # High confidence
    
    def test_error_handling(self, ai_agent):
        """Test error handling."""
        invalid_data = {}
        
        with pytest.raises(ValueError):
            ai_agent.profile(invalid_data)
    
    def test_output_validation(self, ai_agent):
        """Test output schema validation."""
        output = {
            "quality_score": 0.9,
            "issues": [
                {"column": "age", "type": "missing", "severity": "high"}
            ],
            "recommendations": ["Fill missing values"]
        }
        
        is_valid = ai_agent.validate_output(output)
        assert is_valid is True
```

#### Prompt Testing

```python
class TestPromptEngineering:
    """Test prompt engineering."""
    
    def test_prompt_generation(self):
        """Test prompt generation."""
        context = {
            "dataset_name": "sales.csv",
            "rows": 1000,
            "columns": ["date", "revenue", "product"]
        }
        
        prompt = generate_profile_prompt(context)
        
        assert "sales.csv" in prompt
        assert "1000" in prompt
        assert "date" in prompt
    
    def test_few_shot_examples(self):
        """Test few-shot learning."""
        prompt = generate_query_prompt_with_examples()
        
        # Should include examples
        assert "Example 1:" in prompt
        assert "SELECT" in prompt
```

**Execution**:
```bash
pytest tests/ai/                    # AI tests
pytest tests/ai/test_agents        # Agent tests
pytest tests/ai/test_prompts       # Prompt tests
pytest tests/ai/ -v --tb=short     # Detailed output
```

### 4. Performance Testing

**Purpose**: Verify performance meets SLAs

**Tools**:
- **Load Testing**: Locust, Apache JMeter
- **Profiling**: Python cProfile, Chrome DevTools
- **Benchmarking**: pytest-benchmark

#### Load Testing

```python
from locust import HttpUser, task, between

class AnalyticsUser(HttpUser):
    wait_time = between(1, 3)
    
    @task(3)
    def list_projects(self):
        """List projects - 3:1 ratio."""
        self.client.get("/api/v1/projects")
    
    @task(1)
    def create_project(self):
        """Create project."""
        self.client.post("/api/v1/projects", json={
            "name": f"Project-{random.randint(1, 1000)}"
        })
    
    def on_start(self):
        """Setup: login."""
        self.client.post("/api/v1/auth/token", json={
            "username": "test@example.com",
            "password": "password"
        })
```

**Execution**:
```bash
locust -f tests/performance/locustfile.py \
  --host=http://localhost:8000 \
  --users=1000 \
  --spawn-rate=100 \
  --run-time=5m
```

#### Performance Benchmarks

```python
@pytest.mark.benchmark
def test_query_performance(benchmark):
    """Benchmark query execution."""
    def execute_query():
        return database.query("SELECT * FROM sales")
    
    result = benchmark(execute_query)
    assert result.duration < 5.0  # 5 seconds max
```

### 5. Security Testing

**Purpose**: Identify security vulnerabilities

**Tools**:
- **Static Analysis**: Bandit, ESLint security
- **Dependency Scanning**: pip-audit, npm audit
- **Dynamic Testing**: OWASP ZAP, penetration testing

#### Security Unit Tests

```python
def test_password_hashing():
    """Verify passwords are hashed."""
    password = "SecurePassword123!"
    hashed = hash_password(password)
    
    # Password should not be in plaintext
    assert password not in hashed
    assert len(hashed) > 20
    
    # Should verify correctly
    assert verify_password(password, hashed)
    assert not verify_password("WrongPassword", hashed)

def test_sql_injection_prevention():
    """Verify SQL injection prevention."""
    malicious_input = "'; DROP TABLE users; --"
    
    query = build_query("SELECT * FROM users WHERE name = ?", [malicious_input])
    # Should safely escape the input
    assert "DROP TABLE" not in execute(query)

def test_csrf_token_validation():
    """Verify CSRF protection."""
    response = client.post("/api/v1/projects", json={"name": "test"})
    
    # Should reject requests without CSRF token
    assert response.status_code == 403
    
    # Should accept with valid token
    token = get_csrf_token()
    response = client.post(
        "/api/v1/projects",
        json={"name": "test"},
        headers={"X-CSRF-Token": token}
    )
    assert response.status_code == 201
```

### 6. User Acceptance Testing (UAT)

**Purpose**: Verify product meets user requirements

**Process**:
1. Define test scenarios
2. Create test data
3. Perform manual testing
4. Collect user feedback
5. Document issues
6. Verify fixes

**Test Scenarios**:
- Create project → Upload data → Analyze → Export report
- Collaborate on analysis
- Share dashboard with team
- Real-time updates

## Continuous Integration

### CI/CD Pipeline

```yaml
# GitHub Actions example
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.10
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run unit tests
        run: pytest tests/unit --cov
      
      - name: Run integration tests
        run: pytest tests/integration
      
      - name: Security scan
        run: bandit -r backend/
      
      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

## Test Coverage Goals

| Component | Target | Current |
|-----------|--------|---------|
| **Backend API** | 85% | 82% |
| **Frontend Components** | 80% | 78% |
| **AI Agents** | 90% | 88% |
| **Database Layer** | 85% | 84% |
| **Utils/Helpers** | 90% | 89% |
| **Overall** | 85% | 83% |

## Test Environment

### Local Development
```bash
# Setup
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

# Run tests
npm run test          # Frontend
pytest                # Backend
```

### Staging Environment
- Automated deployment
- Full test suite runs
- Performance testing
- Security scanning

### Production
- Canary deployments
- Smoke tests
- Monitoring
- Rollback capability

## Test Data Management

### Test Data Strategy
- Isolated test databases
- Seed data for consistency
- Realistic data volumes
- Anonymized production data (if used)

### Data Cleanup
```python
@pytest.fixture(autouse=True)
def cleanup():
    """Cleanup after each test."""
    yield
    # Cleanup code
    db.session.rollback()
    clear_cache()
```

## Regression Testing

### Automated Regression Suite
- Critical user paths
- Previously fixed bugs
- Core features
- Integration points

**Execution**:
```bash
pytest tests/regression/         # Run regression tests
pytest tests/regression/ --tb=short  # Quick failures
```

## Bug Reporting

### Bug Triage
1. Reproduce issue
2. Assign severity
3. Identify component
4. Create test case
5. Fix and verify

### Severity Levels
| Level | SLA | Example |
|-------|-----|---------|
| **Critical** | 4 hours | Data loss, security breach |
| **High** | 24 hours | Feature broken, major bug |
| **Medium** | 1 week | Minor feature issue |
| **Low** | 2 weeks | UI polish, documentation |

---

## Related Documents
- [Unit Testing](Unit_Testing.md)
- [Integration Testing](Integration_Testing.md)
- [AI Testing](AI_Testing.md)
- [Performance Testing](Performance_Testing.md)
- [Security Testing](Security_Testing.md)

**Last Updated**: December 2024
