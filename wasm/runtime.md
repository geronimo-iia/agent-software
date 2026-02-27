---
title: "WASM Runtime Integration"
summary: "How to integrate and execute WASM modules in agent systems using Wasmtime"
read_when:
  - Integrating WASM execution into agent runtime
  - Setting up Wasmtime host bindings
  - Configuring WASM module lifecycle management
status: active
last_updated: "2025-01-16"
---

# WASM Runtime Integration

## Wasmtime Setup

### Basic Runtime Configuration
```rust
use wasmtime::*;

fn create_engine() -> Result<Engine> {
    let mut config = Config::new();
    config.consume_fuel(true);           // Enable fuel metering
    config.epoch_interruption(true);    // Enable epoch interruption
    config.wasm_component_model(true);  // Enable Component Model
    Engine::new(&config)
}
```

### Store and Module Lifecycle
```rust
async fn execute_tool(engine: &Engine, wasm_bytes: &[u8], input: &str) -> Result<String> {
    // Create isolated store per execution
    let mut store = Store::new(engine, ());
    
    // Set resource limits
    store.set_fuel(1_000_000)?;
    store.set_epoch_deadline(2);
    
    // Load and instantiate module
    let module = Module::new(engine, wasm_bytes)?;
    let instance = Instance::new(&mut store, &module, &[])?;
    
    // Call exported function
    let run_func = instance.get_typed_func::<(String,), String>(&mut store, "execute")?;
    let result = run_func.call(&mut store, (input.to_string(),))?;
    
    Ok(result)
}
```

## WASI Integration

### Capability Configuration
```rust
use wasmtime_wasi::WasiCtxBuilder;

fn create_wasi_context(tool_workspace: &Path) -> Result<WasiCtx> {
    WasiCtxBuilder::new()
        .preopened_dir(tool_workspace, ".")?  // Scoped file access
        .inherit_stdout()
        .inherit_stderr()
        .build()
}
```

### Host Function Bindings
```rust
// Custom host functions for agent tools
fn add_agent_functions(linker: &mut Linker<()>) -> Result<()> {
    linker.func_wrap("env", "log_message", |message: String| {
        println!("Tool log: {}", message);
    })?;
    
    linker.func_wrap("env", "get_timestamp", || -> u64 {
        std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap()
            .as_secs()
    })?;
    
    Ok(())
}
```

## Component Model Integration

### Loading Components
```rust
use wasmtime::component::*;

async fn execute_component(engine: &Engine, component_bytes: &[u8]) -> Result<()> {
    let component = Component::new(engine, component_bytes)?;
    let mut store = Store::new(engine, ());
    
    let mut linker = Linker::new(engine);
    wasmtime_wasi::add_to_linker_async(&mut linker)?;
    
    let instance = linker.instantiate_async(&mut store, &component).await?;
    
    // Call component exports
    let run = instance.get_typed_func::<(), ()>(&mut store, "run")?;
    run.call_async(&mut store, ()).await?;
    
    Ok(())
}
```

## Error Handling

### Trap Classification
```rust
fn handle_execution_error(error: Error) -> String {
    match error.downcast_ref::<Trap>() {
        Some(Trap::OutOfFuel) => "Tool exceeded instruction limit".to_string(),
        Some(Trap::EpochDeadlineReached) => "Tool exceeded time limit".to_string(),
        Some(Trap::MemoryOutOfBounds) => "Tool accessed invalid memory".to_string(),
        _ => format!("Tool execution failed: {}", error),
    }
}
```

## Performance Optimization

### Module Pre-compilation
```rust
// Pre-compile modules for faster startup
fn precompile_module(engine: &Engine, wasm_bytes: &[u8]) -> Result<Vec<u8>> {
    engine.precompile_module(wasm_bytes)
}

// Load pre-compiled module
fn load_precompiled(engine: &Engine, compiled_bytes: &[u8]) -> Result<Module> {
    unsafe { Module::deserialize(engine, compiled_bytes) }
}
```

### Instance Pooling
```rust
use wasmtime::InstancePre;

struct ToolPool {
    instance_pre: InstancePre<()>,
    max_instances: usize,
}

impl ToolPool {
    fn new(engine: &Engine, module: &Module, max_instances: usize) -> Result<Self> {
        let mut linker = Linker::new(engine);
        // Configure linker...
        
        let instance_pre = linker.instantiate_pre(module)?;
        Ok(Self { instance_pre, max_instances })
    }
    
    async fn execute(&self, store: &mut Store<()>, input: &str) -> Result<String> {
        let instance = self.instance_pre.instantiate_async(store).await?;
        // Execute...
        Ok("result".to_string())
    }
}
```