---
title: "WASM Security and Sandboxing"
summary: "Security patterns for WASM agent tools including dual metering, capability isolation, and sandbox configuration"
read_when:
  - Implementing secure WASM tool execution
  - Configuring resource limits for untrusted code
  - Setting up capability-based access control
  - Designing sandbox isolation for agent tools
status: active
last_updated: "2025-01-16"
---

# WASM Security and Sandboxing

## Isolation Model

WASM provides strong isolation guarantees for untrusted agent tools:

### Memory Isolation
- Each module gets isolated linear memory
- Cannot access host process memory
- No pointer arithmetic can escape sandbox

### Capability-Based Access
- No implicit system access
- All I/O must be explicitly granted via WASI imports
- Fine-grained permission model

```rust
// Example: Grant only specific capabilities
let mut linker = Linker::new(&engine);
wasi::add_to_linker(&mut linker, |s| s)?;

// Only allow file read, no write
linker.instance("wasi:filesystem/types")?;
linker.func_wrap("wasi:filesystem/preopens", "get-directories", |_| Ok(()))?;
// Omit write functions to deny write access
```

## Dual Metering Pattern

Prevent resource exhaustion with two independent kill switches:

### Fuel Metering (Instruction Count)
```rust
const FUEL_LIMIT: u64 = 1_000_000;  // ~1M WASM instructions

store.set_fuel(FUEL_LIMIT)?;
match instance.call(&mut store, &[]) {
    Err(e) if e.downcast_ref::<Trap>() == Some(&Trap::OutOfFuel) => {
        return Err("Fuel exhausted - too many instructions");
    }
    result => result,
}
```

### Epoch Metering (Wall-Clock Time)
```rust
const EPOCH_DEADLINE: u64 = 2;  // 2 ticks = ~200ms

// Shared watchdog thread
thread::spawn(move || loop {
    thread::sleep(Duration::from_millis(100));
    engine.increment_epoch();
});

store.epoch_deadline_trap();
store.set_epoch_deadline(EPOCH_DEADLINE);
```

### Why Both Are Needed

| Failure Mode | Fuel Catches | Epoch Catches |
|---|---|---|
| Infinite loop `while true {}` | ✓ | ✗ |
| Blocking I/O `sleep(∞)` | ✗ | ✓ |
| CPU-intensive burst | ✓ | ✗ |
| Slow syscalls | ✗ | ✓ |

## WASI Configuration

### Minimal Capability Grant
```rust
// Only grant what the tool actually needs
let mut builder = WasiCtxBuilder::new();

// File access - scoped to specific directory
builder.preopened_dir("/tmp/tool-workspace", ".")?;

// No network access (omit sockets)
// No environment variables (omit inherit_env)
// Limited clock access
builder.inherit_stdout().inherit_stderr();

let wasi = builder.build();
```

### Capability Validation
```rust
// Validate tool manifest against granted capabilities
fn validate_capabilities(manifest: &ToolManifest, wasi_ctx: &WasiCtx) -> Result<()> {
    for required in &manifest.required_capabilities {
        match required {
            "filesystem:read" => ensure_filesystem_read_granted(wasi_ctx)?,
            "network:http" => return Err("Network access not permitted"),
            _ => return Err(format!("Unknown capability: {}", required)),
        }
    }
    Ok(())
}
```

## Security Checklist

### Before Execution
- [ ] Validate WASM bytecode with `wasm-tools validate`
- [ ] Check tool manifest for required capabilities
- [ ] Configure minimal WASI permissions
- [ ] Set fuel and epoch limits

### During Execution
- [ ] Monitor resource consumption
- [ ] Log capability usage
- [ ] Enforce timeout limits
- [ ] Isolate from other tools

### After Execution
- [ ] Clean up temporary files
- [ ] Audit capability usage
- [ ] Measure performance metrics
- [ ] Check for resource leaks

## Common Vulnerabilities

### Resource Exhaustion
```rust
// BAD: No limits
let result = instance.call(&mut store, &[])?;

// GOOD: Dual metering
store.set_fuel(FUEL_LIMIT)?;
store.set_epoch_deadline(EPOCH_DEADLINE);
let result = instance.call(&mut store, &[])?;
```

### Capability Leakage
```rust
// BAD: Inherit all host capabilities
let wasi = WasiCtxBuilder::new().inherit_all().build();

// GOOD: Explicit minimal grants
let wasi = WasiCtxBuilder::new()
    .preopened_dir("/tmp/sandbox", ".")
    .build();
```

### Side Channel Attacks
- Use separate processes for sensitive operations
- Avoid sharing memory between tool instances
- Implement timing attack mitigations

## Performance vs Security Trade-offs

| Configuration | Security | Performance | Use Case |
|---|---|---|---|
| Strict limits, no I/O | High | Fast | Pure compute tools |
| File access, tight limits | Medium | Medium | File processing tools |
| Network access, loose limits | Low | Slow | Integration tools |

Choose the appropriate balance based on tool trust level and requirements.