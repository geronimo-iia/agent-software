# Agent Software

Opinionated standards, engineering practices, and methodologies for agent software development — organized by language and practice area.

This project contains knowledge documents only — no skills, no executable code.

## Workspace Rules

The `rules/` directory contains templates for Amazon Q workspace rules that automatically activate relevant standards as context. Copy templates from `rules/templates/` to your project's `.amazonq/rules/` directory to enable automatic context inclusion when working with specific file types or topics.

## Topics

### Mature

| Topic | Description |
|---|---|
| [version-control-release/](version-control-release/definition.md) | Semantic commits, SemVer, git workflows, automated releases |
| [python/](python/definition.md) | Project structure, tooling (uv, ruff, pyright), CI/CD |
| [rust/](rust/definition.md) | Project structure, tooling (clippy, rustfmt), CI/CD |
| [project-management/](project-management/definition.md) | Roadmap documentation, phase-based development |
| [rules/](rules/definition.md) | Workspace rule templates for auto-activating standards |

### Draft — needs refinement

> Specifications exist but have no implementation experience yet. Content may evolve as tooling is built.

| Topic | Description |
|---|---|
| [wasm/](wasm/definition.md) | Component model, toolchain, runtime, security |
| [information-flow-tracking/](information-flow-tracking/definition.md) | Taint labels, propagation rules, sink policies |


## Structure

Each topic contains:
- `definition.md` — scope and when to reference
- Topic-specific guides with implementation standards

## Standards & Conventions

This project follows the foundational specifications from [agent-foundation](../agent-foundation/):

- [Documentation Authoring](../agent-foundation/docs/authoring-guide.md) — frontmatter schema (`title`, `summary`, `read_when`, `status`)
- [Hub Configuration](../agent-foundation/docs/hub-configuration.md) — agentctl.toml format and hub registration

## Index

`index.json` is auto-generated on push to `main` via [`.github/workflows/publish.yml`](.github/workflows/publish.yml) using `agentctl`.

## License

[MIT](LICENSE)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
