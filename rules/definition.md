---
title: "Workspace Rules Standards"
summary: "Standards for creating Amazon Q workspace rules that auto-activate relevant documentation."
read_when:
  - Setting up workspace rules
  - Configuring context auto-activation
  - Organizing project standards
status: draft
last_updated: "2026-07-16"
---

# Workspace Rules Standards

## Purpose

Workspace rules automatically include relevant agent-software standards as context when working on specific types of files or tasks, ensuring consistent application of development practices.

## Rule Templates

Ready-to-use rule templates are available in `rules/templates/`:

- [rust-development.md](templates/rust-development.md) — Auto-activate Rust standards
- [python-development.md](templates/python-development.md) — Auto-activate Python standards  
- [version-control.md](templates/version-control.md) — Auto-activate version control standards
- [wasm-development.md](templates/wasm-development.md) — Auto-activate WASM standards
- [information-flow-tracking.md](templates/information-flow-tracking.md) — Auto-activate security standards
- [project-management.md](templates/project-management.md) — Auto-activate project management standards

## Usage

1. Copy relevant templates from `agent-software/rules/templates/` 
2. Place in your project's `.amazonq/rules/` directory
3. Customize activation triggers if needed
4. Rules automatically activate when conditions are met

## Rule Structure

Each rule template includes:
- **Activation Triggers** — File patterns, keywords, or contexts that activate the rule
- **Auto-Include Content** — Specific agent-software documents to include as context
- **Purpose** — Clear explanation of when and why the rule activates
- **Scope** — What types of work benefit from the rule

## Benefits

- **Automatic Context** — Relevant standards available without manual lookup
- **Consistency** — Same practices applied across all projects
- **Efficiency** — No need to remember which standards apply when
- **Maintenance** — Centralized templates, easy updates

## Template Exclusion

Templates are excluded from hub generation via `agentctl.toml`:
```toml
[generate]
ignore = ["rules/templates/"]
```

This keeps templates available for copying while excluding them from published documentation hubs.