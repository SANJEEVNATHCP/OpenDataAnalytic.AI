# Coding Standards

## Overview

InsightOS maintains high code quality standards across all languages and components. These standards ensure consistency, maintainability, and reliability.

## TypeScript/JavaScript Standards

### Code Style

#### Naming Conventions

```typescript
// Constants: UPPER_SNAKE_CASE
const MAX_RETRIES = 3;
const DEFAULT_TIMEOUT = 5000;

// Variables & Functions: camelCase
const userData = {
  firstName: "John",
  lastName: "Doe"
};

function getUserProfile(userId: string): Promise<User> {
  // implementation
}

// Classes & Types: PascalCase
class UserService {}
interface UserProfile {}
type AnalysisResult = {};
enum Status {}
```

#### File Naming

```
src/
├── components/      # React components
│   ├── DatasetManager.tsx
│   ├── VisualizationChart.tsx
│   └── Dashboard.tsx
├── pages/          # Page components
│   ├── HomePage.tsx
│   └── ProjectPage.tsx
├── hooks/          # Custom hooks
│   ├── useDataset.ts
│   └── useAnalysis.ts
├── services/       # Services
│   ├── apiClient.ts
│   └── analyticsService.ts
└── utils/          # Utilities
    └── formatting.ts
```

### TypeScript

#### Type Annotations

```typescript
// ✅ Good: Explicit types
interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

function getUser(userId: string): Promise<User> {
  return fetchUser(userId);
}

// ❌ Avoid: any type
function getUser(userId: any): any {
  return fetchUser(userId);
}

// ✅ Good: Union types
type Status = "active" | "inactive" | "pending";

// ✅ Good: Generic types
function processArray<T>(items: T[]): T[] {
  return items.map(item => transform(item));
}
```

#### Imports

```typescript
// ✅ Organize imports
// 1. External libraries
import React, { useState, useEffect } from "react";
import axios from "axios";

// 2. Relative imports
import { UserService } from "../services/UserService";
import { formatDate } from "../utils/formatting";

// ✅ Use named imports
import { getUser, updateUser } from "../api/users";

// ❌ Avoid: default imports when multiple exports
// import * as users from "../api/users";
```

### React Components

#### Functional Components

```typescript
// ✅ Good: Functional component with hooks
interface DatasetManagerProps {
  projectId: string;
  onDatasetSelect?: (dataset: Dataset) => void;
}

export const DatasetManager: React.FC<DatasetManagerProps> = ({
  projectId,
  onDatasetSelect
}) => {
  const [datasets, setDatasets] = useState<Dataset[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchDatasets = async () => {
      setLoading(true);
      try {
        const data = await api.getDatasets(projectId);
        setDatasets(data);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };

    fetchDatasets();
  }, [projectId]);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div className="dataset-manager">
      {datasets.map(dataset => (
        <DatasetCard
          key={dataset.id}
          dataset={dataset}
          onSelect={() => onDatasetSelect?.(dataset)}
        />
      ))}
    </div>
  );
};

export default DatasetManager;
```

#### Props Pattern

```typescript
// ✅ Good: Explicit Props interface
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: "primary" | "secondary";
  size?: "small" | "medium" | "large";
}

export const Button: React.FC<ButtonProps> = ({
  label,
  onClick,
  disabled = false,
  variant = "primary",
  size = "medium"
}) => {
  // component
};
```

### ESLint & Prettier

```json
// .eslintrc.json
{
  "extends": ["eslint:recommended", "react-app"],
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "prefer-const": "error",
    "eqeqeq": "error"
  }
}

// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 100,
  "tabWidth": 2
}
```

**Run linters**:
```bash
npm run lint          # Run ESLint
npm run lint:fix      # Auto-fix issues
npm run format        # Prettier format
```

## Python Standards

### Code Style (PEP 8)

#### Naming Conventions

```python
# Constants: UPPER_SNAKE_CASE
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 5000

# Variables & Functions: snake_case
user_data = {
    "first_name": "John",
    "last_name": "Doe"
}

def get_user_profile(user_id: str) -> User:
    """Get user profile by ID."""
    pass

# Classes: PascalCase
class UserService:
    pass
```

#### Type Hints

