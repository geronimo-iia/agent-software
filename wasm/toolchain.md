---
title: "WASM Toolchain Setup"
summary: "Standard toolchain configuration for WebAssembly development with Rust"
read_when:
  - Setting up a new WASM development environment
  - Configuring build tools for WASM modules
  - Troubleshooting WASM compilation issues
status: active
last_updated: "2025-01-16"
---

# WASM Toolchain Setup

## Required Tools

### Rust Toolchain
```bash
# Install Rust with rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add WASM targets
rustup target add wasm32-wasip2      # WASI interface - file, network, clock access
rustup target add wasm32-unknown-unknown  # Pure compute, no I/O
```

### WASM Tools
```bash
# Install wasm-opt for binary size optimization (20-60% reduction)
cargo install wasm-opt

# Install wasm-tools for validation and inspection
cargo install wasm-tools

# Install wit-bindgen for component model bindings
cargo install wit-bindgen-cli
```

## Target Selection

| Target | Use case |
|---|---|
| `wasm32-wasip2` | Agent tools needing system access (files, network, time) |
| `wasm32-unknown-unknown` | Pure compute modules with no I/O requirements |

## Project Configuration

### Cargo.toml
```toml
[package]
name = "agent-tool"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]  # Required for WASM modules

[dependencies]
wit-bindgen = "0.24"     # Component Model bindings

[package.metadata.component]
package = "agent:tool"
```

### Build Configuration
```bash
# Build for WASM target
cargo build --target wasm32-wasip2 --release

# Optimize WASM binary (reduces size 20-60%)
wasm-opt -O3 --enable-bulk-memory target/wasm32-wasip2/release/agent_tool.wasm -o dist/agent_tool.wasm

# Validate WASM module
wasm-tools validate dist/agent_tool.wasm

# Inspect module structure
wasm-tools print dist/agent_tool.wasm
```

## Docker Build Environment

### Dockerfile
```dockerfile
# Dockerfile.wasm-builder
FROM rust:1.82-slim

RUN rustup target add wasm32-wasip2 \
    && cargo install wasm-opt wasm-tools wit-bindgen-cli \
    && rm -rf /usr/local/cargo/registry

WORKDIR /build
ENTRYPOINT ["cargo", "build", "--target", "wasm32-wasip2", "--release"]
```

### Build Script
```bash
#!/usr/bin/env bash
# scripts/build-wasm.sh
set -euo pipefail

PROJECT_DIR="${1:?usage: build-wasm.sh <project-dir>}"
OUTPUT_DIR="${2:-./dist}"

mkdir -p "$OUTPUT_DIR"

# Build with Docker for reproducible environment
docker run --rm \
    -v "$(realpath "$PROJECT_DIR")":/build \
    -v cargo-cache:/usr/local/cargo/registry \
    wasm-builder:latest

# Copy and optimize output
WASM_FILE=$(find "$PROJECT_DIR/target/wasm32-wasip2/release" -name "*.wasm" | head -1)
wasm-opt -O3 "$WASM_FILE" -o "$OUTPUT_DIR/$(basename "$WASM_FILE")"

echo "Built: $OUTPUT_DIR/$(basename "$WASM_FILE")"
```

## Development Workflow

1. **Setup** - Install toolchain and create project structure
2. **Define interfaces** - Write WIT files for component interfaces
3. **Generate bindings** - Use wit-bindgen to create Rust bindings
4. **Implement** - Write Rust code using generated bindings
5. **Build** - Compile to WASM with optimization
6. **Test** - Validate module functionality and interfaces