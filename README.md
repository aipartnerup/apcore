<div align="center">
  <img src="./apcore-logo.svg" alt="apcore logo" width="200"/>
</div>

# apcore — AI-Perceivable Core

> A schema-enforced module framework where every interface is inherently perceivable by AI.

A schema-driven module development framework that makes every interface naturally perceivable and understandable by AI.

**apcore is a protocol specification.** Language implementations are maintained in separate repositories — see [Implementations](#implementations).

---

## Table of Contents

- [What is apcore?](#what-is-apcore)
- [Why AI-Perceivable?](#why-ai-perceivable)
- [Core Principles](#core-principles)
- [Architecture Overview](#architecture-overview)
- [Quick Start](#quick-start)
- [Module Development](#module-development)
  - [Class-based Modules](#1-class-based-modules)
  - [@module Decorator](#2-module-decorator)
  - [module() Function Call](#3-module-function-call)
  - [External Binding (Zero Code Modification)](#4-external-binding-zero-code-modification)
- [Schema System](#schema-system)
  - [Three-Layer Metadata Design](#three-layer-metadata-design)
  - [Module Annotations](#module-annotations)
  - [LLM Extension Fields](#llm-extension-fields)
- [Context Object](#context-object)
- [ACL Access Control](#acl-access-control)
- [Middleware](#middleware)
- [Configuration](#configuration)
- [Observability](#observability)
- [Error Handling](#error-handling)
- [Cross-Language Support](#cross-language-support)
- [Relationship with Other Tools](#relationship-with-other-tools)
- [Implementations](#implementations)
- [Ecosystem](#ecosystem)
- [Documentation Index](#documentation-index)
- [Contributing](#contributing)
- [License](#license)

---

## What is apcore?

apcore is a **universal module development framework** that makes every module naturally perceivable and understandable by AI through enforced Schema definitions.

```
┌─────────────────────────────────────────────────────────────┐
│                apcore — AI-Perceivable Core                 │
│                                                             │
│  Universal framework + Enforced AI-Perceivable support     │
│  - Directory as ID (zero-config module discovery)          │
│  - Schema-driven (input/output mandatory)                  │
│  - ACL / Observability / Middleware                        │
└─────────────────────────────────────────────────────────────┘
                            ↓ Modules callable by
      ┌──────────┬──────────┬──────────┬──────────┐
      │          │          │          │          │
  Legacy Code  AI/LLM    HTTP API    CLI Tool   MCP Server
   (import)  (understands) (REST)    (terminal)  (Claude)
```

**Not just an AI framework, but a universal framework that is naturally AI-Perceivable.**

### Core Problem

Traditional module development faces a fundamental contradiction:

```
Traditional modules: Code can call, but AI cannot understand
AI-Perceivable modules: Code can call, AI can also perceive and understand
```

AI has become an important caller in software systems, but most modules lack AI-understandable metadata. apcore fundamentally solves this by **enforcing** `input_schema` / `output_schema` / `description`.

### One-Sentence Summary

> apcore solves **how to build modules** (development framework), not how to call tools (communication protocol).
> Once modules are built, they can be called by code / AI / HTTP / CLI / MCP or any other means.

---

## Why AI-Perceivable?

### Real-World Scenarios

| Scenario | Without apcore | With apcore |
|------|------------|------------|
| LLM calling your business functions | Manually write tool descriptions, map parameters | Schema auto-provided, LLM understands directly |
| New team members onboarding | Read source code, guess parameters | Clear from Schema + annotations |
| Cross-team module reuse | Outdated docs, unclear interfaces | Schema is doc, enforced validation |
| Security audit | Manually trace call relationships | ACL + call chain auto-tracked |
| Expose as MCP Server | Rewrite interface definitions | Adapter reads Schema directly |

### Design Decision

```
Reality: AI has become a key caller in software systems
Decision: Enforce input_schema / output_schema / description
Result: Modules understandable by both humans and AI, no extra cost
```

---

## Core Principles

| Principle | Description |
|------|------|
| **Schema-Driven** | All modules enforce `input_schema` / `output_schema` / `description` |
| **Directory as ID** | Directory path auto-maps to module ID, zero config |
| **AI-Perceivable** | Schema enables AI/LLM perception and understanding—a design requirement, not optional |
| **Universal Framework** | Modules callable by code/AI/HTTP/CLI or any other means |
| **Progressive Integration** | Existing code gains AI-Perceivable capability via decorators, function calls, or YAML binding |
| **Cross-Language Spec** | Language-agnostic protocol specification, any language can implement conformant SDK |

### Differences from Traditional Frameworks

| | Traditional Frameworks | apcore |
|---|---------|--------|
| **Schema** | Optional | **Enforced** |
| **AI-Perceivable** | Not guaranteed | **Guaranteed** |
| **Module Discovery** | Manual registration | Auto-discovery from directory |
| **Input Validation** | Implement yourself | Framework automatic |
| **Behavior Annotations** | None | `readonly` / `destructive` / `requires_approval` etc. |
| **Call Tracing** | Implement yourself | `trace_id` auto-propagated |

---

## Architecture Overview

apcore's architecture consists of two orthogonal dimensions: **Framework Technical Architecture** (vertical) and **Business Layering Recommendations** (horizontal).

### Framework Technical Architecture (Vertical)

The technical layers of the framework itself, defining the complete flow from module registration to execution:

```
┌─────────────────────────────────────────────────┐
│      Application Layer                          │
│   HTTP API / CLI / MCP Server / Custom Interface│
└─────────────────────┬───────────────────────────┘
                      ↓ calls
┌─────────────────────────────────────────────────┐
│      Execution Layer                            │
│  ACL check → Input validation → Middleware chain│
│  → Execute → Output validation                  │
└──────────┬──────────────────────────────────────┘
           ↓ lookup module
┌─────────────────────────────────────────────────┐
│      Registry Layer                             │
│  Scan & discover → ID mapping → Interface       │
│  validation → Module storage                    │
└──────────┬──────────────────────────────────────┘
           ↓ read
┌─────────────────────────────────────────────────┐
│      Module Layer                               │
│  User-written business modules (conforming to   │
│  Module interface specification)                │
└─────────────────────────────────────────────────┘
```

### Business Layering Recommendations (Horizontal)

Under the `extensions/` directory, modules should be organized by responsibility (enforced by ACL):

```
extensions/
├── api/                    # API Layer: Handle external requests
│   └── ACL: Can only call orchestrator.*
│
├── orchestrator/           # Orchestration Layer: Compose business flows
│   └── ACL: Can only call executor.* and common.*
│
├── executor/               # Execution Layer: Concrete business operations
│   └── ACL: Can call common.*, can connect to external systems
│
└── common/                 # Common Layer: Shared utilities and helpers
    └── ACL: Read-only operations, called by all layers
```

**Key Points**:
- Framework technical architecture (Application → Execution → Registry → Module) is apcore's **implementation mechanism**
- Business layering (api → orchestrator → executor → common) is a **best practice recommendation**, enforced through ACL configuration
- The two are orthogonal: any business layer module (api/orchestrator/executor/common) goes through the same framework layer processing

### Execution Flow

A module call goes through the following steps:

```
executor.call("executor.email.send_email", inputs, context)
   │
   ├─ 1. Context processing: Create/update call context (trace_id, caller_id, call_chain)
   ├─ 2. Lookup module: Find target module from Registry
   ├─ 3. ACL check: Verify caller has permission to call target module
   ├─ 4. Input validation: Validate input parameters against input_schema
   ├─ 5. Middleware before: Execute middleware before() hooks in sequence
   ├─ 6. Module execution: Call module.execute(inputs, context)
   ├─ 7. Output validation: Validate output result against output_schema
   ├─ 8. Middleware after: Execute middleware after() hooks in reverse order
   │
   └─ Return result
```

### Directory as ID

Module IDs are automatically generated from relative paths under the **module root directory** (default root is `extensions/`, multiple roots can be configured):

```
File path:                                   Canonical ID:
extensions/api/handler/user.py           →  api.handler.user
extensions/executor/email/send_email.py  →  executor.email.send_email
extensions/common/util/validator.py      →  common.util.validator

Rules:
1. Remove module root prefix (default `extensions/`)
2. Remove file extension
3. Replace `/` with `.`
4. Must match: ^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$
5. Maximum length: 128 characters

**Multiple Roots and Namespaces**

- If multiple module root directories are configured, each root directory automatically uses the directory name as a **namespace**, ensuring Module ID uniqueness within the same Registry.
- For example: `extensions_roots: ["./extensions", "./plugins"]` → `extensions.executor.email.send_email`, `plugins.my_tool`.
- Automatic namespacing can be overridden with explicit configuration (e.g., `{root: "./extensions", namespace: "core"}` → `core.executor.email.send_email`).
- Single root mode has no namespace by default (backward compatible); in multi-root mode, at most one root can set `namespace: ""` to omit the prefix.

**Cross-Project Conflicts**

- Same IDs from different projects **will not conflict**, unless they are merged into the same Registry / call domain (e.g., unified gateway or shared executor).
```

---

## Quick Start

> Detailed documentation: [Registry API](./docs/api/registry-api.md) | [Executor API](./docs/api/executor-api.md) | [Context Object](./docs/api/context-object.md)

```bash
pip install apcore
```

```python
from apcore import Module, Registry, Executor, Context
from pydantic import BaseModel, Field

# 1. Define module (Schema-driven)
class GreetInput(BaseModel):
    name: str = Field(..., description="User name")

class GreetOutput(BaseModel):
    message: str

class GreetModule(Module):
    """Generate greeting message"""
    input_schema = GreetInput
    output_schema = GreetOutput

    def execute(self, inputs: dict, context: Context) -> dict:
        return {"message": f"Hello, {inputs['name']}!"}

# 2. Register & call
registry = Registry()
registry.register("executor.greet", GreetModule())

executor = Executor(registry=registry)
result = executor.call("executor.greet", {"name": "World"})
print(result)  # {"message": "Hello, World!"}
```

### Project Directory Structure

```
my-project/
├── apcore.yaml               # Framework configuration
├── extensions/               # Module directory (directory path = module ID)
│   ├── api/                  # API layer
│   ├── orchestrator/         # Orchestration layer
│   └── executor/             # Execution layer
├── schemas/                  # Schema definitions (YAML, shared across languages)
└── acl/                      # Permission configuration
```

---

## Module Development

> Detailed definitions: [Module Interface](./docs/api/module-interface.md) | [Creating Modules Guide](./docs/guides/creating-modules.md)

apcore provides four ways to define modules, suitable for different scenarios:

### 1. Class-based Modules

The most complete approach, supporting all features:

```python
# extensions/executor/email/send_email.py
# Module ID auto-generated: executor.email.send_email

from apcore import Module, ModuleAnnotations, Context
from pydantic import BaseModel, Field

class SendEmailInput(BaseModel):
    """LLM understands what parameters are needed through this Schema"""
    to: str = Field(..., description="Recipient email address")
    subject: str = Field(..., description="Email subject")
    body: str = Field(..., description="Email body")

class SendEmailOutput(BaseModel):
    """LLM understands what is returned through this Schema"""
    success: bool
    message_id: str = None

class SendEmailModule(Module):
    """Send email module

    Detailed documentation:
    - Supports text and HTML format emails
    - Uses SMTP protocol to connect to external mail server
    - Configuration items: smtp_host, smtp_port, smtp_user, smtp_pass

    Usage example:
      Input: {"to": "user@example.com", "subject": "Hello", "body": "World"}
      Output: {"success": true, "message_id": "msg_123"}

    Notes:
    - SMTP server information must be configured in the configuration file
    - Gmail limits 500 emails/day, other providers may have different limits
    - EmailSendError exception will be raised on send failure
    """

    # Core layer (must be defined)
    input_schema = SendEmailInput
    output_schema = SendEmailOutput
    description = "Send email to specified recipient. Uses SMTP protocol, non-idempotent operation, requires mail server configuration."

    # Optional: Detailed documentation (for complex modules)
    documentation = """
# Features
Send emails via SMTP protocol, supporting plain text and HTML formats.

## Configuration Requirements
- SMTP server information must be configured in apcore.yaml
- Valid SMTP authentication credentials required

## Use Cases
- Send notification emails, verification codes, reports

## Limitations
- Gmail: 500 emails/day
- Attachment size: ≤25MB
"""

    # Annotation layer (optional, type-safe)
    annotations = ModuleAnnotations(
        readonly=False,        # Has side effects
        destructive=False,     # Won't delete/overwrite data
        idempotent=False,      # Repeated calls will send repeatedly
        requires_approval=True, # Requires user confirmation
        open_world=True,       # Connects to external system (SMTP)
    )
    tags = ["email", "notification"]

    def execute(self, inputs: dict, context: Context) -> dict:
        validated = SendEmailInput(**inputs)
        # ... send email logic ...
        return SendEmailOutput(success=True, message_id="msg_123").model_dump()
```

**This module automatically has:**
- LLM-understandable Schema and behavior annotations
- Auto-generated ID (`executor.email.send_email`)
- Input/output validation
- Call chain tracing, observability

### 2. @module Decorator

Suitable for scenarios where **source code can be modified**, one-line integration:

```python
# Before: Plain function
def send_email(to: str, subject: str, body: str) -> dict:
    """Send email"""
    return {"success": True, "message_id": "msg_123"}

# After: Add one line decorator, automatically becomes an apcore module
from apcore import module

@module(id="email.send", tags=["email"])
def send_email(to: str, subject: str, body: str) -> dict:
    """Send email"""
    return {"success": True, "message_id": "msg_123"}
# Schema automatically inferred from type annotations
```

### 3. module() Function Call

Suitable for scenarios where **you don't want to modify source code**, completely non-invasive to existing code:

```python
from apcore import module

# Existing business code, no modification needed
class EmailService:
    def send(self, to: str, subject: str, body: str) -> dict:
        """Send email"""
        return {"success": True}

service = EmailService()
module(service.send, id="email.send")  # Register as apcore module
```

### 4. External Binding (Zero Code Modification)

Suitable for scenarios where **source code cannot be modified** (third-party libraries, legacy systems, etc.), pure YAML configuration:

```yaml
# bindings/email.binding.yaml
bindings:
  - module_id: "email.send"
    target: "myapp.services.email:send_email"   # Callable object path
    description: "Send email"
    auto_schema: true     # Auto-generate Schema from type annotations
    annotations:
      open_world: true
      requires_approval: true
    tags: ["email"]
```

### Comparison of Four Approaches

| Approach | Code Invasiveness | Use Case | Schema Definition |
|------|-----------|---------|------------|
| Class-based | High (write new class) | New module development | Manual definition (most complete) |
| `@module` Decorator | Low (add one line) | Modifiable code | Inferred from type annotations |
| `module()` Function Call | Very low (don't modify original function) | Existing classes/methods | Inferred from type annotations |
| External Binding | Zero | Cannot modify source code scenarios | Auto-inferred or manually specified |

---

## Schema System

> Detailed definitions: [Schema Definition Guide](./docs/guides/schema-definition.md) | [ModuleAnnotations API](./docs/api/module-interface.md#34-annotations)

### Three-Layer Metadata Design

Each module's metadata is divided into three layers, progressing from required to optional:

```
┌──────────────────────────────────────────────────┐
│ Core Layer (REQUIRED)                            │
│ input_schema / output_schema / description       │
│ → AI understands "what this module does"         │
│                                                  │
│ + documentation (OPTIONAL, detailed docs)        │
│ → AI understands "detailed use cases and         │
│   constraints"                                   │
├──────────────────────────────────────────────────┤
│ Annotation Layer (OPTIONAL, type-safe)           │
│ annotations / examples / tags / version          │
│ → AI understands "how to use correctly"          │
├──────────────────────────────────────────────────┤
│ Extension Layer (OPTIONAL, free dictionary)      │
│ metadata: dict[str, Any]                         │
│ → Custom requirements (framework doesn't         │
│   validate)                                      │
└──────────────────────────────────────────────────┘
```

### Description and Documentation Fields

**Borrowing from Claude Skill's Progressive Disclosure design, apcore uses two fields to organize module documentation:**

| Field | Required | Length Limit | Markdown | Purpose |
|------|--------|----------|----------|------|
| `description` | **Required** | ≤200 characters | No | Brief module function description for AI quick matching and understanding |
| `documentation` | Optional | ≤5000 characters | Yes | Detailed documentation including use cases, constraints, configuration requirements |

- **Module discovery phase**: AI reads all modules' `description`, quickly determines candidate modules
- **Call decision phase**: AI loads `documentation` on-demand, learns detailed usage and constraints

> Complete format rules and correspondence with Claude Skill / OpenAPI: see [Protocol Specification §4.8](./PROTOCOL_SPEC.md#48-description-and-documentation). Code examples: see [Class-based Modules](#1-class-based-modules) above.

### Schema Definition

Schema is based on **JSON Schema Draft 2020-12**, supports YAML format definition (shared across languages). Schema files are placed in the `schemas/` directory, with paths corresponding to module IDs.

> Complete Schema format and YAML examples: see [Schema Definition Guide](./docs/guides/schema-definition.md) | [Protocol Specification §4](./PROTOCOL_SPEC.md#4-schema-specification).

### Module Annotations

Annotations describe module **behavior characteristics**, helping AI make safer call decisions:

| Annotation | Type | Description | AI Behavior Impact |
|------|------|------|------------|
| `readonly` | bool | No side effects, read-only operation | AI can safely call autonomously |
| `destructive` | bool | May delete or overwrite data | AI should request user confirmation before calling |
| `idempotent` | bool | Repeated calls have same result | AI can safely retry |
| `requires_approval` | bool | Requires explicit user consent | AI must wait for user approval |
| `open_world` | bool | Connects to external systems | AI should inform user of external interaction |

```python
# Read-only query - AI can call autonomously
annotations = ModuleAnnotations(readonly=True)

# Delete operation - AI needs to request confirmation
annotations = ModuleAnnotations(destructive=True, requires_approval=True)

# External API call - AI needs to inform user
annotations = ModuleAnnotations(open_world=True, idempotent=True)
```

### LLM Extension Fields

Fields with `x-` prefix in Schema are LLM-specific extensions, don't affect standard JSON Schema validation:

| Field | Description | Example |
|------|------|------|
| `x-llm-description` | Extended description for LLM (more detailed than description) | `"User's login password, at least 8 characters"` |
| `x-examples` | Example values to help LLM understand format | `["user@example.com"]` |
| `x-sensitive` | Mark sensitive fields (password, API Key, etc.) | `true` |
| `x-constraints` | Business constraints described in natural language | `"Must be a registered user"` |
| `x-deprecated` | Deprecation information | `{"since": "2.0", "use": "new_field"}` |

> Complete usage and examples: see [Schema Definition Guide](./docs/guides/schema-definition.md) | [Protocol Specification §4.3](./PROTOCOL_SPEC.md#43-llm-extension-fields).

---

## Context Object

Context is the execution context that runs through the entire call chain, carrying tracing, permissions, and shared data:

```python
class Context:
  trace_id: str           # Call trace ID (UUID v4; W3C trace-id compatible in distributed scenarios)
    caller_id: str | None   # Caller module ID (None for top-level calls)
    call_chain: list[str]   # Call chain (accumulated in call order)
    executor: Executor      # Executor reference (entry point for inter-module calls)
    identity: Identity      # Caller identity
    data: dict              # Shared data (reference-shared within call chain)
```

### Call Chain Propagation

```python
# Top-level call
context = Context(trace_id="abc-123", identity=Identity(id="user_1", roles=["admin"]))

# Module A is called
#   trace_id: "abc-123"        ← Stays the same
#   caller_id: None            ← No caller at top level
#   call_chain: ["module_a"]

# Module A internally calls Module B
result = context.executor.call("module_b", inputs, context)
#   trace_id: "abc-123"        ← Stays the same
#   caller_id: "module_a"      ← Caller is module_a
#   call_chain: ["module_a", "module_b"]

# Module B internally calls Module C
#   trace_id: "abc-123"        ← Same trace_id for entire chain
#   caller_id: "module_b"
#   call_chain: ["module_a", "module_b", "module_c"]
```

**Key Feature:** `context.data` is **reference-shared** throughout the entire call chain, allowing modules to pass intermediate results and implement pipeline-style data flow.

---

## ACL Access Control

> Detailed definitions: [ACL Configuration Guide](./docs/guides/acl-configuration.md) | [Protocol Specification §6](./PROTOCOL_SPEC.md#6-acl-specification)

ACL (Access Control List) controls which modules can call which modules, default deny:

```yaml
# acl/global_acl.yaml
rules:
  # API layer can only call orchestration layer
  - callers: ["api.*"]
    targets: ["orchestrator.*"]
    effect: allow

  # Orchestration layer can call execution layer
  - callers: ["orchestrator.*"]
    targets: ["executor.*"]
    effect: allow

  # Forbid cross-layer calls (API directly calling execution layer)
  - callers: ["api.*"]
    targets: ["executor.*"]
    effect: deny

  # System internal modules unrestricted
  - callers: ["@system"]
    targets: ["*"]
    effect: allow

default_effect: deny  # Default deny when no rules match

audit:
  enabled: true
  log_level: info
  include_denied: true
```

### Special Caller Identifiers

| Identifier | Description |
|------|------|
| `@external` | Top-level external call (HTTP request, CLI command, etc.) |
| `@system` | Framework internal call |
| `*` | Wildcard, matches all |

### Conditional Rules

```yaml
rules:
  - callers: ["api.*"]
    targets: ["executor.payment.*"]
    effect: allow
    conditions:
      identity_types: ["user"]      # Only user identity
      roles: ["admin", "finance"]   # Only admin or finance roles
      max_call_depth: 5             # Maximum call depth
```

---

## Middleware

> Detailed definitions: [Middleware Guide](./docs/guides/middleware.md)

Middleware uses the **Onion Model**, allowing custom logic to be inserted before and after module execution:

```
Request → [MW1.before → [MW2.before → [MW3.before →
          [Module.execute()]
← MW3.after] ← MW2.after] ← MW1.after] ← Response
```

```python
class LoggingMiddleware(Middleware):
    def before(self, module_id: str, inputs: dict, context: Context) -> dict:
        log.info(f"Calling {module_id} with trace_id={context.trace_id}")
        return inputs  # Can modify inputs

    def after(self, module_id: str, inputs: dict, output: dict, context: Context) -> dict:
        log.info(f"Result from {module_id}: success")
        return output  # Can modify output

    def on_error(self, module_id: str, inputs: dict, error: Exception, context: Context):
        log.error(f"Error in {module_id}: {error}")
        # Optional: Convert exception, trigger alerts, etc.
```

Typical middleware scenarios: logging, performance monitoring, caching, rate limiting, retry, auditing.

---

## Configuration

The framework is centrally configured through `apcore.yaml`:

```yaml
# apcore.yaml
version: "1.0.0"

project:
  name: "my-ai-project"
  version: "0.1.0"

# Module discovery (single root mode, backward compatible)
extensions:
  root: "./extensions"       # Module root directory
  auto_discover: true        # Auto-scan and discover
  lazy_load: true            # Lazy load (load module only on first call)
  max_depth: 8               # Maximum directory depth

# Or: Multi-root mode (namespace isolation)
# extensions:
#   roots:
#     - root: "./extensions"
#       namespace: "core"    # Explicit namespace → core.executor.email.send_email
#     - "./plugins"          # Auto namespace → plugins.my_tool

# Schema loading
schema:
  root: "./schemas"
  strategy: "yaml_first"     # yaml_first | native_first | yaml_only
  validation:
    strict: true             # Strict validation mode
    coerce_types: true       # Automatic type coercion

# Access control
acl:
  root: "./acl"
  default_effect: "deny"     # Default deny
  audit:
    enabled: true

# Logging
logging:
  level: "info"              # trace | debug | info | warn | error | fatal
  format: "json"             # json | text

# Observability
observability:
  tracing:
    enabled: true
    sampling_rate: 1.0       # 1.0 = full collection, 0.1 = 10% sampling
    exporter: "stdout"       # stdout | otlp | jaeger
  metrics:
    enabled: true
    exporter: "prometheus"

# Middleware
middleware:
  - class: "myapp.middleware.LoggingMiddleware"
    priority: 100
  - class: "myapp.middleware.CachingMiddleware"
    priority: 50
    config:
      ttl: 300
```

---

## Observability

apcore has built-in three pillars of observability, compatible with OpenTelemetry:

### Tracing

- `trace_id` is automatically generated and propagated through the call chain
- Span naming convention: `apcore.{component}.{operation}`
- Supports export to stdout / OTLP / Jaeger

### Logging

- Structured logging, automatically includes `trace_id`
- Fields marked with `x-sensitive` are automatically redacted (e.g., passwords show as `***REDACTED***`)
- Executor automatically provides `context.redacted_inputs`, middleware and logs should use redacted data

### Metrics

| Metric Name | Type | Description |
|--------|------|------|
| `apcore_module_calls_total` | Counter | Total module calls |
| `apcore_module_duration_seconds` | Histogram | Module execution duration distribution |
| `apcore_module_errors_total` | Counter | Total module errors |

---

## Error Handling

apcore defines a unified error format and standard error codes:

### Error Format

```json
{
  "code": "SCHEMA_VALIDATION_ERROR",
  "message": "Input validation failed",
  "details": {"field": "email", "reason": "invalid format"},
  "cause": null,
  "trace_id": "abc-123",
  "timestamp": "2026-01-01T00:00:00Z"
}
```

### Standard Error Codes

| Category | Error Code | Description | Retryable |
|------|--------|------|--------|
| Module | `MODULE_NOT_FOUND` | Module does not exist | No |
| Module | `MODULE_EXECUTE_ERROR` | Execution exception | Depends |
| Module | `MODULE_TIMEOUT` | Execution timeout | Yes |
| Schema | `SCHEMA_VALIDATION_ERROR` | Input/output validation failed | No |
| Schema | `SCHEMA_NOT_FOUND` | Schema file does not exist | No |
| ACL | `ACL_DENIED` | Permission denied | No |
| Binding | `BINDING_INVALID_TARGET` | Invalid binding target path | No |
| Binding | `BINDING_CALLABLE_NOT_FOUND` | Bound callable object not found | No |
| General | `GENERAL_INTERNAL_ERROR` | Internal error | Yes |

---

## Cross-Language Support

apcore is a **language-agnostic framework specification**. Canonical IDs are automatically adapted to local naming conventions in different languages:

```
Canonical ID (universal): executor.email.send_email

Local representation:
Python:     executor/email/send_email.py      class SendEmailModule
Rust:       executor/email/send_email.rs       struct SendEmailModule
Go:         executor/email/send_email.go       type SendEmailModule
Java:       executor/email/SendEmail.java      class SendEmailModule
TypeScript: executor/email/sendEmail.ts        class SendEmailModule
```

### ID Mapping Rules

- Automatic language detection (based on file extension)
- Case conversion (PascalCase ↔ snake_case ↔ camelCase)
- Path separator normalization (`/` vs `::` vs `.`)
- Supports manual override (`id_map` configuration)

### Conformance Levels

Any language SDK implementation can choose different conformance levels:

| Level | Scope | Includes |
|------|------|------|
| **Level 0 (Core)** | Minimally viable | ID mapping, Schema loading, Registry, Executor |
| **Level 1 (Standard)** | Production ready | + ACL, middleware, error handling, observability |
| **Level 2 (Full)** | Complete implementation | + Hot reload, distributed execution, advanced monitoring |

**Reference Implementation**: [apcore-python](https://github.com/aipartnerup/apcore-python)

---

## Relationship with Other Tools

### apcore vs MCP

| | apcore | MCP |
|---|--------|-----|
| **Positioning** | Development framework | Communication protocol |
| **Solves** | How to **build** modules | How to **call** tools |
| **Focus** | Code organization, Schema, ACL, observability | Transport format, RPC |
| **Relationship** | apcore modules can be exposed as MCP Server | MCP is one exposure method |

### apcore vs LangChain / LlamaIndex

| | apcore | LangChain etc. |
|---|--------|-------------|
| **Positioning** | Module development framework | LLM application development framework |
| **Focus** | Module standardization, Schema, permissions | Chaining, Prompt, RAG |
| **Relationship** | Complementary — apcore modules can serve as LangChain Tools | |

### apcore vs CrewAI / AutoGen

| | apcore | CrewAI etc. |
|---|--------|----------|
| **Positioning** | Module development framework | Agent orchestration framework |
| **Focus** | Standardizing individual modules | Multi-agent collaboration strategies |
| **Relationship** | Complementary — Agents can call apcore modules | |

**In short:** apcore focuses on **building standardized, AI-understandable modules**, complementary rather than competitive with upper-layer AI protocols/frameworks.

---

## Implementations

Language SDK implementations of the apcore protocol specification:

| Language | Repository | Features | Install |
|------|------|------|------|
| **Python** | [apcore-python](https://github.com/aipartnerup/apcore-python) | Schema validation, Registry, Executor, @module decorator, YAML bindings, ACL, Middleware, Observability, Async support | `pip install apcore` |
| **Typescript** | [apcore-typescript](https://github.com/aipartnerup/apcore-typescript) | Schema validation, Registry, Executor, @module decorator, YAML bindings, ACL, Middleware, Observability, Async support | `npm install apcore-js` |

> Interested in implementing apcore for another language? See the [Protocol Specification](./PROTOCOL_SPEC.md) and [Conformance Definition](./docs/spec/conformance.md).

---

## Ecosystem

The apcore ecosystem uses a **core + independent adapters** architecture. The core does not include any framework-specific implementations; adapters are developed in independent repositories by official or community contributors.

### Adapter Types

| Type | Examples | Description |
|------|------|------|
| **Web Frameworks** | `apcore-fastapi`, `apcore-flask`, `apcore-express` | Expose modules as HTTP APIs |
| **AI Protocols** | `apcore-mcp`, `apcore-openai-tools` | Expose modules as AI tools |
| **RPC** | `apcore-grpc`, `apcore-thrift` | Expose modules as RPC services |

All adapters are built on the core's `module()` and External Binding mechanisms.
Development guide: see [Adapter Development Guide](./docs/guides/adapter-development.md).

---

## Documentation Index

### Core Documentation

| Document | Description |
|------|------|
| [Protocol Specification](./PROTOCOL_SPEC.md) | Complete framework specification (RFC 2119 Conformant) |
| [Scope Definition](./SCOPE.md) | Responsibility boundaries (what's in/out of scope) |

### Concepts & Architecture

| Document | Description |
|------|------|
| [Core Concepts](./docs/concepts.md) | Design philosophy and core concepts explained |
| [Architecture Design](./docs/architecture.md) | Internal architecture, component interaction, memory model |

### API Reference

| Document | Description |
|------|------|
| [Module Interface](./docs/api/module-interface.md) | Module interface definition |
| [Context Object](./docs/api/context-object.md) | Execution context |
| [Registry API](./docs/api/registry-api.md) | Registry API |
| [Executor API](./docs/api/executor-api.md) | Executor API |

### Feature Specifications

| Document | Description |
|------|------|
| [ACL System](./docs/features/acl-system.md) | Pattern-based Access Control List with first-match-wins evaluation |
| [Core Executor](./docs/features/core-executor.md) | Core execution engine with 10-step pipeline |
| [Decorator & YAML Bindings](./docs/features/decorator-bindings.md) | `@module` decorator and YAML-based module creation |
| [Middleware System](./docs/features/middleware-system.md) | Composable middleware pipeline with onion execution model |
| [Observability](./docs/features/observability.md) | Distributed tracing, metrics, and structured logging |
| [Registry System](./docs/features/registry-system.md) | Module discovery, registration, and querying |
| [Schema System](./docs/features/schema-system.md) | Schema loading, validation, `$ref` resolution, and export |

### Usage Guides

| Document | Description |
|------|------|
| [Creating Modules](./docs/guides/creating-modules.md) | Module creation tutorial (including four approaches) |
| [Schema Definition](./docs/guides/schema-definition.md) | Complete Schema usage |
| [ACL Configuration](./docs/guides/acl-configuration.md) | Access control configuration |
| [Middleware](./docs/guides/middleware.md) | Middleware development |
| [Adapter Development](./docs/guides/adapter-development.md) | Framework adapter development |
| [Testing Modules](./docs/guides/testing-modules.md) | Module testing guide |
| [Multi-Language Development](./docs/guides/multi-language.md) | Cross-language development guide |

### Specification Documents

| Document | Description |
|------|------|
| [Type Mapping](./docs/spec/type-mapping.md) | Cross-language type mapping |
| [Conformance Definition](./docs/spec/conformance.md) | Implementation conformance levels |
| [Algorithm Reference](./docs/spec/algorithms.md) | Core algorithm summary (including namespace, redaction, etc.) |

---

## Contributing

Contributions are welcome in the following forms:

- **Specification Feedback**: Suggest improvements to the protocol specification in Issues
- **SDK Implementation**: Implement SDKs for other languages based on [Protocol Specification](./PROTOCOL_SPEC.md)
- **Adapter Development**: Develop adapters for web frameworks or AI protocols
- **Documentation Improvements**: Fix, translate, or supplement documentation

---

## License

Apache 2.0