```python
# ✅ Good: Full type hints
from typing import List, Dict, Optional, Union

def process_users(
    users: List[User],
    filter_active: bool = True
) -> Dict[str, User]:
    """Process users list."""
    return {user.id: user for user in users}

def get_user(
    user_id: Union[str, int]
) -> Optional[User]:
    """Get user by ID."""
    pass

# ✅ Good: Function docstrings
def analyze_dataset(dataset_id: str, depth: int = 1) -> AnalysisResult:
    """
    Analyze dataset with AI.
    
    Args:
        dataset_id: The ID of the dataset to analyze
        depth: Analysis depth level (1-5)
        
    Returns:
        Analysis result with insights
        
    Raises:
        ValueError: If dataset_id is invalid
        RuntimeError: If analysis fails
    """
    pass
```

#### Import Organization

```python
# ✅ Organize imports
# 1. Standard library
import os
import sys
from typing import List, Optional
from datetime import datetime

# 2. Third-party libraries
import numpy as np
import pandas as pd
from fastapi import FastAPI, HTTPException

# 3. Local imports
from backend.app.services import UserService
from backend.app.models import User
from backend.app.config import settings
```

### FastAPI Best Practices

```python
from fastapi import FastAPI, APIRouter, HTTPException, Depends
from pydantic import BaseModel
from typing import List

app = FastAPI()
router = APIRouter(prefix="/api/v1/users", tags=["users"])

# ✅ Good: Pydantic models
class UserCreate(BaseModel):
    """User creation model."""
    email: str
    name: str
    
    class Config:
        schema_extra = {
            "example": {
                "email": "user@example.com",
                "name": "John Doe"
            }
        }

class UserResponse(BaseModel):
    """User response model."""
    id: str
    email: str
    name: str
    created_at: datetime

# ✅ Good: Endpoint documentation
@router.get("/{user_id}", response_model=UserResponse)
async def get_user(user_id: str):
    """
    Get user by ID.
    
    - **user_id**: User's unique identifier
    
    Returns:
        User object
        
    Raises:
        HTTPException: If user not found
    """
    user = await UserService.get_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@router.post("", response_model=UserResponse, status_code=201)
async def create_user(user: UserCreate):
    """Create new user."""
    existing = await UserService.get_by_email(user.email)
    if existing:
        raise HTTPException(status_code=409, detail="Email already exists")
    return await UserService.create(user)
```

### Linting & Formatting

```bash
# Black: Code formatter
black backend/

# isort: Import sorting
isort backend/

# Flake8: Style guide enforcement
flake8 backend/

# Pylint: Code analysis
pylint backend/

# MyPy: Type checking
mypy backend/

# All together
black backend/ && isort backend/ && flake8 backend/ && mypy backend/
```

## Git Standards

### Commit Messages

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Style changes
- `refactor`: Code refactoring
- `perf`: Performance improvement
- `test`: Test changes
- `chore`: Build/CI changes

#### Examples

```
feat(data-cleaning): add automatic null value detection

Implement AI-powered null value detection in data profiling.
Uses statistical methods to identify and suggest filling strategies.

- Detects null values per column
- Calculates severity score
- Suggests filling methods
- Adds validation tests

Closes #234
```

```
fix(visualization): correct pie chart percentage rounding

Previously pie chart percentages didn't sum to 100% due to rounding.
Fixed by adjusting final slice percentage to account for accumulated error.

Fixes #567
```

### Branch Naming

```
feature/description           # New features
bugfix/description           # Bug fixes
docs/description             # Documentation
refactor/description         # Refactoring
perf/description             # Performance
```

## Error Handling

### TypeScript/JavaScript

```typescript
// ✅ Good: Proper error handling
async function fetchAnalysis(id: string): Promise<Analysis> {
  try {
    const response = await fetch(`/api/v1/analyses/${id}`);
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    if (error instanceof NetworkError) {
      console.error("Network error:", error.message);
      throw new Error("Failed to connect to server");
    }
    
    throw error;
  }
}

// ✅ Good: Error boundaries in React
class ErrorBoundary extends React.Component {
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("Error caught:", error);
    this.setState({ hasError: true });
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}
```

### Python

