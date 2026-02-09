# apcore Documentation

> Complete technical documentation for the apcore (AI-Perceivable Core) framework.

This directory contains all technical documentation for apcore, covering core concepts, architecture design, API references, usage guides, and framework specifications.
For a project overview and quick start, see the [main README](../README.md).

## Directory Overview

```
docs/
├── README.md                          ← This file (navigation index)
├── concepts.md                        ← Design philosophy & core concepts
├── architecture.md                    ← Framework technical architecture
├── api/                               ← API reference (authoritative definitions)
│   ├── module-interface.md            ← Module base class interface
│   ├── context-object.md              ← Context execution context
│   ├── registry-api.md                ← Registry center
│   └── executor-api.md                ← Executor
├── guides/                            ← Usage guides (7 articles)
│   ├── creating-modules.md            ← Getting started with module creation
│   ├── schema-definition.md           ← Schema definition in detail
│   ├── middleware.md                   ← Middleware development
│   ├── acl-configuration.md           ← ACL permission configuration
│   ├── testing-modules.md             ← Module testing strategies
│   ├── adapter-development.md         ← Adapter development
│   └── multi-language.md              ← Cross-language development
└── spec/                              ← Framework specifications (for SDK implementors)
    ├── algorithms.md                  ← Core algorithm reference (23 algorithms)
    ├── type-mapping.md                ← Cross-language type mapping
    └── conformance.md                 ← Conformance level definitions
```

Also see the root directory: [PROTOCOL_SPEC.md](../PROTOCOL_SPEC.md) — Complete protocol specification in RFC style (4,450+ lines)

## Documentation Structure

### Concepts & Architecture

| Document | Description |
|----------|-------------|
| [concepts.md](./concepts.md) | apcore's design philosophy, core concepts, and modular philosophy |
| [architecture.md](./architecture.md) | Framework technical architecture, core component interactions, and execution flow |

### [API Reference](./api/) - Authoritative Definitions

Core interface definitions for the module system, including complete API documentation for the Module base class, Context object, Registry center, and Executor.

| API Document | Description |
|--------------|-------------|
| [Module Interface](./api/module-interface.md) | Complete definition of the Module base class |
| [Context Object](./api/context-object.md) | Complete definition of the execution context |
| [Registry API](./api/registry-api.md) | Module registry center API |
| [Executor API](./api/executor-api.md) | Module executor API |

### [Usage Guides](./guides/)

Practical tutorials covering everything from creating your first module to middleware development, ACL configuration, and cross-language development. 7 guides in total, covering the full path from beginner to advanced.

| Guide | Description |
|-------|-------------|
| [Creating Modules](./guides/creating-modules.md) | Create modules from scratch, introducing multiple module definition approaches |
| [Schema Definition](./guides/schema-definition.md) | Detailed Schema definition, mastering input/output structure declaration |
| [Middleware](./guides/middleware.md) | Middleware development, extending pre/post module execution logic |
| [ACL Configuration](./guides/acl-configuration.md) | ACL permission configuration, setting up access control rules between modules |
| [Testing Modules](./guides/testing-modules.md) | Module testing strategies, covering unit tests, Schema tests, and integration tests |
| [Adapter Development](./guides/adapter-development.md) | Developing apcore adapters for third-party web frameworks |
| [Multi-Language Development](./guides/multi-language.md) | Developing modules in multiple languages using YAML Schema |

### [Framework Specifications](./spec/) - Authoritative Specifications

Formal technical specifications for SDK implementors, defining core algorithms, cross-language type mapping, and conformance levels.

- [Protocol Specification](../PROTOCOL_SPEC.md) - Complete protocol specification in RFC style
- [Type Mapping](./spec/type-mapping.md) - Cross-language type mapping
- [Conformance Definition](./spec/conformance.md) - Implementation conformance levels
- [Algorithm Reference](./spec/algorithms.md) - Core algorithm compendium

## Recommended Reading Order

If you are new to apcore, it is recommended to read in the following order:

1. [Core Concepts](./concepts.md) — Understand the design philosophy
2. [Creating Modules](./guides/creating-modules.md) — Hands-on: create your first module
3. [Schema Definition](./guides/schema-definition.md) — Master input/output Schema
4. [Module Interface](./api/module-interface.md) — Learn the complete interface definition
5. [Context Object](./api/context-object.md) — Understand the execution context
6. [Middleware](./guides/middleware.md) — Extend the execution flow
7. [ACL Configuration](./guides/acl-configuration.md) — Configure access control

After completing the above, you can read the advanced guides on testing, adapter development, multi-language development, etc. as needed.

## Concept Index

Quickly find authoritative definitions for concepts:

| Concept | Authoritative Definition | Quick Reference |
|---------|--------------------------|-----------------|
| Module | [module-interface.md](./api/module-interface.md) | [README](../README.md#module-development) |
| ModuleAnnotations | [module-interface.md#annotations](./api/module-interface.md#34-annotations) | [README](../README.md#schema-system) |
| Context | [context-object.md](./api/context-object.md) | [README](../README.md#context-object) |
| Canonical ID | [PROTOCOL_SPEC.md §2](../PROTOCOL_SPEC.md#2-naming-specification) | [README](../README.md#directory-as-id) |
| Registry | [registry-api.md](./api/registry-api.md) | [README](../README.md#quick-start) |
| Executor | [executor-api.md](./api/executor-api.md) | [README](../README.md#quick-start) |
| ACL | [PROTOCOL_SPEC.md §6](../PROTOCOL_SPEC.md#6-acl-specification) | [README](../README.md#acl-access-control) |
| Middleware | [middleware.md](./guides/middleware.md) | [README](../README.md#middleware) |
