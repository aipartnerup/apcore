# apcore — AI-Perceivable Core Development Roadmap

## Project Phases

```
Framework Specification → Python Reference Implementation → Multi-language SDK (based on community demand)
```

---

## Phase 0: Framework Specification (Current)

### Goal
Define a complete framework specification to ensure that any subsequent language implementation follows a unified standard.

### Task List

- [x] Core design discussion
- [x] Naming Spec
- [x] Directory Spec
- [x] Schema Spec
- [x] Module Spec
- [x] ACL Spec
- [x] Protocol compatibility mapping reference (MCP/A2A - Appendix)
- [x] RFC 2119 normative keywords introduction
- [x] Terminology definition table
- [x] Pseudocode algorithms (23 core algorithms)
- [x] Cross-language type mapping table
- [x] Conformance level definitions (Level 0/1/2)
- [x] JSON Schema validation files
- [x] Functional module definition specification (PROTOCOL_SPEC §5.11), including cross-language syntax comparison
- [x] External Schema binding specification (PROTOCOL_SPEC §5.12)
- [x] Binding file validation JSON Schema
- [ ] Protocol specification review

### Deliverables
- `PROTOCOL_SPEC.md` - Complete protocol specification document (RFC 2119 Conformant)
- `schemas/*.schema.json` - JSON Schema for configuration validation (5 files, including binding.schema.json)
- `docs/spec/type-mapping.md` - Cross-language type mapping specification
- `docs/spec/conformance.md` - Conformance definitions
- `docs/spec/algorithms.md` - Core algorithms summary
- `docs/guides/testing-modules.md` - Module testing guide
- `docs/guides/multi-language.md` - Multi-language development guide

---

## Phase 1: Python Reference Implementation (Core MVP)

### Goal
Implement core functionality in Python to validate protocol feasibility.

### Core Components

| Component | Description | Status |
|------|------|------|
| `IDConverter` | Canonical ID ↔ Local ID conversion | TODO |
| `SchemaLoader` | YAML Schema loading → Pydantic | TODO |
| `Registry` | Automatic module discovery and registration | TODO |
| `Executor` | Module invocation, Schema validation | TODO |
| `ModuleDecorator` | `@module` / `module()` + automatic Schema generation | TODO |
| `BindingLoader` | YAML binding file loading + target resolution | TODO |

### Task List

- [ ] Project initialization (pyproject.toml, uv)
- [ ] IDConverter implementation
- [ ] SchemaLoader implementation (YAML → Pydantic)
- [ ] Registry implementation (directory scanning)
- [ ] Executor implementation
- [ ] ModuleDecorator implementation (@module decorator + module() function call)
- [ ] Type annotation introspection and Schema generation
- [ ] BindingLoader implementation
- [ ] Binding target resolution
- [ ] Unit tests
- [ ] Integration tests
- [ ] Example modules

### Dependencies
```
python >= 3.11
pydantic >= 2.0
pyyaml >= 6.0
pluggy >= 1.0
```

---

## Phase 2: Core Complete

### Goal
Enhance core functionality: access control, middleware, observability.

### Components

| Component | Description | Status |
|------|------|------|
| `ACLChecker` | Permission checking | TODO |
| `MiddlewareManager` | Middleware management | TODO |
| `ErrorHandler` | Error handling | TODO |
| `ObservabilityProvider` | Tracing, logging, metrics | TODO |

### Task List

- [ ] ACLChecker
- [ ] MiddlewareManager
- [ ] ErrorHandler
- [ ] TracingProvider (OpenTelemetry)
- [ ] Integration tests

---

## Phase 3: CLI & Developer Experience

### Goal
Provide comprehensive command-line tools to improve the developer experience.

### CLI Commands

```bash
apcore init <project>      # Initialize project
apcore create module        # Create module template
apcore schema export       # Export YAML Schema
apcore schema validate     # Validate Schema consistency
apcore run                 # Start service
apcore test                # Run tests
```

### Task List

