---
title: "WASM Component Model"
summary: "WIT interface conventions and wit-bindgen usage for agent WASM modules"
read_when:
  - Designing interfaces for WASM agent tools
  - Implementing WASM component bindings
  - Integrating WASM modules with agent systems
status: active
last_updated: "2025-01-16"
---

# WASM Component Model

## WIT Interface Conventions

### Agent Tool Interface
```wit
// wit/agent-tool.wit
package agent:tool@0.1.0;

interface execute {
  record input {
    parameters: string,
    context: option<string>,
  }
  
  record output {
    result: string,
    status: status,
    metadata: option<string>,
  }
  
  enum status {
    success,
    error,
    partial,
  }
  
  execute: func(input: input) -> result<output, string>;
}

world agent-tool {
  export execute;
}
```

### File Operations Interface
```wit
// wit/file-ops.wit
interface file-operations {
  read-file: func(path: string) -> result<string, string>;
  write-file: func(path: string, content: string) -> result<unit, string>;
  list-directory: func(path: string) -> result<list<string>, string>;
}
```

## Rust Implementation

### Generated Bindings Usage
```rust
// src/lib.rs
use wit_bindgen::generate!({
    world: "agent-tool",
    path: "wit",
});

struct AgentTool;

impl Guest for AgentTool {
    fn execute(input: Input) -> Result<Output, String> {
        // Tool implementation
        let result = process_parameters(&input.parameters)?;
        
        Ok(Output {
            result,
            status: Status::Success,
            metadata: Some("execution completed".to_string()),
        })
    }
}

export!(AgentTool);

fn process_parameters(params: &str) -> Result<String, String> {
    // Implementation logic
    Ok(format!("Processed: {}", params))
}
```

## Interface Design Patterns

### Input/Output Structure
- **Structured data** - Use records for complex parameters
- **Error handling** - Return `result<T, string>` for fallible operations
- **Metadata** - Include optional metadata for debugging/logging
- **Status codes** - Use enums for operation status

### Resource Management
```wit
interface resource-manager {
  resource handle;
  
  create-handle: func(config: string) -> result<handle, string>;
  use-handle: func(h: borrow<handle>) -> result<string, string>;
  close-handle: func(h: handle) -> result<unit, string>;
}
```

## Best Practices

- **Minimal interfaces** - Keep WIT definitions focused and minimal
- **Versioning** - Use semantic versioning in package declarations
- **Error messages** - Provide descriptive error strings
- **Documentation** - Comment WIT interfaces thoroughly
- **Resource management** - Use explicit resource handles for stateful operations

## Security Considerations

### Capability-Based Access
WASM modules have no implicit capabilities. Every system interaction must be explicitly granted:

```wit
// Only grant specific capabilities needed
interface file-access {
  // Read-only access to specific directory
  read-file: func(path: string) -> result<string, string>;
  // No write capability granted
}
```

### Isolation Guarantees
- **Memory isolation** - Module cannot access host process memory
- **No implicit I/O** - All system access must be explicitly imported
- **Control flow integrity** - Bytecode is validated before execution
- **Deterministic execution** - Same inputs produce same outputs

## Resource Limits

WASM isolation does not include resource limits. Implement dual metering:

```rust
// Fuel limit - instruction count
store.set_fuel(1_000_000)?;

// Epoch deadline - wall-clock time
store.set_epoch_deadline(2); // 2 ticks = ~200ms
```