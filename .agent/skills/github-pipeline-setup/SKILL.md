---
name: github-pipeline-setup
description: Set up robust GitHub Actions CI/CD pipelines for testing. Use this skill whenever the user mentions GitHub Actions, CI/CD pipelines, workflow files, .github/workflows, setting up automated testing, pull request checks, or wants to configure continuous integration. Also trigger when they want to add test runners, coverage reports, or branch protection to their repository.
---

# GitHub Pipeline Setup

This skill helps you create production-ready GitHub Actions workflows focused on comprehensive testing with proper configuration, caching, and coverage reporting.

## Core Principles

1. **Pull Request Protection**: Never allow direct pushes to main - all changes through PRs
2. **Trigger on PR Activity**: Run tests on PR creation and each new commit
3. **Use npm/Node.js**: Default to npm for JavaScript/TypeScript projects
4. **Coverage Tracking**: Generate and report test coverage (no README badges)
5. **Fail Fast**: Stop on first test failure to save CI minutes

## Workflow Structure

### Basic Template

All workflows should follow this structure:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]  # Only for commits that make it through PR

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Environment
        # ... setup steps
      
      - name: Install Dependencies
        # ... with caching
      
      - name: Run Tests
        # ... with coverage
      
      - name: Upload Coverage
        # ... coverage artifacts
```

## Implementation Guidelines

### 1. Environment Setup

**Node.js Projects (React, Vue):**
```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'  # Use LTS version
    cache: 'npm'        # Built-in npm caching
```

**Python Projects:**
```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.11'  # Use stable recent version
    cache: 'pip'            # Built-in pip caching
```

**Multi-Version Testing (Test Matrices):**
```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    # or
    python-version: ['3.10', '3.11', '3.12']
```

### 2. Dependency Installation

**npm (always use for JS projects):**
```yaml
- name: Install dependencies
  run: npm ci  # Use ci instead of install for reproducible builds
```

**Python:**
```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
    pip install pytest pytest-cov  # Add test dependencies
```

### 3. Running Tests with Coverage

**Vitest (React/Vue):**
```yaml
- name: Run tests
  run: npm test -- --coverage --run
  
- name: Upload coverage reports
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: coverage-report
    path: coverage/
```

**pytest (Python):**
```yaml
- name: Run tests
  run: pytest --cov=. --cov-report=xml --cov-report=html
  
- name: Upload coverage reports
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: coverage-report
    path: |
      htmlcov/
      coverage.xml
```

### 4. Environment Variables & Secrets

**Public Environment Variables:**
```yaml
env:
  NODE_ENV: test
  CI: true
```

**Secrets (sensitive data):**
```yaml
- name: Run tests
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    API_KEY: ${{ secrets.API_KEY }}
  run: npm test
```

**Setting secrets:** Go to repo Settings → Secrets and variables → Actions → New repository secret

### 5. Branch Protection Rules

After creating the workflow, configure branch protection:

1. Go to repo Settings → Branches → Add rule
2. Branch name pattern: `main`
3. Enable:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Select your CI workflow as a required status check
4. Optional but recommended:
   - ✅ Require linear history (no merge commits)
   - ✅ Do not allow bypassing the above settings

This prevents direct pushes to main - all changes must go through PRs with passing tests.

### 6. Advanced Patterns

**Monorepo with Multiple Projects:**
```yaml
jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      react: ${{ steps.filter.outputs.react }}
      vue: ${{ steps.filter.outputs.vue }}
      python: ${{ steps.filter.outputs.python }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            react:
              - 'packages/react/**'
            vue:
              - 'packages/vue/**'
            python:
              - 'packages/python/**'
  
  test-react:
    needs: detect-changes
    if: needs.detect-changes.outputs.react == 'true'
    runs-on: ubuntu-latest
    steps:
      # ... test React project
```

**Parallel Jobs:**
```yaml
jobs:
  test-frontend:
    runs-on: ubuntu-latest
    steps:
      # ... frontend tests
  
  test-backend:
    runs-on: ubuntu-latest
    steps:
      # ... backend tests
```

**Fail Fast Strategy:**
```yaml
strategy:
  fail-fast: true  # Stop all jobs if one fails
  matrix:
    node-version: [18, 20]
```

## Complete Examples

### React/Vue Project

```yaml
name: Frontend CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests with coverage
        run: npm test -- --coverage --run
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-report
          path: coverage/
          retention-days: 30
```

### Python/Django Project

```yaml
name: Python CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-django
      
      - name: Run tests with coverage
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          DJANGO_SETTINGS_MODULE: myproject.settings.test
        run: |
          pytest --cov=. --cov-report=xml --cov-report=html --cov-report=term
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-report
          path: |
            htmlcov/
            coverage.xml
          retention-days: 30
```

### Full-Stack Monorepo

```yaml
name: Full-Stack CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  test-frontend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage --run
      
      - name: Upload coverage
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: frontend-coverage
          path: frontend/coverage/
  
  test-backend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./backend
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
          cache-dependency-path: backend/requirements.txt
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        run: pytest --cov=. --cov-report=xml --cov-report=html
      
      - name: Upload coverage
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: backend-coverage
          path: backend/htmlcov/
```

## Output Format

When setting up a pipeline, always:

1. **Ask clarifying questions** if project structure is unclear:
   - Single project or monorepo?
   - Need database services (postgres, redis, etc.)?
   - Any special build steps or environment setup?

2. **Create the workflow file** at `.github/workflows/ci.yml`

3. **Provide setup instructions**:
   - Where to place the file
   - How to configure branch protection rules
   - How to add secrets if needed
   - How to view coverage reports after test runs

4. **Explain what it does** in plain language so they understand each section

## Common Patterns to Include

- **Caching**: Always use built-in caching for npm/pip
- **Artifact Upload**: Always upload coverage reports with `if: always()` so they're available even if tests fail
- **Retention**: Set retention-days (default 90 is often too long, suggest 30)
- **Clear Job Names**: Use descriptive names like "test-frontend" not "job1"
- **Explicit Node/Python Versions**: Don't use "latest" - pin to specific LTS versions

## What NOT to Do

- ❌ Don't use `npm install` - use `npm ci` for reproducible builds
- ❌ Don't forget `if: always()` on coverage uploads
- ❌ Don't use wildcards in branch protection (be explicit: `main` not `main*`)
- ❌ Don't add deployment steps yet (user wants to focus on testing first)
- ❌ Don't create README badges (user doesn't want them)
- ❌ Don't use deprecated actions (use @v4, @v5, not @v2)

## Tips for Success

- Start simple, add complexity only if needed
- Test the workflow on a feature branch first
- Monitor CI usage in repo Insights → Actions to optimize runtime
- Group related steps with clear names
- Add comments in YAML for complex configurations
