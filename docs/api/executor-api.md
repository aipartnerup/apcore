# Executor API

> **Canonical Definition** - This document is the authoritative definition of the Executor interface

> Executor is responsible for module execution, Context management, middleware scheduling, and observability.

## 1. Interface Overview

```python
from typing import Any
from apcore import Registry, Context, Middleware


class Executor:
    """Module executor"""

    def __init__(
        self,
        registry: Registry,
        middlewares: list[Middleware] | None = None,
        acl: "ACL | None" = None,
        approval_handler: "ApprovalHandler | None" = None
    ) -> None:
        """
        Initialize Executor

        Args:
            registry: Module registry
            middlewares: Middleware list (executed in order)
            acl: Access Control List
            approval_handler: Approval handler for modules with requires_approval=true
        """
        ...

    # ============ Execution Methods ============

    def call(
        self,
        module_id: str,
        inputs: dict[str, Any] | None = None,
        context: Context | None = None
    ) -> dict[str, Any]:
        """
        Synchronously call module

        Args:
            module_id: Module ID
            inputs: Input parameters
            context: Call context (optional, auto-created)

        Returns:
            Module output

        Raises:
            ModuleNotFoundError: Module does not exist
            ValidationError: Input/output validation failed
            ACLDeniedError: Insufficient permissions
            ApprovalDeniedError: Approval explicitly rejected
            ApprovalTimeoutError: Approval timed out
            ApprovalPendingError: Approval pending, retry with _approval_token (Phase B)
            CallDepthExceededError: Call chain depth exceeded
            CircularCallError: Circular call detected
            CallFrequencyExceededError: Same module call frequency exceeded
            ModuleError: Module execution error
        """
        ...

    async def call_async(
        self,
        module_id: str,
        inputs: dict[str, Any] | None = None,
        context: Context | None = None
    ) -> dict[str, Any]:
        """
        Asynchronously call module

        Note: Modules only need to define one execute() method (def or async def),
        framework auto-detects and selects appropriate calling method. call_async is used
        to call modules in async context.
        """
        ...

    async def stream(
        self,
        module_id: str,
        inputs: dict[str, Any] | None = None,
        context: Context | None = None
    ) -> AsyncIterator[dict[str, Any]]:
        """
        Stream module output chunk by chunk

        Yields partial results as they become available.
        Calls the module's stream() method if defined,
        otherwise falls back to standard execute().
        """
        ...

    def validate(
        self,
        module_id: str,
        inputs: dict[str, Any]
    ) -> "ValidationResult":
        """Only validate inputs, don't execute"""
        ...

    # ============ Middleware Management ============

    def use(self, middleware: Middleware) -> "Executor":
        """Add middleware (Class-based)"""
        ...

    def use_before(self, callback: "Callable") -> "Executor":
        """Add before function middleware (Function-first API)"""
        ...

    def use_after(self, callback: "Callable") -> "Executor":
        """Add after function middleware (Function-first API)"""
        ...

    def remove(self, middleware: Middleware) -> bool:
        """Remove middleware"""
        ...

    # ============ Properties ============

    @property
    def registry(self) -> Registry:
        """Get Registry"""
        ...

    @property
    def middlewares(self) -> list[Middleware]:
        """Get middleware list"""
        ...
```

---

## 2. Initialization

### 2.1 Basic Initialization

```python
from apcore import Registry, Executor

# Create Registry
registry = Registry(extensions_dir="./extensions")
registry.discover()

# Create Executor
executor = Executor(registry)
```

### 2.2 With Middleware

```python
from apcore import Registry, Executor
from apcore.middleware import LoggingMiddleware, MetricsMiddleware

executor = Executor(
    registry=registry,
    middlewares=[
        LoggingMiddleware(),
        MetricsMiddleware()
    ]
)
```

### 2.3 With ACL

```python
from apcore import Registry, Executor, ACL

acl = ACL.load("./acl/global_acl.yaml")
executor = Executor(registry=registry, acl=acl)
```

### 2.4 With ApprovalHandler

```python
from apcore import Registry, Executor
from apcore.approval import AutoApproveHandler, CallbackApprovalHandler, ApprovalResult

# Auto-approve (testing/development)
executor = Executor(registry=registry, approval_handler=AutoApproveHandler())

# Custom callback
async def my_approval_logic(request):
    if request.module_id.startswith("dangerous."):
        return ApprovalResult(status="rejected", reason="Dangerous modules blocked")
    return ApprovalResult(status="approved")

executor = Executor(
    registry=registry,
    approval_handler=CallbackApprovalHandler(my_approval_logic)
)
```

