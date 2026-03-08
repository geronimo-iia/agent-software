# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added
- `rules/` — workspace rules best practices (merged from `agent-software-engineering`)
- `information-flow-tracking/` — taint labels, propagation rules, sink policies (merged from `agent-software-engineering`)

### Changed
- README intro rewritten to reflect broader scope

## [0.2.0] - 2026-07-15

### Changed
- CI workflow migrated from `agent-hub-indexer` to `agentctl`
- `README.md` rewritten with frontmatter and topic table
- `rust/ci-cd.md` — fixed deprecated `cargo publish --token` flag, added `cargo audit` step, added Release Checklist section
- `rust/project-structure-tools.md` — added Testing Patterns section (fixtures, function pointer injection, tempdir, CLI flag isolation)

## [0.1.0] - 2025-01-16

### Added
- Initial specifications: `version-control-release/`, `python/`, `rust/`, `wasm/`
- `index.json` generated via `agent-hub-indexer`
- CI workflow publishing `index.json` on push to `main`
