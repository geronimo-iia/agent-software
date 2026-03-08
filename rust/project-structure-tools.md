---
title: "Rust Project Structure & Tools"
summary: "Standard Rust project structure using modern tooling: Cargo.toml, clippy, rustfmt, and cargo test."
read_when:
  - Setting up new Rust projects
  - Standardizing Rust project toolchain
  - Configuring modern Rust development environment
  - Establishing Rust project conventions
status: active
last_updated: "2025-01-16"
---

# Rust Project Structure & Tools

## Standard Project Structure

### Binary Project

**Purpose**: Executable applications with optional library code for internal use.

```
project-name/
├── Cargo.toml              # Project configuration
├── Cargo.lock              # Dependency lock file
├── README.md               # Project documentation
├── .gitignore             # Git ignore patterns
├── src/
│   ├── main.rs            # Binary entry point
│   ├── lib.rs             # Library code (optional)
│   └── cli.rs             # CLI module
├── tests/                 # Integration tests
│   └── integration_test.rs
└── benches/               # Benchmarks (optional)
    └── benchmark.rs
```

### Library Project

**Purpose**: Reusable code published as crates for other projects to depend on.

```
project-name/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── src/
│   ├── lib.rs             # Library entry point
│   └── module.rs          # Library modules
├── examples/              # Example usage
│   └── example.rs
└── tests/
    └── integration_test.rs
```

### Workspace Project

**Purpose**: Multiple related crates managed together, sharing dependencies and build configuration.

```
workspace/
├── Cargo.toml             # Workspace configuration
├── crate-a/
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
└── crate-b/
    ├── Cargo.toml
    └── src/
        └── main.rs
```

## Core Toolchain

### Package Management: cargo
- **Dependency management**: Built-in package manager
- **Build system**: Compilation and linking
- **Lock files**: Deterministic builds with Cargo.lock

### Code Quality: clippy
- **Linting**: Catch common mistakes and improve code
- **Best practices**: Idiomatic Rust suggestions
- **Performance**: Optimization hints

### Formatting: rustfmt
- **Code formatting**: Consistent style across projects
- **Configuration**: Customizable via rustfmt.toml
- **IDE integration**: Format on save

### Testing: cargo test
- **Unit tests**: In-module tests with #[test]
- **Integration tests**: Separate tests/ directory
- **Doc tests**: Examples in documentation

## Configuration: Cargo.toml

```toml
[package]
name = "project-name"
version = "0.1.0"
edition = "2021"
authors = ["Author Name <author@example.com>"]
description = "Project description"
license = "MIT OR Apache-2.0"
repository = "https://github.com/user/project"

[dependencies]
clap = { version = "4.0", features = ["derive"] }
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }

[dev-dependencies]
criterion = "0.5"

[[bin]]
name = "project-name"
path = "src/main.rs"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true
```

## Configuration: rustfmt.toml

```toml
edition = "2021"
max_width = 100
tab_spaces = 4
use_small_heuristics = "Default"
imports_granularity = "Crate"
group_imports = "StdExternalCrate"
```

## Configuration: clippy.toml

```toml
# Clippy configuration
avoid-breaking-exported-api = false
```

## Development Workflow

### Setup
```bash
# Create new project
cargo new project-name
cargo new --lib library-name

# Build project
cargo build
cargo build --release
```

### Code Quality
```bash
# Format code
cargo fmt

# Check formatting
cargo fmt -- --check

# Lint with clippy
cargo clippy
cargo clippy -- -D warnings

# Check without building
cargo check
```

### Testing
```bash
# Run all tests
cargo test

# Run specific test
cargo test test_name

# Run with output
cargo test -- --nocapture

# Run doc tests
cargo test --doc
```

### Documentation
```bash
# Generate documentation
cargo doc

# Open documentation in browser
cargo doc --open
```

### Publishing
```bash
# Dry run
cargo publish --dry-run

# Publish to crates.io
cargo publish
```

## Best Practices

### Error Handling
- Use `Result<T, E>` for recoverable errors
- Use `panic!` only for unrecoverable errors
- Implement custom error types with `thiserror`

### CLI Applications
- Use `clap` for argument parsing
- Use `anyhow` for application-level error handling
- Use `env_logger` or `tracing` for logging

### Async Code
- Use `tokio` for async runtime
- Use `async-trait` for async traits
- Avoid blocking operations in async contexts

### Dependencies
- Minimize dependencies
- Pin versions in applications
- Use workspace dependencies for multi-crate projects

### Testing
- Write unit tests alongside code
- Use integration tests for public API
- Use doc tests for examples
- Use `proptest` for property-based testing

## Testing Patterns

### Fixtures

Store static test data in `tests/fixtures/`. Load with `include_str!` or by path:

```rust
// tests/fixtures/config-valid.json
// Load in tests:
let path = Path::new("tests/fixtures/config-valid.json");
let config = Config::load_from(path).unwrap();
```

Prefer fixtures over inline strings for non-trivial data (JSON, TOML, Markdown).

### Function pointer injection

For functions that perform I/O (HTTP, filesystem), define a `Fetcher` type alias and inject it:

```rust
pub type Fetcher = fn(&str) -> Result<String>;

pub fn get_from(dir: &Path, url: &str, fetcher: Fetcher) -> Result<String> {
    // use fetcher instead of direct HTTP call
}

// Production call:
get_from(dir, url, http_fetch)

// Test call:
fn ok_fetcher(_url: &str) -> Result<String> { Ok("...".into()) }
fn err_fetcher(_url: &str) -> Result<String> { Err(anyhow!("fail")) }
get_from(dir, url, ok_fetcher)
```

Prefer function pointers over trait objects for simple injection — no `dyn`, no `Arc`, no `mockall`.

### Isolated filesystem tests

Use `tempfile::tempdir()` to avoid polluting `~/.agentctl` or other real paths:

```rust
// [dev-dependencies]
// tempfile = "3"

#[test]
fn test_writes_cache() {
    let dir = tempfile::tempdir().unwrap();
    refresh_to(dir.path(), "http://example.com", ok_fetcher).unwrap();
    assert!(dir.path().join("index.json").exists());
}
```

### Integration test isolation via CLI flag

Expose a `--config <path>` global flag so integration tests can point to a temp config:

```rust
fn with_config(dir: &Path) -> Vec<&str> {
    vec!["--config", dir.join("config.json").to_str().unwrap()]
}

// Usage:
Command::cargo_bin("agentctl")
    .args(with_config(&dir))
    .args(["hub", "list"])
    .assert().success();
```

This keeps integration tests fully isolated from the user's real config.

## Common Dependencies

### CLI
```toml
clap = { version = "4.0", features = ["derive"] }
anyhow = "1.0"
env_logger = "0.11"
```

### Serialization
```toml
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
toml = "0.8"
```

### Async
```toml
tokio = { version = "1.0", features = ["full"] }
async-trait = "0.1"
```

### HTTP
```toml
reqwest = { version = "0.11", features = ["json"] }
axum = "0.7"
```

### Testing
```toml
[dev-dependencies]
criterion = "0.5"
proptest = "1.0"
mockall = "0.12"
```
