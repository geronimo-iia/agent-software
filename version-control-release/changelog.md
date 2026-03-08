---
title: "Changelog"
summary: "Standard for maintaining a human-readable changelog using Keep a Changelog format, derived from semantic commits."
read_when:
  - Writing or updating a CHANGELOG.md file
  - Deciding what to include in a changelog entry
  - Promoting an unreleased section to a version on release
  - Understanding the relationship between commits and changelog entries
status: active
last_updated: "2026-07-15"
---

# Changelog

A changelog is a curated, human-readable record of notable changes per release. It is not a git log — it is written for users and contributors, not machines.

Follow the [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format.

---

## Format

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

## [1.2.0] - 2026-07-15

### Added
- `--quiet` global flag — suppresses output, implies `--yes`
- `--yes` global flag — auto-approves all lifecycle steps

### Changed
- `execute_lifecycle` signature gains `quiet: bool` parameter

### Fixed
- Version read from `metadata.version` per spec, not top-level

## [1.1.0] - 2026-06-01

### Added
- Hub registry with TTL-based index cache
```

---

## Sections

Use only the sections that apply to the release. Omit empty sections.

| Section | When to use |
|---|---|
| `Added` | New features, new commands, new flags |
| `Changed` | Changes to existing behavior |
| `Deprecated` | Features that will be removed in a future release |
| `Removed` | Features removed in this release |
| `Fixed` | Bug fixes |
| `Security` | Security fixes |

---

## Rules

**`[Unreleased]`** — always present at the top. Accumulate changes here as work lands. Promote to a version entry on release.

**Version headers** — use `[MAJOR.MINOR.PATCH] - YYYY-MM-DD`. Version must match `Cargo.toml` / `pyproject.toml`. Date is the release date.

**Entries are user-facing** — describe the impact, not the implementation. One line per entry.

**Derived from semantic commits** — `feat:` → Added, `fix:` → Fixed, `BREAKING CHANGE` → Changed or Removed. See [git-commit-semantic.md](git-commit-semantic.md).

---

## What to exclude

- `chore:` commits (dependency bumps, CI tweaks, formatting)
- `refactor:` commits with no behavior change
- `docs:` commits (unless the doc is the product)
- Internal implementation details irrelevant to users

---

## When to update `[Unreleased]`

Update `CHANGELOG.md` in the same commit as the change — not as a separate pass before release:

- Adding a new feature (`feat:`) → add entry under `Added`
- Fixing a bug (`fix:`) → add entry under `Fixed`
- Breaking a public API or behavior → add entry under `Changed` or `Removed`
- Deprecating something → add entry under `Deprecated`

Do not wait until release day to write changelog entries — context is lost and entries become vague.

---

## On release

1. Rename `[Unreleased]` to `[X.Y.Z] - YYYY-MM-DD`
2. Add a new empty `[Unreleased]` section above it
3. Bump version in `Cargo.toml` / `pyproject.toml`
4. Commit: `chore: release vX.Y.Z`
5. Tag: `vX.Y.Z`

See [semantic-versioning.md](semantic-versioning.md) for version bump rules.
