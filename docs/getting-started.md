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

The easiest way to create a module is using the `@module` decorator (Python) or the `FunctionModule` class (TypeScript).

=== "Python"

    ```python
    from apcore import module, Registry

    registry = Registry()

    @module(id="math.add", description="Add two numbers", tags=["math"], registry=registry)
    def add(a: int, b: int) -> dict:
        return {"sum": a + b}
    ```

=== "TypeScript"

    ```typescript
    import { Type } from '@sinclair/typebox';
    import { FunctionModule } from 'apcore-js';

    const add = new FunctionModule({
      moduleId: 'math.add',
      description: 'Add two numbers',
      tags: ['math'],
      inputSchema: Type.Object({ a: Type.Number(), b: Type.Number() }),
      outputSchema: Type.Object({ sum: Type.Number() }),
      execute: (inputs) => ({ sum: (inputs.a as number) + (inputs.b as number) }),
    });
    ```

## 3. Registry and Execution

Modules are stored in a **Registry** and called through an **Executor**. The Executor handles validation, access control, and middleware.

=== "Python"

    ```python
    from apcore import Executor

    # Registry already has "math.add" registered via @module(registry=...)
    executor = Executor(registry=registry)

    result = executor.call("math.add", {"a": 10, "b": 5})
    print(result)  # {'sum': 15}
    ```

=== "TypeScript"

    ```typescript
    import { Registry, Executor } from 'apcore-js';

    // 1. Register the module
    const registry = new Registry();
    registry.register('math.add', add);

    // 2. Create an executor
    const executor = new Executor({ registry });

    // 3. Call the module
    const result = await executor.call('math.add', { a: 10, b: 5 });
    console.log(result); // { sum: 15 }
    ```

## 4. Auto-Discovery (Directory as ID)

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
    registry = Registry(extensions_dir="./extensions")
    registry.discover()
    # Module ID is automatically "math.add"
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
    const registry = new Registry({ extensionsDir: './extensions' });
    await registry.discover();
    // Module ID is automatically "math.add"
    ```

## 5. Adding Middleware

Middleware can intercept calls for logging, tracing, or security.

=== "Python"

    ```python
    from apcore import LoggingMiddleware

    executor.use(LoggingMiddleware())
    ```

=== "TypeScript"

    ```typescript
    import { ObsLoggingMiddleware } from 'apcore-js';

    executor.use(new ObsLoggingMiddleware());
    ```

## Next Steps

- [Creating Modules Guide](guides/creating-modules.md) - Deep dive into module definition.
- [Schema Definition Guide](guides/schema-definition.md) - Learn about advanced validation.
- [ACL Configuration](guides/acl-configuration.md) - Secure your modules.
- [apcore-mcp](https://github.com/aipartnerup/apcore-mcp) - Expose these modules as an MCP Server.
