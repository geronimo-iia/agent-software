# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added
- `information-flow-tracking/` — taint labels, propagation rules, sink policies (merged from `agent-software-engineering`)
- `version-control-release/changelog.md` — changelog standard with format, sections, update guidelines, and release process

### Changed
- README intro rewritten to reflect broader scope

## [0.2.0] - 2026-07-15

### Added
- `agentctl.toml` — hub id `geronimo-iia/agent-software`, ignore list
- `CHANGELOG.md`, `CONTRIBUTING.md`, `LICENSE` (MIT)

### Changed
- CI workflow migrated from `agent-hub-indexer` to `agentctl`
- `README.md` rewritten — opinionated intro, topics table, standards references, license and contributing links
- `rust/ci-cd.md` — fixed deprecated `cargo publish --token` flag, added `cargo audit` step, added Release Checklist section
- `rust/project-structure-tools.md` — added Testing Patterns section (fixtures, function pointer injection, tempdir, CLI flag isolation)

## [0.1.0] - 2025-01-16

### Added
- Initial specifications: `version-control-release/`, `python/`, `rust/`, `wasm/`
- `index.json` generated via `agent-hub-indexer`
- CI workflow publishing `index.json` on push to `main`
