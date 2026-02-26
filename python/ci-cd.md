---
title: "Python CI/CD"
summary: "GitHub Actions workflows for Python projects using uv, ruff, pyright, and pytest with automated quality checks."
read_when:
  - Setting up CI/CD for Python projects
  - Automating code quality checks and testing
  - Configuring GitHub Actions for Python toolchain
  - Implementing continuous integration workflows
status: active
last_updated: "2025-01-16"
---

# Python CI/CD

## GitHub Actions Workflow

**.github/workflows/ci.yml**

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v3
        with:
          version: "latest"

      - name: Set up Python ${{ matrix.python-version }}
        run: uv python install ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          uv sync --extra dev

      - name: Check formatting
        run: |
          uv run ruff format --check

      - name: Lint code
        run: |
          uv run ruff check

      - name: Type check
        run: |
          uv run pyright

      - name: Run tests
        run: |
          uv run pytest --cov=src --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml
          fail_ci_if_error: true
```

## Additional Workflows

### Dependency Updates

**.github/workflows/dependabot.yml**

```yaml
name: Dependabot Auto-merge

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Enable auto-merge for Dependabot PRs
        run: gh pr merge --auto --merge "$PR_URL"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

### Release Workflow

**.github/workflows/release.yml**

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v3

      - name: Build package
        run: |
          uv build

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          password: ${{ secrets.PYPI_API_TOKEN }}
```

## Configuration Files

### Dependabot

**.github/dependabot.yml**

```yaml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

### Coverage Configuration

**pyproject.toml additions:**

```toml
[tool.coverage.run]
source = ["src"]
omit = ["tests/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
]
```

## Quality Gates

### Required Checks
- ✅ Code formatting (ruff format)
- ✅ Linting (ruff check)
- ✅ Type checking (pyright)
- ✅ Tests passing (pytest)
- ✅ Coverage threshold (>80%)

### Branch Protection
Configure branch protection rules in GitHub:
- Require status checks to pass
- Require up-to-date branches
- Require review from code owners