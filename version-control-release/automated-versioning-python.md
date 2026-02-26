---
title: "Automated Versioning Setup (Python)"
summary: "Step-by-step guide for setting up automated semantic versioning, changelog generation, and release workflows for Python projects."
read_when:
  - Implementing semantic commits with tooling automation
  - Setting up commitizen and conventional commits validation
  - Configuring automated changelog generation and versioning
  - Creating CI/CD workflows for semantic releases
status: active
last_updated: "2025-01-16"
---

# Automated Versioning Setup (Python)

## Installation

```bash
uv add --dev commitizen
```

## Configuration

### pyproject.toml

```toml
[tool.commitizen]
version = "0.9.8"
tag_format = "v$version"
update_changelog_on_bump = true
version_scheme = "semver"
bump_message = "bump: v$old_version → v$new_version"

[tool.commitizen.changelog]
changelog_file = "docs/CHANGELOG.md"
tag_format = "v$version"
```

### Pre-commit Hooks

**.pre-commit-config.yaml**

```yaml
repos:
  - repo: https://github.com/commitizen-tools/commitizen
    rev: v4.1.0
    hooks:
      - id: commitizen
        stages: [commit-msg]
      - id: cz-check
        stages: [commit-msg]

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
```

Install hooks:
```bash
pre-commit install
```

## GitHub Actions Workflow

**.github/workflows/release.yml**

```yaml
name: Semantic Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install commitizen
        run: uv pip install commitizen

      - name: Semantic Release
        run: |
          cz bump
          cz changelog
          
      - name: Push changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add pyproject.toml CHANGELOG.md
          git commit -m "chore: update version and changelog"
          git push origin main --tags
```

## Usage

### Interactive Commit
```bash
cz commit
```

### Version Bump
```bash
cz bump
```

### Generate Changelog
```bash
cz changelog
```