```python
# ✅ Good: Specific exception handling
def process_dataset(file_path: str) -> Dataset:
    """Process dataset file."""
    try:
        data = pd.read_csv(file_path)
    except FileNotFoundError:
        raise ValueError(f"File not found: {file_path}")
    except pd.errors.ParserError as e:
        raise ValueError(f"Invalid CSV format: {str(e)}")
    except Exception as e:
        logger.error(f"Unexpected error: {str(e)}")
        raise
    
    return Dataset(data=data)

# ✅ Good: Custom exceptions
class DatasetError(Exception):
    """Base exception for dataset operations."""
    pass

class InvalidDatasetError(DatasetError):
    """Raised when dataset is invalid."""
    pass

class DatasetNotFoundError(DatasetError):
    """Raised when dataset is not found."""
    pass
```

## Documentation

### Docstrings

#### TypeScript

```typescript
/**
 * Analyzes a dataset using AI profiling.
 * 
 * @param datasetId - The ID of the dataset to analyze
 * @param options - Optional analysis configuration
 * @returns Promise resolving to analysis results
 * @throws Error if dataset not found
 * 
 * @example
 * const results = await analyzeDataset("ds_123");
 */
function analyzeDataset(
  datasetId: string,
  options?: AnalysisOptions
): Promise<AnalysisResult> {
  // implementation
}
```

#### Python

```python
def analyze_dataset(dataset_id: str, depth: int = 1) -> dict:
    """
    Analyze dataset using AI profiling.
    
    This function performs comprehensive analysis on a dataset including
    profiling, quality assessment, and pattern detection.
    
    Args:
        dataset_id: Unique identifier for the dataset
        depth: Analysis depth level (1-5), default is 1
        
    Returns:
        Dictionary containing:
            - quality_score: Overall quality (0-1)
            - columns: Column analysis results
            - issues: Detected data quality issues
            
    Raises:
        ValueError: If dataset_id is invalid
        RuntimeError: If analysis fails
        
    Example:
        >>> result = analyze_dataset("ds_123")
        >>> print(result["quality_score"])
        0.92
    """
```

## Testing Standards

### Test Naming

```typescript
// ✅ Clear test names
describe("DatasetManager", () => {
  it("should display list of datasets", () => {});
  it("should upload new dataset with correct format", () => {});
  it("should show error for invalid file format", () => {});
});
```

```python
# ✅ Clear test names
def test_profile_returns_quality_score():
    """Test that profiling returns quality score."""
    pass

def test_profile_raises_error_for_empty_dataset():
    """Test that profiling raises error for empty dataset."""
    pass
```

## Performance Guidelines

### Optimization Checklist

- [ ] Minimize bundle size
- [ ] Use lazy loading
- [ ] Implement caching
- [ ] Optimize database queries
- [ ] Use async operations
- [ ] Profile regularly
- [ ] Monitor performance

### React Performance

```typescript
// ✅ Memoization
const DatasetItem = React.memo(({ dataset, onClick }: Props) => {
  return <div onClick={() => onClick(dataset)}>{dataset.name}</div>;
});

// ✅ useCallback
const handleSelect = useCallback((dataset: Dataset) => {
  setSelected(dataset);
}, []);

// ✅ useMemo
const filteredDatasets = useMemo(
  () => datasets.filter(d => d.status === "active"),
  [datasets]
);
```

### Python Performance

```python
# ✅ Connection pooling
from sqlalchemy.pool import QueuePool

engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,
    max_overflow=40
)

# ✅ Query optimization
# Use .select() instead of loading all
users = session.query(User).filter(User.active == True).all()

# Use bulk operations
session.bulk_insert_mappings(User, user_list)
```

## Code Review Checklist

- [ ] Code follows naming conventions
- [ ] Type safety is maintained
- [ ] Error handling is comprehensive
- [ ] Tests are included
- [ ] Documentation is clear
- [ ] No console logs in production code
- [ ] No hardcoded values
- [ ] Performance considerations addressed
- [ ] Security best practices followed
- [ ] Accessibility considerations made

---

## Related Documents
- [Git Workflow](Git_Workflow.md)
- [Testing Strategy](../09_Testing/Testing_Strategy.md)
- [Deployment Guide](Deployment.md)

**Last Updated**: December 2024
