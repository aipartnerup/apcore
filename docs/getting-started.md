# Getting Started

apcore defines the **Cognitive Interface** for your application. It ensures your logic is naturally perceivable by AI Agents through enforced schemas, behavioral guardrails, and self-healing error guidance.

Whether you are building from scratch or upgrading a legacy system, apcore provides a path for you. See [Choose Your Integration Path](./guides/creating-modules.md#choose-your-integration-path) for details.

## 1. Installation

=== "Python"

    ```bash
    pip install apcore
    ```

    Requires Python 3.11+.

=== "TypeScript"

    ```bash
    npm install apcore-js
    ```

    Requires Node.js 18+ and TypeScript 5.5+.

## 2. Your First Module

The `APCore` client is the recommended entry point. It manages Registry and Executor for you.

=== "Python"

    ```python
    from apcore import APCore

    client = APCore()

    @client.module(id="math.add", description="Add two numbers", tags=["math"])
    def add(a: int, b: int) -> dict:
        return {"sum": a + b}

    # Call the module
    result = client.call("math.add", {"a": 10, "b": 5})
    print(result)  # {'sum': 15}
    ```

    For simple scripts, you can also use the global `apcore.*` functions directly:

    ```python
    import apcore

    @apcore.module(id="math.add", description="Add two numbers")
    def add(a: int, b: int) -> dict:
        return {"sum": a + b}

    result = apcore.call("math.add", {"a": 10, "b": 5})
    ```

=== "TypeScript"

    ```typescript
    import { Type } from '@sinclair/typebox';
    import { APCore } from 'apcore-js';

    const client = new APCore();

    client.module({
      id: 'math.add',
      description: 'Add two numbers',
      tags: ['math'],
      inputSchema: Type.Object({ a: Type.Number(), b: Type.Number() }),
      outputSchema: Type.Object({ sum: Type.Number() }),
      execute: (inputs) => ({ sum: (inputs.a as number) + (inputs.b as number) }),
    });

    const result = await client.call('math.add', { a: 10, b: 5 });
    console.log(result); // { sum: 15 }
    ```

## 3. Preflight Validation

Before executing, you can validate inputs without running the module:

=== "Python"

    ```python
    preflight = client.validate("math.add", {"a": 10, "b": 5})
    if preflight.valid:
        print("All checks passed")
    else:
        print(f"Errors: {preflight.errors}")
    ```

=== "TypeScript"

    ```typescript
    const preflight = client.validate('math.add', { a: 10, b: 5 });
    if (preflight.valid) {
      console.log('All checks passed');
    } else {
      console.log('Errors:', preflight.errors);
    }
    ```

## 4. Streaming

Modules that implement `stream()` can yield output chunks incrementally:

=== "Python"

    ```python
    async for chunk in client.stream("my.streaming_module", {"query": "hello"}):
        print(chunk)
    ```

=== "TypeScript"

    ```typescript
    for await (const chunk of client.stream('my.streaming_module', { query: 'hello' })) {
      console.log(chunk);
    }
    ```

## 5. Auto-Discovery (Directory as ID)

Instead of manual registration, apcore can automatically discover modules in a directory.

=== "Python"

    Project structure:
    ```text
    my-project/
    └── extensions/
        └── math/
            └── add.py  # Contains the module
    ```

    ```python
    from apcore import APCore

    client = APCore()
    client.discover()
    # Module ID is automatically "math.add"
    result = client.call("math.add", {"a": 10, "b": 5})
    ```

=== "TypeScript"

    Project structure:
    ```text
    my-project/
    └── extensions/
        └── math/
            └── add.ts  # Export default or named module
    ```

    ```typescript
    import { APCore } from 'apcore-js';

    const client = new APCore();
    await client.discover();
    // Module ID is automatically "math.add"
    const result = await client.call('math.add', { a: 10, b: 5 });
    ```

## 6. Adding Middleware

Middleware can intercept calls for logging, tracing, or security.

=== "Python"

    ```python
    from apcore import LoggingMiddleware, TracingMiddleware

    client.use(LoggingMiddleware())
    client.use(TracingMiddleware())
    ```

=== "TypeScript"

    ```typescript
    import { ObsLoggingMiddleware } from 'apcore-js';

    client.use(new ObsLoggingMiddleware());
    ```

## 7. Advanced: Manual Registry + Executor

For full control, you can manage Registry and Executor separately:

=== "Python"

    ```python
    from apcore import Registry, Executor, module

    registry = Registry(extensions_dir="./extensions")
    registry.discover()

    executor = Executor(registry=registry)
    result = executor.call("math.add", {"a": 10, "b": 5})
    ```

=== "TypeScript"

    ```typescript
    import { Registry, Executor } from 'apcore-js';

    const registry = new Registry({ extensionsDir: './extensions' });
    await registry.discover();

    const executor = new Executor({ registry });
    const result = await executor.call('math.add', { a: 10, b: 5 });
    ```

## Next Steps

- [Creating Modules Guide](guides/creating-modules.md) - Deep dive into module definition.
- [Schema Definition Guide](guides/schema-definition.md) - Learn about advanced validation.
- [ACL Configuration](guides/acl-configuration.md) - Secure your modules.
- [apcore-mcp](https://github.com/aipartnerup/apcore-mcp) - Expose these modules as an MCP Server.