When an `approval_handler` is set, modules declaring `requires_approval=true` in their annotations will trigger the handler at Step 4.5 of the execution pipeline. See [Approval System](../features/approval-system.md).

---

## 3. Calling Modules

### 3.1 Basic Call

```python
result = executor.call(
    module_id="executor.email.send_email",
    inputs={
        "to": "user@example.com",
        "subject": "Hello",
        "body": "World"
    }
)

print(result)
# {"success": True, "message_id": "msg_123", "error": None}
```

### 3.2 With Context

```python
from apcore import Context

# Create custom Context
context = Context(
    trace_id="custom-trace-123",
    identity=Identity(id="user_456", type="user", roles=["admin"]),
    data={"locale": "zh-CN"}
)

result = executor.call(
    module_id="executor.email.send_email",
    inputs={"to": "user@example.com", "subject": "Hi", "body": "Hello"},
    context=context
)
```

### 3.3 Async Call

```python
import asyncio

async def main():
    result = await executor.call_async(
        module_id="executor.email.send_email",
        inputs={"to": "user@example.com", "subject": "Hi", "body": "Hello"}
    )
    print(result)

asyncio.run(main())
```

### 3.4 Batch Concurrent Calls

```python
import asyncio

async def send_batch_emails(emails: list[dict]):
    tasks = [
        executor.call_async(
            module_id="executor.email.send_email",
            inputs=email
        )
        for email in emails
    ]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return results

# Usage
emails = [
    {"to": "user1@example.com", "subject": "Hi", "body": "Hello 1"},
    {"to": "user2@example.com", "subject": "Hi", "body": "Hello 2"},
    {"to": "user3@example.com", "subject": "Hi", "body": "Hello 3"},
]
results = asyncio.run(send_batch_emails(emails))
```

---

## 4. Input Validation

### 4.1 Auto Validation

```python
# Inputs are automatically validated against input_schema
try:
    result = executor.call(
        module_id="executor.email.send_email",
        inputs={
            "to": "invalid-email",  # Format error
            "subject": "Hi"
            # Missing required field body
        }
    )
except ValidationError as e:
    print(f"Validation failed: {e.errors}")
    # [
    #   {"field": "to", "message": "Invalid email format"},
    #   {"field": "body", "message": "Field required"}
    # ]
```

### 4.2 Pre-validation

```python
# Only validate, don't execute
validation = executor.validate(
    module_id="executor.email.send_email",
    inputs={"to": "user@example.com", "subject": "Hi"}
)

if validation.valid:
    print("Inputs are valid")
else:
    print(f"Validation errors: {validation.errors}")
```

---

## 5. Error Handling

### 5.1 Error Types

```python
from apcore import (
    ModuleNotFoundError,
    ValidationError,
    ACLDeniedError,
    ApprovalDeniedError,
    ApprovalTimeoutError,
    ApprovalPendingError,
    CallDepthExceededError,
    CircularCallError,
    CallFrequencyExceededError,
    ModuleError
)

try:
    result = executor.call(
        module_id="executor.email.send_email",
        inputs={"to": "user@example.com", "subject": "Hi", "body": "Hello"}
    )
except ModuleNotFoundError as e:
    # Module does not exist
    print(f"Module not found: {e.module_id}")

except ValidationError as e:
    # Input/output validation failed
    print(f"Validation failed: {e.errors}")

except ACLDeniedError as e:
    # Insufficient permissions
    print(f"Access denied: {e.message}")
    print(f"Required: {e.required_permission}")
    print(f"Caller: {e.caller_id}")

except ApprovalDeniedError as e:
    # Approval explicitly rejected (requires_approval=true module)
    print(f"Approval denied: {e.reason}")

except ApprovalTimeoutError as e:
    # Approval timed out waiting for response
    print(f"Approval timed out: {e.reason}")

except CallDepthExceededError as e:
    # Call chain depth exceeded
    print(f"Call depth exceeded: {e.current_depth}/{e.max_depth}")
    print(f"Call chain: {e.call_chain}")

except CircularCallError as e:
    # Circular call (strict cycle: A→B→A)
    print(f"Circular call detected: {e.module_id}")
    print(f"Call chain: {e.call_chain}")

except CallFrequencyExceededError as e:
    # Same module call frequency exceeded (A→B→C→B→C→B→C...)
    print(f"Call frequency exceeded: {e.module_id} called {e.count}/{e.max_repeat} times")
    print(f"Call chain: {e.call_chain}")

except ModuleError as e:
    # Module execution error
    print(f"Module error: {e.message}")
    print(f"Module ID: {e.module_id}")
    print(f"Trace ID: {e.trace_id}")
```

### 5.2 Error Context

