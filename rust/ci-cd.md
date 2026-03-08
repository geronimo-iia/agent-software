---
title: "Rust CI/CD"
summary: "Continuous integration and deployment pipelines for Rust projects using GitHub Actions."
read_when:
  - Setting up CI/CD for Rust projects
  - Configuring automated testing and releases
  - Establishing Rust deployment workflows
status: active
last_updated: "2025-01-16"
---

# Rust CI/CD

## GitHub Actions Workflow

### Basic CI Pipeline

**.github/workflows/ci.yml**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  CARGO_TERM_COLOR: always

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: dtolnay/rust-toolchain@stable
      
      - uses: Swatinem/rust-cache@v2
      
      - name: Check formatting
        run: cargo fmt -- --check
      
      - name: Clippy
        run: cargo clippy -- -D warnings
      
      - name: Build
        run: cargo build --verbose
      
      - name: Run tests
        run: cargo test --verbose
```

### Multi-Platform Testing

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        rust: [stable, beta]
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.rust }}
      
      - uses: Swatinem/rust-cache@v2
      
      - run: cargo test --verbose
```

### Release Pipeline

**.github/workflows/release.yml**

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            target: x86_64-unknown-linux-gnu
          - os: macos-latest
            target: x86_64-apple-darwin
          - os: macos-latest
            target: aarch64-apple-darwin
          - os: windows-latest
            target: x86_64-pc-windows-msvc
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.target }}
      
      - uses: Swatinem/rust-cache@v2
      
      - name: Build release
        run: cargo build --release --target ${{ matrix.target }}
      
      - name: Package binary
        shell: bash
        run: |
          cd target/${{ matrix.target }}/release
          if [ "${{ matrix.os }}" = "windows-latest" ]; then
            7z a ../../../${{ matrix.target }}.zip *.exe
          else
            tar czf ../../../${{ matrix.target }}.tar.gz *
          fi
      
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.target }}
          path: ${{ matrix.target }}.*
  
  release:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      contents: write
    
    steps:
      - uses: actions/download-artifact@v4
      
      - name: Create release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            */*.tar.gz
            */*.zip
```

### Cargo Publish

Add as a `publish` job after `release` in `release.yml`:

```yaml
  publish:
    needs: release
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - name: Publish to crates.io
        env:
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_TOKEN }}
        run: cargo publish --locked
```

> Note: `--token` flag is deprecated — use `CARGO_REGISTRY_TOKEN` env var instead.

## Coverage

### Using tarpaulin

```yaml
- name: Install tarpaulin
  run: cargo install cargo-tarpaulin

- name: Generate coverage
  run: cargo tarpaulin --out Xml

- name: Upload coverage
  uses: codecov/codecov-action@v3
```

## Security Audit

Add to the CI job after tests:

```yaml
      - name: Security audit
        run: cargo audit
```

Requires `cargo-audit` installed on the runner. Add to CI setup:

```yaml
      - name: Install cargo-audit
        run: cargo install cargo-audit --locked
```

## Benchmarking

```yaml
- name: Run benchmarks
  run: cargo bench --no-fail-fast
```

## Distribution

### Channels

| Channel | Install command | Requires Rust? | Best for |
|---|---|---|---|
| [crates.io](https://crates.io) | `cargo install <name>` | Yes | Developers |
| GitHub Releases | download binary | No | All users |
| [cargo-binstall](https://github.com/cargo-bins/cargo-binstall) | `cargo binstall <name>` | No | Developers |
| Homebrew tap | `brew install <tap>/<name>` | No | macOS/Linux users |

Recommended: publish to crates.io + GitHub Releases binaries. `cargo-binstall` then works automatically.

### What you need to publish

**crates.io**:
1. Create account at [crates.io](https://crates.io) (sign in with GitHub)
2. Generate API token: Account Settings → API Tokens → New Token (scope: `publish-new`, `publish-update`)
3. Add token as GitHub Actions secret: repo → Settings → Secrets → `CARGO_TOKEN`
4. Ensure `Cargo.toml` has `description`, `license`, `repository` fields — required by crates.io

**GitHub Releases** (already covered by release.yml above — no extra setup needed).

**Homebrew tap** (after first release):
1. Create a GitHub repo named `homebrew-agent` under your account
2. Add a Formula file `agentctl.rb` referencing the GitHub Release tarball and its SHA256
3. Users install with `brew tap <user>/agent && brew install agentctl`
4. The tap scales to future agent-* binaries — one repo, multiple formulas

### Cargo.toml required fields for crates.io

```toml
[package]
name = "agent-ctl"
version = "0.1.0"
edition = "2021"
description = "CLI for agent hub validation, index generation, and skill management"
license = "MIT OR Apache-2.0"
repository = "https://github.com/<user>/agentctl"
readme = "README.md"
```

### cargo-binstall integration

Add to `Cargo.toml` so `cargo binstall` knows where to find binaries:

```toml
[package.metadata.binstall]
pkg-url = "{ repo }/releases/download/v{ version }/{ target }.tar.gz"
bin-dir = "agentctl"
```

---

## Release Checklist

Before tagging a release:

1. Bump `version` in `Cargo.toml`
2. Update `CHANGELOG.md` — move `[Unreleased]` to `[x.y.z] - YYYY-MM-DD`
3. Commit: `chore: bump version to x.y.z`
4. Tag: `git tag vx.y.z && git push origin vx.y.z`

> Tagging before bumping `Cargo.toml` causes `cargo publish` to fail with "crate already exists" if the previous version was already published.

## Best Practices

### Caching
- Use `Swatinem/rust-cache@v2` for dependency caching
- Significantly speeds up CI runs

### Matrix Testing
- Test on multiple OS (Linux, macOS, Windows)
- Test on stable and beta Rust versions
- Test with different feature flags

### Release Automation
- Tag-based releases (`v*` pattern)
- Cross-platform binary builds
- Automatic GitHub release creation

### Security
- Run `cargo audit` in CI
- Use Dependabot for dependency updates
- Pin action versions with SHA

### Performance
- Use `--locked` flag to ensure Cargo.lock consistency
- Run clippy with `-D warnings` to fail on warnings
- Use release profile for benchmarks
