---
title: "Semantic Versioning (SemVer)"
summary: "Guide for semantic versioning principles and version calculation rules based on commit types."
read_when:
  - Understanding semantic versioning (SemVer) format and rules
  - Calculating version bumps from commit types
  - Setting up automated version management
status: draft
last_updated: "2025-01-16"
---

# Semantic Versioning (SemVer)

## Format
```
MAJOR.MINOR.PATCH
```

## Version Bump Rules

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| Breaking changes | MAJOR | 0.9.8 → 1.0.0 |
| New features (backward compatible) | MINOR | 0.9.8 → 0.10.0 |
| Bug fixes (backward compatible) | PATCH | 0.9.8 → 0.9.9 |

## Commit Type to Version Mapping

| Commit Type | Version Part |
|-------------|--------------|
| `BREAKING CHANGE` + any type | MAJOR |
| `feat:` (no breaking) | MINOR |
| `fix:`, `perf:`, `test:` | PATCH |
| `docs:`, `style:`, `chore:`, `ci:` | No bump |

## Version Calculation Examples

| Commits in Release | Version Change |
|-------------------|----------------|
| `feat(api): add new endpoint` | 0.9.8 → 0.10.0 |
| `fix(auth): correct token validation` | 0.9.8 → 0.9.9 |
| `feat(api): BREAKING CHANGE` | 0.9.8 → 1.0.0 |
| `docs(README): update guide` | No version bump |

## Changelog Generation

Commits are automatically categorized:
- **Added**: `feat:` commits
- **Changed**: `refactor:` commits  
- **Fixed**: `fix:` commits
- **Breaking Changes**: Commits with `BREAKING CHANGE:` footer