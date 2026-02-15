# apcore — AI-Perceivable Core Scope Definition

> Clarify what we do and what we don't do.

## 1. Project Positioning

**One-line definition**:
> apcore (AI-Perceivable Core) is a Schema-driven module development framework that makes every interface naturally perceivable and understandable by AI.

**Core problem**:
> How to build modules that can be both invoked by code and perceived and understood by AI/LLMs?

**Answer**:
> Enforce modules to define `input_schema` / `output_schema` / `description`, making modules inherently AI-Perceivable.

**Positioning**:
```
apcore = General-purpose module development framework + Mandatory AI-Perceivable standard

Not: A framework that can only be used in AI scenarios
But: A general-purpose framework that is inherently AI-Perceivable
```

**Analogy**:
- apcore is similar to "Django/Spring" (defines how to build modules)
- MCP is similar to "HTTP protocol" (defines communication format)
- apflow is "an application built on apcore"

---

## 2. Core Responsibilities

> This section uses RFC 2119 keywords; see [PROTOCOL_SPEC.md §1.5](./PROTOCOL_SPEC.md#15-normative-keywords) for their meanings.

### 2.1 Module Standardization
- [ ] **Definition (MUST)**: All modules **must** define ID, input_schema, output_schema, and description
- [ ] **Discovery (MUST)**: The framework **must** support automatic module discovery based on directory scanning
- [ ] **Loading (SHOULD)**: The framework **should** support lazy loading and dependency injection
- [ ] **Invocation (MUST)**: The framework **must** automatically perform input validation, output validation, and error handling upon invocation

### 2.2 Schema System
- [ ] **Definition (MUST)**: All modules **must** define input_schema / output_schema, based on JSON Schema Draft 2020-12
- [ ] **Validation (MUST)**: The framework **must** perform Schema validation on input and output at runtime
- [ ] **Conversion (SHOULD)**: The framework **should** support conversion from YAML Schema to language-native types

### 2.3 Naming and Addressing
- [ ] **Directory as ID (MUST)**: The directory path **must** serve as the single source of truth for the module ID
- [ ] **ID Map (SHOULD)**: The framework **should** provide automatic cross-language ID conversion, and **may** allow overrides via YAML configuration

### 2.4 Access Control
- [ ] **ACL (MUST)**: The framework **must** provide an inter-module invocation access control mechanism
- [ ] **Audit (SHOULD)**: The framework **should** support invocation audit logs

### 2.5 Observability
- [ ] **Tracing (MUST)**: trace_id **must** be propagated throughout the call chain
- [ ] **Logging (SHOULD)**: The framework **should** provide structured logging support
- [ ] **Metrics (MAY)**: The framework **may** collect metrics such as invocation count, latency, etc.

### 2.6 Extension Mechanism
- [ ] **Middleware (MUST)**: The framework **must** provide before/after/on_error middleware hooks
- [ ] **Extension Points (SHOULD)**: The framework **should** provide replaceable extension points such as loaders, executors, etc.

### 2.7 Existing Application Integration
- [ ] **`module()` Registration (MUST)**: The framework **must** provide a `module()` mechanism to wrap existing callables (functions or methods) as standard modules; languages that support Decorator syntax should provide it in Decorator form, while languages that don't should provide it as a function call
- [ ] **External Schema Binding (MUST)**: The framework **must** support YAML binding files to map existing functions as modules with zero code modification
- [ ] **Type Inference (MUST)**: The framework **must** support automatic JSON Schema generation from language-native type information

---

## 3. Appendix Reference (Non-core, for Reference Only)

The following content is **not part of the core protocol** and serves only as **usage scenario mapping references**:

### 3.1 AI Protocol Mapping Reference
- MCP (Model Context Protocol) mapping approach
- A2A (Agent-to-Agent) mapping approach
- OpenAI Function Calling mapping approach

> These are references for "how to expose apcore modules to external AI systems", not core apcore functionality.

---

## 4. Out of Scope (Won't Do)

Things explicitly **not within the project scope**:

| Won't Do | Reason | Owned By |
|------|------|--------|
| **Workflow Engine** | Application-layer orchestration logic, not framework core | apflow and other upstream projects |
| **MCP/A2A Adapters** | Usage scenarios, not core protocol | Independent adapter projects |
| **Concrete Business Modules** | We are a framework, not an application | Upstream projects |
| **LLM Invocation Wrappers** | Stay neutral, don't bind to a specific LLM | LangChain/LlamaIndex |
| **Agent Strategies** | Too high-level, belongs to application logic | CrewAI/AutoGen |
| **Distributed Execution** | Advanced runtime feature | Independent extension projects |
| **UI/CLI Applications** | Focus on core protocol | Upstream projects |
| **Specific Cloud Service Integrations** | Stay neutral | Extension packages |
| **Framework Adapters** (Flask/FastAPI/Django) | Keep core pure, belongs to ecosystem projects | Independent repositories (e.g., apcore-fastapi) |

> apcore only defines the adapter interface specification (see [Adapter Development Guide](./docs/guides/adapter-development.md)); it does not include any framework-specific implementations. The community or official team can develop adapters in independent repositories (e.g., `apcore-fastapi`, `apcore-flask`, `apcore-django`), built on top of the core `module()` and External Binding mechanisms.

---

## 5. User Story Validation

Use these scenarios to validate whether our boundaries are correct:

### Scenario 1: Python Developer Building an AI Application
```
✅ I defined a database query module and it was automatically discovered by the framework
✅ The framework automatically validates input and output
✅ I can invoke other modules from my code
❌ The framework does not provide LLM invocation (I use the openai library myself)
❌ The framework does not provide an Agent loop (I write my own or use LangGraph)
```

### Scenario 2: Go Developer Wants to Use a Python Module
```
✅ The framework defines a cross-language ID specification
✅ The framework defines a Schema specification
⚡ Cross-language invocation requires RPC (provided by extension packages)
```

### Scenario 3: Team Wants to Define Workflows with YAML
```
✅ The core framework provides standardized module invocation capabilities
❌ Workflow engine is not within apcore's scope (implemented by upstream projects like apflow)
✅ The team can use any workflow solution (apflow, custom-built, etc.)
```

### Scenario 4: Wanting to Expose Modules to Claude/ChatGPT
```
✅ Modules have standard Schema, inherently MCP/OpenAI compatible
❌ MCP Server adapter is not within apcore's scope
✅ Refer to the appendix mapping documentation to implement adapters on your own
```

### Scenario 5: Existing Python Application Wants AI-Perceivable Capabilities
```
✅ Add the @module decorator to existing functions to automatically become apcore modules
✅ Use module(service.method, id=...) to register existing class methods without modifying source code
✅ The framework automatically generates Schema from type annotations
✅ Existing code logic remains completely unchanged
✅ Can also use YAML binding files with absolutely no source code modification
❌ The framework does not automatically scan Flask routes (independent repository ecosystem project)
```

---

## 6. Core vs Extension Decision Table

| Feature | Core? | Reason |
|------|-------|------|
| Module definition/discovery/loading | ✅ Core | Most fundamental capability |
| Schema definition/validation | ✅ Core | Required for AI-Perceivable |
| Canonical ID | ✅ Core | Cross-language foundation |
| ACL permissions | ✅ Core | Security foundation |
| Error handling | ✅ Core | Required |
| Tracing/logging | ✅ Core | Observability foundation |
| Middleware mechanism | ✅ Core | Extension foundation |
| **Workflow Engine** | ⚡ Extension | Upper-layer orchestration logic |
| **MCP Adapter** | ⚡ Extension | Protocol adaptation |
| **A2A Adapter** | ⚡ Extension | Protocol adaptation |
| **Distributed Execution** | ⚡ Extension | Advanced feature |
| **Web UI** | ❌ Won't Do | Out of scope |
| `module()` registration (Decorator / function call) | ✅ Core | Least intrusive integration for existing applications |
| External Schema Binding | ✅ Core | Zero code modification integration |
| Type inference Schema generation | ✅ Core | Foundational capability for module() and Binding |
| **Framework Adapters** | ❌ Won't Do (independent repository) | Keep core pure, belongs to ecosystem projects |
| **AI-Assisted Migration Tools** | ⚡ Extension | Developer experience tooling |

---

## 7. Final Confirmation

### Core Protocol Includes:
1. Naming conventions (Directory as ID + ID Map)
2. Directory structure (Directory)
3. Schema specification (Schema)
4. Module specification (Module)
5. Access control configuration (ACL)
6. Error handling (Error)
7. Observability (Observability)
8. Extension mechanism (Extension)
9. Configuration specification (Configuration)
10. Existing application integration (module() + External Binding)

### Appendix Reference (Non-core):
- AI protocol mapping references (MCP, A2A, OpenAI Functions)

---

## 8. Confirmed Decisions

- [x] Workflow: **Won't Do** (application-layer logic, implemented by upstream projects like apflow)
- [x] MCP/A2A adaptation: **Won't Do** (only provide mapping reference documentation)
- [x] Distributed execution: **Won't Do** (runtime feature, not protocol core)
- [ ] Schema and Meta files: Keep separated
- [x] Existing application integration: **Core** (`module()` registration and external Schema binding are core standards)
- [x] Framework adapters: **Won't Do** (independent repositories, apcore only provides adapter interface specification)
- [x] API naming: **No prefix** (rely on language namespaces, except C language which uses `apcore_` prefix)

## 9. Document Structure

```
apcore/
├── PROTOCOL_SPEC.md          # Core protocol specification (13 chapters)
├── SCOPE.md                  # Scope definition
├── README.md                 # Project introduction
└── docs/                     # Detailed documentation
    ├── concepts.md           # Core concepts
    ├── architecture.md       # Internal architecture
    ├── api/                  # API reference
    ├── guides/               # Usage guides
    └── spec/                 # Framework specification
```
