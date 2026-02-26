---
title: "Python Project Structure & Tools"
summary: "Standard Python project structure using modern tooling: pyproject.toml, uv, ruff, pyright, and pytest."
read_when:
  - Setting up new Python projects
  - Standardizing Python project toolchain
  - Configuring modern Python development environment
  - Establishing Python project conventions
status: active
last_updated: "2025-01-16"
---

# Python Project Structure & Tools

## Standard Project Structure

```
project-name/
├── pyproject.toml          # Project configuration
├── README.md               # Project documentation
├── .gitignore             # Git ignore patterns
├── src/
│   └── project_name/      # Source code package
│       ├── __init__.py
│       └── main.py
├── tests/                 # Test files
│   ├── __init__.py
│   └── test_main.py
└── docs/                  # Documentation (optional)
```

## Core Toolchain

### Package Management: uv
- **Dependency management**: Fast, reliable Python package installer
- **Virtual environments**: Built-in venv management
- **Lock files**: Deterministic dependency resolution

### Code Quality: ruff
- **Linting**: Fast Python linter (replaces flake8, pylint)
- **Formatting**: Code formatter (replaces black)
- **Import sorting**: Automatic import organization

### Type Checking: pyright
- **Static analysis**: Type checking and error detection
- **IDE integration**: Language server protocol support
- **Configuration**: Flexible type checking rules

### Testing: pytest
- **Test framework**: Simple, powerful testing
- **Fixtures**: Reusable test components
- **Plugins**: Extensive ecosystem

## Configuration: pyproject.toml

```toml
[project]
name = "project-name"
version = "0.1.0"
description = "Project description"
authors = [{name = "Author Name", email = "author@example.com"}]
requires-python = ">=3.11"
dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "ruff>=0.1.0",
    "pyright>=1.1.0",
]

[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[tool.ruff]
line-length = 120
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]
ignore = []

[tool.pyright]
include = ["src"]
exclude = ["**/__pycache__"]
typeCheckingMode = "basic"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
```

## Development Workflow

### Setup
```bash
# Install dependencies
uv sync

# Install development dependencies
uv sync --extra dev
```

### Code Quality
```bash
# Format and lint
uv run ruff format
uv run ruff check --fix

# Type checking
uv run pyright
```

### Testing
```bash
# Run tests
uv run pytest

# Run with coverage
uv run pytest --cov=src
```