- [ ] CLI framework (typer/click)
- [ ] init command
- [ ] create command
- [ ] schema command
- [ ] run command
- [ ] test command

---

## Phase 4: Advanced Features

### Goal
Advanced features to improve production readiness.

### Feature List

- [ ] Module hot-reloading
- [ ] Distributed execution
- [ ] Monitoring and tracing (OpenTelemetry)
- [ ] Extension marketplace / Registry
- [ ] Web UI

---

## Future Vision: AI-Assisted Migration Tool

### Goal
Leverage AI capabilities to help developers migrate existing applications into AI-Perceivable modules.

### Feature List

- [ ] **Project Scanner**: Analyze source code to identify functions and methods suitable for wrapping as modules
- [ ] **Schema Suggester**: Generate Schema proposals from function signatures and docstrings
- [ ] **Interactive Migration**: AI-guided step-by-step migration with manual confirmation at each step
- [ ] **Batch Conversion**: Automatically generate binding files or decorator annotations
- [ ] **Migration Report**: Generate conversion reports including coverage, suggestions, and risk assessments

> This feature is positioned as an extension tool (non-core) and can be implemented as a standalone CLI tool or IDE plugin.

---

## Ecosystem Development

### Principles
- **Pure Core**: The apcore core does not contain any framework-specific code
- **Independent Repositories**: Each adapter has its own independent repository with independent version management
- **Community-Driven**: Priority is determined by community demand

### Official Adapter Roadmap

#### Web Framework Adapters
| Adapter | Framework | Language | Priority |
|--------|------|------|--------|
| `apcore-fastapi` | FastAPI | Python | P0 |
| `apcore-flask` | Flask | Python | P1 |
| `apcore-express` | Express.js | TypeScript | P1 |
| `apcore-django` | Django | Python | P2 |
| `apcore-gin` | Gin | Go | P2 |
| `apcore-spring` | Spring Boot | Java | P2 |

#### AI Protocol Adapters
| Adapter | Protocol | Priority |
|--------|------|--------|
| `apcore-mcp` | Model Context Protocol | P0 |
| `apcore-openai-tools` | OpenAI Function Calling | P1 |

### Ecosystem Milestones
| Phase | Goal |
|------|------|
| E0 | apcore-fastapi + apcore-mcp reference implementation |
| E1 | 3+ community-contributed adapters |
| E2 | 10+ adapters, covering 5+ language ecosystems |

---

## Multi-language SDK (On Demand)

Priority is determined by community demand.

### Rust
- Use cases: High performance, embedded, WASM
- Can provide Python bindings via PyO3

### Go
- Use cases: Cloud-native, Kubernetes
- Can develop Operators

### Java
- Use cases: Enterprise-grade
- Spring Boot Starter

### TypeScript
- Use cases: Frontend, Node.js
- NPM package

---

## Milestones

| Version | Goal | Expected Timeline |
|------|------|----------|
| v0.1.0 | Protocol specification complete (including functional module definition + External Binding specification) | - |
| v0.2.0 | Python Core MVP | - |
| v0.3.0 | Core Complete (ACL, Middleware, Observability) | - |
| v0.4.0 | CLI tools | - |
| v0.5.0 | Advanced Features (hot-reloading, OpenTelemetry) | - |
| v1.0.0 | Production ready | - |

---

## Contributing Guide

### Priority Contribution Areas

1. **Protocol specification review** - Identify issues or ambiguities in the specification
2. **Python implementation** - Core component development
3. **Test cases** - Unit tests, integration tests
4. **Documentation** - Usage documentation, examples

### How to Contribute

1. Fork the repository
2. Create a feature branch
3. Submit a PR
4. Await review

---

## Related Resources

- [Protocol Specification](./PROTOCOL_SPEC.md)
- [MCP Documentation](https://modelcontextprotocol.io/)
- [A2A Protocol](https://github.com/a2aproject/A2A)
- [Pydantic](https://docs.pydantic.dev/)
- [Pluggy](https://pluggy.readthedocs.io/)