```python
try:
    result = executor.call(...)
except ModuleError as e:
    # Get complete error context
    print(f"Error: {e.message}")
    print(f"Module: {e.module_id}")
    print(f"Trace ID: {e.trace_id}")
    print(f"Call Chain: {e.call_chain}")
    print(f"Inputs: {e.inputs}")

    # Log to logging system
    logger.error(
        f"Module execution failed",
        extra={
            "trace_id": e.trace_id,
            "module_id": e.module_id,
            "call_chain": e.call_chain
        }
    )
```

### 5.3 AI Error Guidance Fields

All `ModuleError` instances carry four optional guidance fields (see PROTOCOL_SPEC §8.1.1) that enable AI agents to programmatically understand and respond to errors without parsing human-readable messages:

| Field | Type | Purpose |
|-------|------|---------|
| `retryable` | `bool \| None` | Whether retrying the same call may succeed. Each error code has a default (see §8.6). `None` means "depends on context". |
| `ai_guidance` | `str \| None` | Machine-readable hint for AI agents, e.g. `"validate input schema before retry"`. |
| `user_fixable` | `bool \| None` | Whether the end-user (not developer) can fix the issue. |
| `suggestion` | `str \| None` | Actionable suggestion for resolving the error. |

Fields with `None` values are omitted from serialized output (sparse serialization).

```python
except SchemaValidationError as e:
    print(e.retryable)      # False (default for this error code)
    print(e.user_fixable)   # True — user can fix their input
    print(e.suggestion)     # "Table names must use only lowercase letters and underscores"
    print(e.ai_guidance)    # "validate input against schema before retry"
```

---

## 6. Execution Flow

### 6.1 Complete Flow

```
executor.call(module_id, inputs, context)
    │
    ├─ 1. Create/validate Context
    │      └─ If context is None, auto-create
    │
    ├─ 2. Call chain safety checks
    │      ├─ 2a. Depth check: len(call_chain) >= 32 → throw CallDepthExceededError
    │      ├─ 2b. Cycle detection: module_id in call_chain → throw CircularCallError
    │      └─ 2c. Frequency detection: call_chain.count(module_id) >= max_repeat → throw CallFrequencyExceededError
    │
    ├─ 3. Lookup module
    │      └─ registry.get(module_id)
    │      └─ If not exists, throw ModuleNotFoundError
    │
    ├─ 4. ACL check
    │      └─ acl.check(caller_id, module_id, context)
    │      └─ If denied, throw ACLDeniedError
    │
    ├─ 4.5. Approval gate (if approval_handler configured)
    │      └─ Only for modules with requires_approval=true
    │      └─ approval_handler.request_approval(request)
    │      └─ If rejected, throw ApprovalDeniedError
    │      └─ If timeout, throw ApprovalTimeoutError
    │      └─ If pending, throw ApprovalPendingError (Phase B)
    │
    ├─ 5. Input validation
    │      └─ Validate against input_schema
    │      └─ If failed, throw ValidationError
    │
    ├─ 6. Middleware before
    │      └─ middleware.before(module_id, inputs, context)
    │
    ├─ 7. Execute module
    │      └─ module.execute(inputs, context)
    │      └─ If failed, call middleware on_error
    │
    ├─ 8. Output validation
    │      └─ Validate against output_schema
    │
    ├─ 9. Middleware after
    │      └─ middleware.after(module_id, inputs, output, context)
    │
    └─ 10. Return result
```

### 6.2 Automatic Context Handling

When modules internally call other modules, Executor automatically handles Context:

```python
# Internal module call
result = context.executor.call(
    module_id="executor.other_module",
    inputs={...},
    context=context  # Pass current context
)

# Executor automatically:
# 1. Keeps trace_id unchanged
# 2. Updates caller_id to current module
# 3. Appends call_chain
# 4. Call chain safety checks (depth + cycle + frequency)
```

### 6.3 Execution State Machine

