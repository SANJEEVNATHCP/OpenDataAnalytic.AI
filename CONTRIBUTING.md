# Contributing to InsightOS

Thank you for your interest in contributing to InsightOS. This document provides guidelines and instructions for contributing to the project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [How to Contribute](#how-to-contribute)
- [Coding Standards](#coding-standards)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Reporting Issues](#reporting-issues)
- [Community](#community)

## Code of Conduct

By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md). Please read it before contributing.

## Getting Started

### Prerequisites

- Node.js 18+
- Python 3.10+
- Git
- Familiarity with React, TypeScript, and FastAPI

### Ways to Contribute

We welcome contributions in many forms:

- **Code**: Bug fixes, features, and improvements
- **Documentation**: Writing, editing, and improving docs
- **Design**: UI/UX improvements and feedback
- **Testing**: Writing tests, finding bugs, quality assurance
- **Community**: Helping others, answering questions, organizing
- **Feedback**: Feature suggestions and product feedback

## Development Setup

### Clone the Repository

```bash
git clone https://github.com/insightos/insightos.git
cd insightos
```

### Frontend Setup

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Run type checking
npm run type-check

# Run linting
npm run lint

# Run tests
npm run test

# Build for production
npm run build
```

### Backend Setup

```bash
# Create virtual environment
python -m venv .venv

# Activate virtual environment
# On Windows:
.venv\Scripts\activate
# On macOS/Linux:
source .venv/bin/activate

# Install dependencies
pip install -r backend/requirements.txt

# Start development server
python -m uvicorn backend.app.main:app --reload

# Run tests
pytest

# Run linting
pylint backend/

# Run type checking
mypy backend/
```

### Database Setup

```bash
# Initialize database
python backend/scripts/init_db.py

# Run migrations
alembic upgrade head
```

## How to Contribute

### Finding Issues

1. Check [open issues](https://github.com/insightos/insightos/issues)
2. Look for issues labeled `good-first-issue` or `help-wanted`
3. Read the issue description and comments
4. Comment to claim the issue before starting work

### Creating a Feature Branch

```bash
# Update main branch
git checkout main
git pull origin main

# Create feature branch
# Use format: feature/short-description or bugfix/short-description
git checkout -b feature/my-feature-name
```

### Making Changes

1. Make focused, atomic commits
2. Write clear commit messages
3. Keep commits small and logical
4. Include tests for new code
5. Update documentation as needed

### Running Tests Locally

```bash
# Frontend tests
npm run test

# Backend tests
pytest

# All tests
npm run test && pytest
```

### Building Locally

```bash
# Frontend
npm run build

# Desktop app (Tauri)
cd src-tauri
cargo build --release
```

## Coding Standards

### TypeScript/React

- Follow ESLint configuration
- Use TypeScript for type safety
- Write component-level documentation
- Use functional components with hooks
- Follow established naming conventions

```typescript
// Good
const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);
  
  return (
    <div className="user-profile">
      {/* component code */}
    </div>
  );
};

export default UserProfile;
```

### Python

- Follow PEP 8
- Use type hints
- Write docstrings using Google style
- Use meaningful variable names
- Keep functions focused

```python
"""User service module."""

def get_user_by_id(user_id: int) -> Optional[User]:
    """
    Retrieve a user by their ID.
    
    Args:
        user_id: The unique identifier of the user
        
    Returns:
        User object if found, None otherwise
        
    Raises:
        ValueError: If user_id is invalid
    """
    if not isinstance(user_id, int) or user_id <= 0:
        raise ValueError("user_id must be a positive integer")
    
    return database.query(User).filter(User.id == user_id).first()
```

### Documentation

- Use clear, concise language
- Include code examples
- Document edge cases
- Add links to related documentation
- Keep READMEs updated

## Commit Guidelines

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, semicolons)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Test additions or changes
- `chore`: Build, CI, dependencies
- `ci`: CI/CD configuration

### Example Commits

```
feat(data-profiling): add automated null value detection

Add automatic detection of null values in dataset profiling.
Implements confidence scoring for each detected pattern.

Closes #123
```

```
fix(visualization): correct pie chart percentage calculation

Previously pie chart percentages didn't sum to 100%.
Fixed rounding logic in percentage calculation.

Fixes #456
```

### Commit Best Practices

- Write meaningful messages
- Reference related issues
- Keep commits focused
- Use imperative mood in subject line
- Capitalize the subject line
- Don't exceed 72 characters in subject

## Pull Request Process

### Before Creating a PR

1. Ensure all tests pass locally
2. Run linting and type checking
3. Update documentation
4. Rebase on latest main
5. Squash if requested during review

### Creating a Pull Request

1. Push your branch to GitHub
2. Open a pull request against `main`
3. Fill out the PR template completely
4. Link related issues
5. Describe changes and testing

### PR Title Format

```
<type>(<scope>): <description>
```

Example: `feat(cleaning): add column renaming suggestions`

### PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issues
Closes #123

## Changes
- Detailed change 1
- Detailed change 2

## Testing
Describe testing performed:
- Unit tests added
- Integration tests passed
- Manual testing on [platform]

## Screenshots (if applicable)
Screenshots or recordings

## Checklist
- [ ] Code follows style guidelines
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] No breaking changes
```

### Review Process

1. Code review by maintainers
2. Automated checks must pass
3. Address review comments
4. Re-request review after changes
5. Maintainer approval and merge

### Approval Requirements

- At least 1 maintainer approval
- All CI checks passing
- Tests coverage maintained
- No merge conflicts

## Reporting Issues

### Before Creating an Issue

1. Search existing issues
2. Check documentation
3. Try the latest code
4. Gather system information

### Issue Template

```markdown
## Description
Clear description of the issue

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: [e.g., Windows 10]
- Node: [e.g., 18.0.0]
- Python: [e.g., 3.10.0]
- Browser: [e.g., Chrome 120]

## Screenshots
If applicable

## Error Messages
Full error logs/stack traces
```

### Issue Labels

- `bug`: Something isn't working
- `enhancement`: Feature request
- `documentation`: Docs improvement
- `good-first-issue`: Good for new contributors
- `help-wanted`: Extra help needed
- `question`: Further information needed

## Community

### Getting Help

- Check [documentation](docs/)
- Search [GitHub Discussions](https://github.com/insightos/insightos/discussions)
- Ask in issue comments
- Email support@insightos.dev

### Contributing Documentation

1. Fork the repository
2. Create a branch
3. Make documentation changes
4. Submit a pull request
5. Follow review process

### Sharing Feedback

- Participate in discussions
- Provide product feedback
- Help test new features
- Share your use cases

## Questions?

For questions about contributing:
- Email: contributors@insightos.dev
- GitHub Discussions: [Community](https://github.com/insightos/insightos/discussions)

---

Thank you for contributing to InsightOS!
