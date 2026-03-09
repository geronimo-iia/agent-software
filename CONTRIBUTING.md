# Contributing

## Adding or updating a document

All `.md` files (except `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `ARCHIVED.md`) must include YAML frontmatter. Follow the authoring standard:

- [Documentation Authoring](https://geronimo-iia.github.io/agent-foundation/docs/authoring-guide/) — required fields (`title`, `summary`, `read_when`), field rules, and examples
- [Specification Organization](https://geronimo-iia.github.io/agent-foundation/docs/organization/) — where to place new documents, naming conventions, domain placement workflow

## Adding a new topic

1. Create a new folder with a `definition.md` — define the domain scope first
2. Add topic-specific documents following the authoring guide
3. Update the Topics table in `README.md`

## Commit format

Follow the semantic commit standard: [git-commit-semantic.md](version-control-release/git-commit-semantic.md)

## Index

`index.json` is auto-generated — do not edit it manually. It is regenerated on every push to `main`.
