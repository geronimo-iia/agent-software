---
title: "Semantic Commits"
summary: "Guide for semantic commit format and conventions to standardize git history and enable automated versioning."
read_when:
  - Setting up commit message standards for a project
  - Understanding semantic commit format and conventions
  - Learning conventional commits specification
status: active
last_updated: "2025-01-16"
---

# Semantic Commits

## Overview

This document provides a guide for adopting **Semantic Commits** in software projects. This standardization improves:
- Git history readability and traceability
- Automated changelog generation
- Release automation
- Breaking change detection
- Version compatibility management

## Semantic Commit Format

### Basic Structure

```
<type>(<scope>): <subject>

<body> (optional)

<footer> (optional)
```

### Commit Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(api): add user authentication` |
| `fix` | Bug fix | `fix(database): correct connection timeout` |
| `refactor` | Code restructuring | `refactor(utils): simplify helper functions` |
| `docs` | Documentation changes | `docs(README): update installation instructions` |
| `style` | Code style (formatting, semicolon) | `style: format code with ruff` |
| `perf` | Performance improvement | `perf(api): optimize database queries` |
| `test` | Test additions/changes | `test(auth): add authentication tests` |
| `chore` | Build process or auxiliary tool changes | `chore(deps): update dependencies` |
| `ci` | CI configuration changes | `ci(workflow): add Docker build job` |
| `build` | Build system changes | `build: update pyproject.toml` |

### Conventional Commits Specification

#### Breaking Changes
```
feat(api): add new authentication provider interface

BREAKING CHANGE: The authentication configuration format has changed.
Old format: {"provider": "oauth", "client_id": "123"}
New format: {"provider": "oauth", "client_id": "123", "scopes": ["read", "write"]}
```

#### Issue References
```
fix(database): resolve connection leak in pool management

Closes: #123
Refs: #456, #789
```


