# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.3.0] - 2026-03-01

### Added

#### Protocol Specification
- **Approval System (§7)** — new section in `PROTOCOL_SPEC.md` defining the `ApprovalHandler` protocol, `ApprovalRequest`/`ApprovalResult` data types, Executor Step 4.5 integration, error types (`APPROVAL_DENIED`, `APPROVAL_TIMEOUT`, `APPROVAL_PENDING`), built-in handlers, protocol bridge handlers, phased implementation (Phase A sync, Phase B async), and conformance levels

#### Feature Documentation
- `docs/features/approval-system.md` — full specification of the Approval System feature

### Changed
- **`PROTOCOL_SPEC.md`** — bumped to v1.3.0-draft; added "Recommended AI Intent Metadata Keys" (§4.6) outlining `x-when-to-use`, `x-when-not-to-use`, `x-common-mistakes`, and `x-workflow-hints` conventions for LLM agents; updated `requires_approval` annotation description to reference runtime enforcement; added approval error codes and error hierarchy; renumbered §7–§13 → §8–§14
- **`docs/api/executor-api.md`** — added `approval_handler` constructor parameter, `ApprovalDeniedError`/`ApprovalTimeoutError` to error types, Step 4.5 to execution flow and state machine
- **`docs/api/module-interface.md`** — updated `requires_approval` annotation description to reference Approval System and runtime enforcement
- **`docs/features/core-executor.md`** — added Step 4.5 (Approval Gate) to the execution pipeline
- **`docs/README.md`** — added Approval System to feature specifications table, directory tree, and concept index

---

## [0.2.0] - 2026-02-23

### Added

#### Protocol Specification
- **Streaming execution protocol** — new section in `PROTOCOL_SPEC.md` defining streaming execution model for long-running or real-time module outputs
- **Module ID constraints & naming conventions** — formal rules for module identifiers added to the protocol spec
- **SDK export requirements** — specified required exports for conformant SDK implementations
- **Module interface improvements** — refined module interface contracts with streaming semantics

#### Feature Documentation
- `docs/features/acl-system.md` — full specification of the Access Control List system
- `docs/features/core-executor.md` — detailed documentation of the 10-step execution pipeline
- `docs/features/decorator-bindings.md` — guide on the `@module` decorator and binding mechanics
- `docs/features/middleware-system.md` — composable middleware pipeline specification
- `docs/features/observability.md` — tracing, metrics, and structured logging documentation
- `docs/features/registry-system.md` — module registry and discovery system documentation
- `docs/features/schema-system.md` — schema-driven module input/output validation documentation

#### CI/CD & Site
- GitHub Actions workflow (`.github/workflows/deploy-docs.yml`) for automated documentation deployment
- MkDocs configuration (`mkdocs.yml`) for the documentation site

### Changed

- **README** — enhanced with unified SDK explanation, TypeScript SDK implementation examples, and updated architecture overview
- **SCOPE.md** — updated to reflect current project scope and feature set
- **`docs/concepts.md`** — rewritten with unified SDK explanation for multi-language context
- **`docs/architecture.md`** — updated to align with protocol spec changes
- **`docs/api/`** — updated `context-object.md`, `executor-api.md`, `module-interface.md`, and `registry-api.md` with version-aligned content
- **`docs/spec/conformance.md`** — revised conformance requirements to match 0.2.0 protocol spec
- **`docs/guides/adapter-development.md`** — updated adapter development guide
- **`mkdocs.yml`** — navigation structure updated to include new feature pages

### Removed

- `ROADMAP.md` — removed; roadmap references updated across documentation
- `docs/guides/creating-modules-translated.md` — removed translated guide from navigation

---

## [0.1.0] - 2026-01-01

Initial Release
### Added

#### Core Framework
- **Schema-driven modules** - Define modules with Pydantic input/output schemas and automatic validation
- **@module decorator** - Zero-boilerplate decorator to turn functions into schema-aware modules
- **Executor** - 10-step execution pipeline with comprehensive safety and security checks
- **Registry** - Module registration and discovery system with metadata support

#### Security & Safety
- **Access Control (ACL)** - Pattern-based, first-match-wins rule system with wildcard support
- **Call depth limits** - Prevent infinite recursion and stack overflow
- **Circular call detection** - Detect and prevent circular module calls
- **Frequency throttling** - Rate limit module execution
- **Timeout support** - Configure execution timeouts per module

#### Middleware System
- **Composable pipeline** - Before/after hooks for request/response processing
- **Error recovery** - Graceful error handling and recovery in middleware chain
- **LoggingMiddleware** - Structured logging for all module calls
- **TracingMiddleware** - Distributed tracing with span support for observability

#### Bindings & Configuration
- **YAML bindings** - Register modules declaratively without modifying source code
- **Configuration system** - Centralized configuration management
- **Environment support** - Environment-based configuration override

#### Observability
- **Tracing** - Span-based distributed tracing integration
- **Metrics** - Built-in metrics collection for execution monitoring
- **Context logging** - Structured logging with execution context propagation

#### Async Support
- **Sync/Async modules** - Seamless support for both synchronous and asynchronous execution
- **Async executor** - Non-blocking execution for async-first applications

#### Developer Experience
- **Type safety** - Full type annotations across the framework
- **Comprehensive tests** - 90%+ test coverage with unit and integration tests
- **Documentation** - Quick start guide, examples, and API documentation
- **Examples** - Sample modules demonstrating decorator-based and class-based patterns
