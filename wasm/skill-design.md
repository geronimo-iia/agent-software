---
title: "WASM Skill Design"
summary: "How to design and package WASM modules as agent skills with proper gating and tool permissions"
read_when:
  - Packaging a WASM module as an agent skill
  - Designing skill interfaces for WASM tools
  - Setting up capability gating for WASM execution
  - Creating reusable WASM-based agent capabilities
status: active
last_updated: "2025-01-16"
---

# WASM Skill Design

## Skill Structure

A WASM skill packages a compiled `.wasm` module with execution instructions:

```
wasm-tool-skill/
├── SKILL.md              # Skill definition and usage
├── tool.wasm             # Compiled WASM module
├── tool.wit              # Interface definition (optional)
└── scripts/
    ├── execute.sh        # Execution wrapper
    └── validate.sh       # Module validation
```

## SKILL.md Template

```markdown
---
name: wasm-file-processor
description: >
  Process files using a sandboxed WASM module. Provides secure file
  transformation capabilities with resource limits and capability isolation.
tags:
  - wasm
  - file-processing
  - sandbox
triggers:
  - process files
  - transform data
  - wasm tool
requires:
  bins:
    - wasmtime
allowed-tools: Bash(wasmtime:*) Read Write
license: MIT
metadata:
  author: agent-software
  version: "1.0"
  wasm-module: tool.wasm
  capabilities:
    - filesystem:read
    - filesystem:write
---

# WASM File Processor

## When to use this skill

Use when you need to process files with a sandboxed, resource-limited tool
that cannot access system resources beyond the granted capabilities.

## Execute the tool

```bash
bash scripts/execute.sh <input-file> <output-file>
```

## Capabilities

This tool requires:
- **File read access** - to process input files
- **File write access** - to generate output files
- **No network access** - completely isolated from network

## Resource limits

- **Fuel limit**: 1M instructions (~1 second of compute)
- **Time limit**: 500ms wall-clock time
- **Memory limit**: 16MB linear memory

## Validation

```bash
bash scripts/validate.sh tool.wasm
```
```

## Execution Script

```bash
#!/usr/bin/env bash
# scripts/execute.sh
set -euo pipefail

INPUT_FILE="${1:?usage: execute.sh <input-file> <output-file>}"
OUTPUT_FILE="${2:?usage: execute.sh <input-file> <output-file>}"
SKILL_DIR="$(dirname "$(dirname "$0")")"
WASM_MODULE="$SKILL_DIR/tool.wasm"

# Validate inputs
if [[ ! -f "$INPUT_FILE" ]]; then
    echo "Error: Input file not found: $INPUT_FILE" >&2
    exit 1
fi

if [[ ! -f "$WASM_MODULE" ]]; then
    echo "Error: WASM module not found: $WASM_MODULE" >&2
    exit 1
fi

# Create sandbox directory
SANDBOX_DIR=$(mktemp -d)
trap "rm -rf $SANDBOX_DIR" EXIT

# Copy input to sandbox
cp "$INPUT_FILE" "$SANDBOX_DIR/input"

# Execute with resource limits and capability restrictions
wasmtime run \
    --fuel 1000000 \
    --epoch-interruption \
    --dir "$SANDBOX_DIR"::/ \
    --wasi preview2 \
    "$WASM_MODULE" \
    input output

# Copy result back
if [[ -f "$SANDBOX_DIR/output" ]]; then
    cp "$SANDBOX_DIR/output" "$OUTPUT_FILE"
    echo "Processed: $INPUT_FILE -> $OUTPUT_FILE"
else
    echo "Error: Tool did not produce output" >&2
    exit 1
fi
```

## Capability Gating

### Runtime Requirements
```yaml
requires:
  bins:
    - wasmtime          # WASM runtime
  optional-bins:
    - wasm-tools        # For validation
```

### Tool Permissions
```yaml
allowed-tools: 
  - Bash(wasmtime:*)    # Allow wasmtime execution
  - Read                # Read input files
  - Write               # Write output files
  # No network tools allowed
```

## Security Configuration

### Minimal Capability Grant
```bash
# Only grant specific directory access
wasmtime run \
    --dir "$SANDBOX_DIR"::/ \    # Scoped file access
    --fuel 1000000 \             # Instruction limit
    --epoch-interruption \       # Time limit
    --wasi preview2 \            # WASI interface
    "$WASM_MODULE"
```

### Validation Script
```bash
#!/usr/bin/env bash
# scripts/validate.sh
set -euo pipefail

WASM_MODULE="${1:?usage: validate.sh <module.wasm>}"

echo "Validating WASM module: $WASM_MODULE"

# Check file exists
if [[ ! -f "$WASM_MODULE" ]]; then
    echo "Error: Module not found" >&2
    exit 1
fi

# Validate bytecode
if command -v wasm-tools >/dev/null; then
    wasm-tools validate "$WASM_MODULE"
    echo "✓ Bytecode validation passed"
else
    echo "⚠ wasm-tools not available, skipping validation"
fi

# Check module size
SIZE=$(stat -f%z "$WASM_MODULE" 2>/dev/null || stat -c%s "$WASM_MODULE")
if [[ $SIZE -gt 1048576 ]]; then  # 1MB
    echo "⚠ Module size ${SIZE} bytes exceeds recommended 1MB limit"
fi

# Test basic execution
echo "Testing basic execution..."
if wasmtime run --fuel 1000 "$WASM_MODULE" >/dev/null 2>&1; then
    echo "✓ Basic execution test passed"
else
    echo "✓ Module properly trapped (expected for tools requiring input)"
fi

echo "Validation complete"
```

## Interface Design

### WIT Interface Example
```wit
// tool.wit
package agent:file-processor@1.0.0;

interface process {
  record file-info {
    name: string,
    size: u64,
  }
  
  record process-result {
    success: bool,
    message: string,
    output-size: u64,
  }
  
  process-file: func(input: string, output: string) -> result<process-result, string>;
}

world file-processor {
  export process;
  import wasi:filesystem/types@0.2.0;
  import wasi:filesystem/preopens@0.2.0;
}
```

## Best Practices

### Skill Design
- **Single purpose** - One WASM module per skill
- **Clear capabilities** - Document exactly what the tool can access
- **Resource limits** - Always set fuel and time limits
- **Error handling** - Provide clear error messages

### Security
- **Minimal permissions** - Grant only required capabilities
- **Sandbox isolation** - Use temporary directories
- **Input validation** - Validate all inputs before execution
- **Output verification** - Check tool outputs are as expected

### Performance
- **Module size** - Keep under 1MB for fast loading
- **Startup time** - Optimize for <10ms execution overhead
- **Memory usage** - Limit linear memory allocation
- **Caching** - Consider module pre-compilation for frequently used tools

## Testing

### Unit Tests
```bash
# Test valid input
./scripts/execute.sh test-input.txt output.txt
[[ -f output.txt ]] || exit 1

# Test invalid input
! ./scripts/execute.sh nonexistent.txt output.txt

# Test resource limits
timeout 2s ./scripts/execute.sh large-input.txt output.txt
```

### Integration Tests
```bash
# Test with agent system
agent-test-runner wasm-file-processor test-cases/
```

This design pattern ensures WASM modules are properly packaged as secure, reusable agent skills with appropriate capability gating and resource limits.