```
  ┌─────────┐
  │  idle   │
  └────┬────┘
       │ call()
       ▼
  ┌──────────┐  depth/cycle/freq  ┌──────────────────────────┐
  │call_chain│───────────────────▶│ error: DEPTH_EXCEEDED    │
  │  guard   │                    │      / CIRCULAR_CALL     │
  └────┬─────┘                    │      / FREQUENCY_EXCEEDED│
       │ check passed             └──────────────────────────┘
       ▼
  ┌─────────┐    module not exist ┌──────────────────┐
  │ resolve │───────────────────▶│ error: NOT_FOUND │
  └────┬────┘                    └──────────────────┘
       │ module found
       ▼
  ┌─────────┐    permission denied ┌──────────────────┐
  │  acl    │────────────────────▶│ error: ACL_DENIED│
  └────┬────┘                     └──────────────────┘
       │ permission passed
       ▼
  ┌──────────┐  rejected/timeout ┌──────────────────────────┐
  │ approval │──────────────────▶│ error: APPROVAL_DENIED   │
  │   gate   │                   │      / APPROVAL_TIMEOUT  │
  └────┬─────┘                   │      / APPROVAL_PENDING  │
       │                         └──────────────────────────┘
       │ approved (or skipped)
       ▼
  ┌──────────┐   validation failed ┌──────────────────────┐
  │ validate │────────────────────▶│ error: VALIDATION    │
  │  input   │                     └──────────────────────┘
  └────┬─────┘
       │ validation passed
       ▼
  ┌──────────┐
  │ before   │──── middleware error ──▶ on_error chain
  │middleware│
  └────┬─────┘
       │
       ▼
  ┌──────────┐   execution error  ┌──────────────────────┐
  │ execute  │────────────────────▶│ on_error middleware  │
  │  module  │                     └──────────────────────┘
  └────┬─────┘
       │ success
       ▼
  ┌──────────┐   validation failed ┌──────────────────────┐
  │ validate │────────────────────▶│ error: VALIDATION    │
  │  output  │                     └──────────────────────┘
  └────┬─────┘
       │
       ▼
  ┌──────────┐
  │  after   │
  │middleware│
  └────┬─────┘
       │
       ▼
  ┌──────────┐
  │ return   │
  │ result   │
  └──────────┘
```

### 6.4 Timeout Specification

| Config | Default | Description |
|------|--------|------|
| Module execution timeout | 30000ms | Can be overridden via `resources.timeout` |
| Global timeout | 60000ms | Total time including middleware and validation |
| ACL check timeout | 1000ms | Max time for ACL rule evaluation |

- After timeout **MUST** throw `MODULE_TIMEOUT` error
- Timeout counting **MUST** start from the first `before()` middleware
- Middleware execution time **SHOULD** count toward total timeout

### 6.5 Concurrent Execution Semantics

- Same Executor instance **MUST** support concurrent calls by multiple threads/coroutines
- Each `call()` **MUST** use independent Context copy (call_chain and caller_id)
- `context.data` is shared by reference, concurrent calls **SHOULD** use different Context instances
- Batch `call_async()` **MAY** execute concurrently, order not guaranteed

### 6.6 Edge Case Handling

Implementations **MUST** handle Executor edge cases per the following table:

| Scenario | Behavior | Level |
|------|------|------|
| `timeout = 0` | Disable timeout limit, log WARN | **MUST** |
| `timeout` is negative | Throw `GENERAL_INVALID_INPUT` | **MUST** |
| `module_id` is empty string `""` | Throw `MODULE_NOT_FOUND` | **MUST** |
| `inputs = null` | Treat as empty dict `{}`, continue validation | **MUST** |
| `context = null` | Create new Context (empty call_chain) | **MUST** |
| Concurrent calls sharing same Context instance | Behavior undefined (race condition), **SHOULD** log WARN | **SHOULD** |
| `call()` during module `unregister()` | If execution started, continue; if not started, throw `MODULE_NOT_FOUND` | **MUST** |
| Call chain depth exceeds `max_call_depth` | Throw `CALL_DEPTH_EXCEEDED` | **MUST** |

