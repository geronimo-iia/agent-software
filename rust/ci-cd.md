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

**.github/workflows/publish.yml**

```yaml
name: Publish

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: dtolnay/rust-toolchain@stable
      
      - name: Publish to crates.io
        run: cargo publish --token ${{ secrets.CARGO_TOKEN }}
```

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

```yaml
- name: Security audit
  uses: rustsec/audit-check@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
```

## Benchmarking

```yaml
- name: Run benchmarks
  run: cargo bench --no-fail-fast
```

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