**Concurrent safety notes:**
- Executor instance **MUST** be thread-safe, supporting multi-threaded concurrent calls
- Each `call()` **SHOULD** use independent Context instance (created via `derive()`)
- See [PROTOCOL_SPEC §12.7 Concurrency Model Specification](../../PROTOCOL_SPEC.md#127-concurrency-model-specification)

---

## 7. Middleware

### 7.1 Adding Middleware

```python
from apcore.middleware import LoggingMiddleware, MetricsMiddleware

# Method 1: Add during initialization (Class-based)
executor = Executor(
    registry=registry,
    middlewares=[LoggingMiddleware(), MetricsMiddleware()]
)

# Method 2: Chain adding (Class-based)
executor = Executor(registry=registry)
executor.use(LoggingMiddleware()).use(MetricsMiddleware())

# Method 3: Function-first API (lightweight scenarios)
executor.use_before(lambda module_id, inputs, ctx: print(f"Calling {module_id}"))
executor.use_after(lambda module_id, inputs, output, ctx: print(f"Done {module_id}"))
```

### 7.2 Execution Order

Middleware executes in onion model:

```
Request → MW1.before → MW2.before → Module → MW2.after → MW1.after → Response

On error:
Request → MW1.before → MW2.before → Module(error) → MW2.on_error → MW1.on_error
```

### 7.3 Removing Middleware

```python
logging_mw = LoggingMiddleware()
executor.use(logging_mw)

# Remove
executor.remove(logging_mw)
```

---

## 8. ACL Integration

### 8.1 Configure ACL

```yaml
# acl/global_acl.yaml
rules:
  # Allow all modules to call common.* modules
  - callers: ["*"]
    targets: ["common.*"]
    effect: allow

  # Only allow orchestrator.* to call executor.*
  - callers: ["orchestrator.*"]
    targets: ["executor.*"]
    effect: allow

  # Deny direct calls to internal.*
  - callers: ["*"]
    targets: ["internal.*"]
    effect: deny

  # Default allow
  - callers: ["*"]
    targets: ["*"]
    effect: allow
```

### 8.2 Using ACL

```python
from apcore import Executor, ACL

acl = ACL.load("./acl/global_acl.yaml")
executor = Executor(registry=registry, acl=acl)

# Permissions automatically checked during calls
try:
    result = executor.call(
        module_id="internal.secret_module",
        inputs={...},
        context=context
    )
except ACLDeniedError as e:
    print(f"Access denied: {e.message}")
```

---

## 9. Observability

### 9.1 Built-in Metrics

Executor automatically collects the following metrics:

| Metric | Type | Description |
|------|------|------|
| `apcore_module_calls_total` | Counter | Total module calls |
| `apcore_module_duration_seconds` | Histogram | Module execution time |
| `apcore_module_errors_total` | Counter | Total module errors |
| `apcore_validation_errors_total` | Counter | Total validation errors |
| `apcore_acl_denied_total` | Counter | Total ACL denials |

### 9.2 Tracing

```python
# All calls carry trace_id
result = executor.call(
    module_id="executor.email.send_email",
    inputs={...}
)

# Can get trace_id from middleware or logs
# Used to trace complete call chain in distributed systems
```

### 9.3 Logging

```python
from apcore.middleware import LoggingMiddleware
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)

# Add logging middleware
executor.use(LoggingMiddleware(
    log_inputs=True,   # Whether to log inputs
    log_outputs=True,  # Whether to log outputs
    log_errors=True    # Whether to log errors
))
```

Log output example:

```
[INFO] [trace:abc-123] Before: executor.email.send_email
[INFO] [trace:abc-123] Inputs: {"to": "user@example.com", ...}
[INFO] [trace:abc-123] After: executor.email.send_email (45ms)
[INFO] [trace:abc-123] Output: {"success": true, ...}
```

> **Security tip**: Logging middleware **SHOULD** use `context.redacted_inputs` instead of raw `inputs` to avoid leaking sensitive fields marked with `x-sensitive` (like passwords, API Keys). See [Context Object — redacted_inputs](./context-object.md#52-redacted-data).

---

## 10. Complete Example

```python
from apcore import Registry, Executor, Context, ACL
from apcore.middleware import LoggingMiddleware, MetricsMiddleware

# 1. Create Registry and discover modules
registry = Registry(extensions_dir="./extensions")
registry.discover()

# 2. Load ACL
acl = ACL.load("./acl/global_acl.yaml")

# 3. Create Executor
executor = Executor(
    registry=registry,
    middlewares=[
        LoggingMiddleware(),
        MetricsMiddleware()
    ],
    acl=acl
)

# 4. Create Context (with identity info)
context = Context.create(
    executor=executor,
    identity=Identity(id="user_123", type="user", roles=["admin"]),
    data={"locale": "zh-CN"}
)

# 5. Call module
try:
    result = executor.call(
        module_id="executor.email.send_email",
        inputs={
            "to": "user@example.com",
            "subject": "Hello",
            "body": "World"
        },
        context=context
    )
    print(f"Success: {result}")

except Exception as e:
    print(f"Error: {e}")

# 6. Async batch calls
async def send_batch():
    tasks = [
        executor.call_async(
            module_id="executor.email.send_email",
            inputs={"to": f"user{i}@example.com", "subject": "Hi", "body": "Hello"},
            context=context
        )
        for i in range(10)
    ]
    return await asyncio.gather(*tasks, return_exceptions=True)

import asyncio
results = asyncio.run(send_batch())
print(f"Sent {len([r for r in results if not isinstance(r, Exception)])} emails")
```

---

## Next Steps

- [Registry API](./registry-api.md) - Module registration and discovery
- [Context Object](./context-object.md) - Execution context
- [Middleware Guide](../guides/middleware.md) - Middleware development
- [ACL Configuration Guide](../guides/acl-configuration.md) - Access control configuration
