# apcore — AI-Perceivable Core Framework Specification

> **Canonical Specification** - This document is the authoritative specification for the apcore protocol

> Version: 1.0.0-draft
> Status: Draft Specification (RFC 2119 Conformant)
> Stability: Specification content is stable, pending reference implementation verification
> Last Updated: 2026-02-07

---

### Table of Contents

- [1. Overview](#1-overview)
- [2. Naming Specification](#2-naming-specification-naming-specification)
- [3. Directory Specification](#3-directory-specification-directory-specification)
- [4. Schema Specification](#4-schema-specification-schema-specification)
- [5. Module Specification](#5-module-specification-module-specification)
- [6. ACL Specification](#6-acl-specification-acl-specification)
- [7. Error Handling Specification](#7-error-handling-specification-error-handling-specification)
- [8. Configuration Specification](#8-configuration-specification-configuration-specification)
- [9. Observability Specification](#9-observability-specification-observability-specification)
- [10. Extension Mechanism](#10-extension-mechanism-extension-mechanism)
- [11. SDK Implementation Guide](#11-sdk-implementation-guide-sdk-implementation-guide)
- [12. Versioning](#12-versioning-versioning)
- [13. Appendix](#13-appendix)
- [Revision History](#revision-history)

---

## 1. Overview

### 1.1 Project Positioning

apcore (AI-Perceivable Core) is a **Schema-driven module development framework** that makes every interface naturally perceivable and understandable by AI.

**One-sentence definition**:
> apcore is a universal module development framework that makes modules both callable by code and perceivable and understandable by AI through enforced Schema definitions.

**Positioning**:
- **Universal Development Framework**: Not just an AI framework, but a standard module development framework
- **Enforced AI-Perceivable**: Schema is mandatory, making modules naturally perceivable and understandable by AI
- **Complementary to MCP/A2A**: MCP/A2A define communication protocols, apcore defines module construction specifications
- Foundation for other projects (such as apflow)

### 1.2 Core Principles

| Principle | Description |
|------|------|
| **Schema-driven** | All modules enforce definition of `input_schema` / `output_schema` / `description` |
| **Three-layer Metadata** | Core (enforced Schema) + Annotations (behavior Annotations) + Extensions (free metadata) |
| **Directory as ID** | Directory path automatically maps to module ID, zero manual configuration |
| **AI-Perceivable** | Schema + Annotations enable AI/LLM to perceive and understand modules, this is a design requirement |
| **Universal Framework** | Modules can be called by code/AI/HTTP/CLI in any manner |

### 1.3 Design Goals

- **Universality**: Modules can be called by code, AI, HTTP, CLI, etc. in any manner
- **AI Perceptibility**: Enforced Schema ensures LLM can perceive and understand modules
- **Developer Experience**: Directory as ID, zero configuration, automatic discovery
- **Cross-language**: Specification supports implementation in any programming language, Python as reference implementation
- **Extensibility**: ACL, middleware, observability

### 1.4 Relationship with MCP/A2A

apcore is a **module construction specification**, MCP/A2A is a **communication protocol**. They are complementary:

```
┌─────────────────────────────────────────────────────────────┐
│              apcore — AI-Perceivable Core                    │
│                                                             │
│  Solves: How to develop AI-Perceivable modules              │
│  - Directory as ID / Schema-driven / ACL / Observability / Middleware │
└─────────────────────────────────────────────────────────────┘
                           ↓ Modules can be exposed as
┌─────────────┬─────────────┬─────────────┬─────────────────┐
│ MCP Server  │  HTTP API   │  CLI Tool   │  gRPC Service   │
│ (Claude)    │  (REST)     │  (Terminal) │  (Microservice) │
└─────────────┴─────────────┴─────────────┴─────────────────┘
```

**apcore focuses on "how to build AI-perceivable modules", MCP focuses on "how to call tools".**

### 1.5 Specification Keywords

Keywords in this document are interpreted according to [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) definitions.

| English | Chinese | Meaning |
|------|------|------|
| **MUST** / **REQUIRED** / **SHALL** | Must | Absolute requirement, non-compliance means non-conformance |
| **MUST NOT** / **SHALL NOT** | Forbidden | Absolutely prohibited |
| **SHOULD** / **RECOMMENDED** | Should | Strongly recommended, can deviate only with sufficient reason |
| **SHOULD NOT** / **NOT RECOMMENDED** | Should Not | Strongly discouraged, unless there's sufficient reason |
| **MAY** / **OPTIONAL** | May | Completely optional, implementer decides |

When this document uses bolded forms of the above keywords, their semantics strictly follow RFC 2119 definitions. Normal font "must", "should", etc. do not have normative meaning.

### 1.6 Term Definitions

Core terms used in this specification are defined as follows:

| Term | English | Definition |
|------|------|------|
| Module | Module | Basic execution unit of apcore, encapsulates a function, **must** define `input_schema`, `output_schema`, and `description` (≤200 characters). Can optionally define `documentation` (≤5000 characters) to provide detailed documentation |
| Schema | Schema | Data structure definition based on JSON Schema Draft 2020-12, used for validating input/output and for AI/LLM understanding |
| Canonical ID | Canonical ID | Globally unique identifier for a module, automatically generated from directory path, format is dot-separated snake_case (e.g., `executor.email.send_email`) |
| Registry | Registry | Core component responsible for module discovery, registration, loading, and management |
| Executor | Executor | Core component responsible for module invocation execution, handles Schema validation, ACL checking, middleware dispatching |
| Context | Context | Runtime context object during module execution, carries trace_id, call chain, identity information, and shared state |
| Access Control List | ACL | Set of rules defining inter-module invocation permissions, based on caller/target pattern matching |
| Middleware | Middleware | Interceptor running before and after module execution, executes in onion model, can modify input/output |
| Extension Point | Extension Point | Replaceable component interface provided by framework (e.g., SchemaLoader, ModuleLoader), allows custom implementation |
| Annotations | Annotations | Module-level behavior metadata (readonly, destructive, etc.), helps AI/LLM make invocation decisions |
| Metadata | Metadata | Completely open key-value dictionary for storing extension information, framework does not validate its content |
| Entry Point | Entry Point | Code entry location of module, format is `filename:ClassName`, can be auto-inferred or manually configured |
| Call Chain | Call Chain | Complete list of module ID paths from root invocation to current invocation, used for loop detection and depth limiting |
| Trace ID | Trace ID | Identifier uniquely identifying a complete invocation chain, **must** be UUID v4 format |
| Identity | Identity | Structured expression of caller identity (user/service/Agent/API Key/system), ACL engine depends on it |

### 1.7 API Naming Conventions

apcore public API uses concise universal names (e.g., `Module`, `Context`, `Registry`),
relying on language-native namespace mechanisms to avoid conflicts:

| Language | Namespace Isolation Method | Example |
|------|----------------|------|
| Python | Package import | `from apcore import Module` / `import apcore` |
| Go | Package name qualification | `apcore.Module(...)` |
| Rust | Module path | `apcore::Module` |
| TypeScript | Module import | `import { Module } from 'apcore'` |
| Java | Package path | `import com.apcore.Module` |
| C | Name prefix | `apcore_module_t`, `apcore_module(...)` |

Implementations **MUST** follow these naming rules:
- In languages with namespace mechanisms, **MUST NOT** add redundant prefixes to public APIs
- In languages without namespace mechanisms (e.g., C), **MUST** use `apcore_` prefix
- Error types **SHOULD** be prefixed with their domain (e.g., `ModuleError`, `SchemaValidationError`)

---

## 2. Naming Specification (Naming Specification)

### 2.1 Directory as ID (Core Rule)

**Directory path is the single source of truth for module IDs**. IDs are automatically generated from directory paths, zero configuration.

Implementations **must** convert directory paths to Canonical IDs according to the following algorithm:

```
Algorithm: directory_to_canonical_id(file_path, extensions_root)

Input:
  file_path      — Complete relative path of module file (e.g., "extensions/executor/validator/db_params.py")
  extensions_root — Extension root directory name (default "extensions")

Output:
  canonical_id   — Dot-separated module ID (e.g., "executor.validator.db_params")

Preconditions:
  - file_path must start with extensions_root + "/"
  - file_path must contain file extension

Steps:
  1. relative_path ← Remove extensions_root + "/" prefix from file_path
  2. relative_path ← Remove file extension (last "." and everything after it)
  3. segments ← Split relative_path by "/"
  4. For each segment, perform validation:
     a. If segment is empty string → Throw INVALID_PATH error
     b. If segment doesn't match /^[a-z][a-z0-9_]*$/ → Throw INVALID_SEGMENT error
  5. canonical_id ← Join all segments with "."
  6. If len(canonical_id) > 128 → Throw ID_TOO_LONG error
  7. Return canonical_id

Complexity: O(n), where n is the number of characters in the path
```

```yaml
directory_to_id:
  # Rule: Remove extensions/ prefix and file extension, convert path separator to dot
  rule: "extensions/{path}/{name}.{ext} → {path}.{name}"

  # Examples
  examples:
    - file: "extensions/executor/validator/db_params.py"
      id: "executor.validator.db_params"

    - file: "extensions/api/handler/task_submit.py"
      id: "api.handler.task_submit"

    - file: "extensions/orchestrator/engine/task_flow.py"
      id: "orchestrator.engine.task_flow"

  # ID format constraints
  format:
    pattern: "^[a-z][a-z0-9_]*(\\.[a-z][a-z0-9_]*)*$"
    separator: "."
    case: "snake_case"
    max_length: 128
```

### 2.2 ID Map (Cross-language Conversion)

**ID Map** module handles cross-language ID conversion, supporting automatic recognition and manual configuration. Implementations **must** support canonical conversion from various language native formats to Canonical ID.

```
Algorithm: normalize_to_canonical_id(local_id, language)

Input:
  local_id  — Language-native format ID (e.g., Rust's "executor::validator::db_params")
  language  — Source language identifier (python | rust | go | java | typescript)

Output:
  canonical_id — Dot-separated snake_case format Canonical ID

Steps:
  1. Determine separator sep based on language:
     - python: "."  |  rust: "::"  |  go: "."  |  java: "."  |  typescript: "."
  2. segments ← Split local_id by sep
  3. For each segment, perform case normalization:
     - If segment is PascalCase → Convert to snake_case
     - If segment is camelCase → Convert to snake_case
     - If segment is already snake_case → Keep unchanged
  4. canonical_id ← Join all segments with "."
  5. Validate canonical_id conforms to ID EBNF syntax (see §2.7)
  6. Return canonical_id
```

```yaml
# apcore.yaml

id_map:
  # Auto-detection: Determine language by file extension
  auto_detect: true

  # Built-in language conversion rules
  languages:
    python:
      extensions: [".py"]
      separator: "."
      file_case: "snake_case"
      class_case: "PascalCase"
      example:
        id: "executor.validator.db_params"
        file: "executor/validator/db_params.py"
        class: "DbParamsValidator"

    rust:
      extensions: [".rs"]
      separator: "::"
      file_case: "snake_case"
      struct_case: "PascalCase"
      example:
        id: "executor.validator.db_params"
        file: "executor/validator/db_params.rs"
        local_id: "executor::validator::db_params"
        struct: "DbParamsValidator"

    go:
      extensions: [".go"]
      separator: "."
      file_case: "snake_case"
      struct_case: "PascalCase"
      example:
        id: "executor.validator.db_params"
        file: "executor/validator/db_params.go"
        struct: "DbParamsValidator"
        package: "validator"

    java:
      extensions: [".java"]
      separator: "."
      file_case: "PascalCase"
      class_case: "PascalCase"
      example:
        id: "executor.validator.db_params"
        file: "executor/validator/DbParams.java"
        class: "DbParamsValidator"
        package: "com.example.extensions.executor.validator"

    typescript:
      extensions: [".ts", ".tsx"]
      separator: "."
      file_case: "camelCase"
      class_case: "PascalCase"
      example:
        id: "executor.validator.db_params"
        file: "executor/validator/dbParams.ts"
        class: "DbParamsValidator"

  # Special mappings (override auto rules)
  overrides:
    # When automatic conversion doesn't meet requirements, manually specify
    "executor.validator.db_params":
      java:
        class: "com.mycompany.DbParamsValidator"
        package: "com.mycompany.validators"
```

### 2.3 Special Word Handling

Rules for handling abbreviations when converting class names:

```yaml
abbreviations:
  # Common abbreviations
  words: [http, api, db, id, url, sql, json, xml, html, css, tcp, udp, ip]

  # Rule: Treat abbreviations as normal words, capitalize only first letter
  # Reason: HttpJsonParser is more readable than HTTPJSONParser

  # Examples
  examples:
    id: "api.handler.http_json_parser"
    class: "HttpJsonParser"      # Not HTTPJSONParser
```

### 2.4 Version Number Handling

```yaml
versioning:
  # Filename format
  pattern: "{name}_v{major}.{ext}"

  # Examples
  examples:
    - file: "db_params.py"       # No version = default version
      id: "executor.validator.db_params"

    - file: "db_params_v2.py"
      id: "executor.validator.db_params_v2"

  # Version resolution
  resolution:
    default: "latest"            # Use latest version by default
    explicit: true               # Allow explicit specification: module_id@v2
```

### 2.5 Reserved Words

```yaml
reserved_words:
  # Framework reserved
  framework: [system, internal, core, apcore, plugin, schema, acl]

  # Programming language keywords
  keywords: [class, def, import, return, if, else, for, while, true, false, null, none]

  # Disallowed patterns
  patterns:
    - "^_.*"         # Starting with underscore
    - "^[0-9].*"     # Starting with digit
    - ".*__.*"       # Double underscore
```

### 2.6 ID Conflict Detection

Implementations **must** perform conflict detection during module scanning, module registration, and dynamic loading.

```
Algorithm: detect_id_conflicts(new_id, existing_ids, reserved_words)

Input:
  new_id         — Canonical ID to be registered
  existing_ids   — Set of already registered IDs
  reserved_words — Set of reserved words (§2.5)

Output:
  conflict_result — { type: string, severity: "error" | "warning", message: string } | null

Steps:
  1. Exact duplication detection:
     If new_id ∈ existing_ids → Return { type: "duplicate_id", severity: "error" }
  2. Reserved word detection:
     For each segment of new_id (split by "."):
       If segment ∈ reserved_words → Return { type: "reserved_word", severity: "error" }
  3. Case collision detection:
     normalized_new ← lowercase(new_id)
     For each existing_id ∈ existing_ids:
       normalized_existing ← lowercase(existing_id)
       If normalized_new == normalized_existing and new_id ≠ existing_id:
         → Return { type: "case_collision", severity: "warning" }
  4. Return null (no conflict)

Complexity: O(n), where n is the number of registered IDs
```

```yaml
conflict_detection:
  when: [Module scanning, Module registration, Dynamic loading]

  types:
    duplicate_id:
      description: "Same ID already exists"
      action: "error"

    reserved_word:
      description: "Uses reserved word"
      action: "error"

    case_collision:
      description: "Only differs in case (cross-language conflict risk)"
      action: "warning"
```

### 2.7 ID Formal Grammar

The formal definition of Canonical ID uses EBNF notation. All implementations **must** reject IDs that do not conform to this grammar.

```ebnf
(* apcore Canonical ID EBNF *)

canonical_id    = segment , { "." , segment } ;
segment         = lower_alpha , { lower_alpha | digit | "_" } ;
lower_alpha     = "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i"
                | "j" | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r"
                | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z" ;
digit           = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;

(* Constraints *)
(* 1. canonical_id total length MUST NOT exceed 128 characters *)
(* 2. segment MUST NOT be a reserved word (see §2.5) *)
(* 3. segment MUST NOT start with a digit (guaranteed by production) *)
(* 4. segment MUST NOT contain consecutive double underscores "__" *)
```

Equivalent regular expression: `^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$`

---

## 3. Directory Specification (Directory Specification)

### 3.1 Standard Directory Structure

Implementations **must** follow the directory structure below. The nesting depth under `extensions/` directory (not including `extensions/` itself) **must not** exceed 8 levels.

```
{project_root}/
├── extensions/                   # Extensions directory (max depth: 8 levels)
│   ├── api/                      # Level 1 grouping: API layer
│   │   ├── validator/            # Level 2 grouping: Validators
│   │   │   ├── http_json_schema.{ext}
│   │   │   ├── http_json_schema_meta.yaml
│   │   │   └── ...
│   │   ├── parser/               # Level 2 grouping: Parsers
│   │   └── handler/              # Level 2 grouping: Handlers
│   │
│   ├── orchestrator/             # Level 1 grouping: Orchestration engine layer
│   │   ├── engine/               # Level 2 grouping: Engine
│   │   ├── validator/
│   │   └── parser/
│   │
│   └── executor/                 # Level 1 grouping: Executor layer
│       ├── handler/
│       └── validator/
│
├── schemas/                      # Schema definition directory (YAML)
│   └── {canonical_id}.schema.yaml
│
├── acl/                          # Access control configuration directory
│   └── {scope}_acl.yaml
│
└── apcore.yaml                   # Framework configuration file
```

### 3.2 Layer Definitions

| Level 1 Grouping | Responsibility | Typical Level 2 Groupings |
|----------|------|-------------|
| `api` | API entry layer, handles external requests | validator, parser, handler |
| `orchestrator` | Orchestration engine layer, task scheduling and process control | engine, validator, parser, scheduler |
| `executor` | Executor layer, actual business execution | handler, validator, connector |
| `common` | Common components | util, middleware, decorator |

### 3.3 ID Generation Rules

```yaml
id_generation:
  # Directory path → Canonical ID
  source: "directory_path"

  # Exclusions
  exclude:
    - extensions_root_dir    # Exclude "extensions/"
    - file_extension      # Exclude ".py", ".rs", etc.

  # Example
  example:
    input: "extensions/executor/validator/db_params.py"
    output: "executor.validator.db_params"
```

### 3.4 Symbolic Link Handling

Implementations **must not** follow symbolic links (symlinks) in default mode. If symbolic link support is needed, it **must** be enabled through explicit configuration:

```yaml
# apcore.yaml
extensions:
  follow_symlinks: false  # Default value, MUST NOT follow
```

- When symbolic link following is enabled, implementations **must** detect symbolic link loops
- The resolved path of symbolic links **must** still be within the `extensions_root` scope

### 3.5 Hidden File Handling

Implementations **must** ignore the following files and directories during module scanning:

| Pattern | Example | Description |
|------|------|------|
| Starts with `.` | `.git/`, `.env` | Hidden files/directories |
| Starts with `_` | `_internal/`, `_test.py` | Internal files/directories |
| `__pycache__/` | — | Python cache |
| `node_modules/` | — | Node.js dependencies |
| `*.pyc` | — | Python compiled files |

Implementations **may** extend the ignore pattern list through configuration.

### 3.6 Scanning Algorithm

Implementations **must** scan the extensions directory according to the following algorithm:

```
Algorithm: scan_extensions(extensions_root, config)

Input:
  extensions_root — Extensions root directory path
  config          — Scan configuration (follow_symlinks, ignore_patterns, max_depth)

Output:
  modules — List of [(file_path, canonical_id)]

Steps:
  1. If extensions_root doesn't exist or is not a directory → Throw CONFIG_ERROR
  2. modules ← []
  3. Recursively traverse extensions_root:
     For each entry (file or directory):
       a. If entry name matches ignore_patterns (§3.5) → Skip
       b. If entry is a symbolic link and config.follow_symlinks == false → Skip
       c. If entry is a directory:
          - If current depth >= config.max_depth (default 8) → Skip and issue warning
          - Otherwise → Recurse into it
       d. If entry is a file and extension belongs to supported_extensions:
          - canonical_id ← directory_to_canonical_id(entry.path, extensions_root)
          - If canonical_id passes validation → Append (entry.path, canonical_id) to modules
          - If validation fails → Log warning
  4. Perform detect_id_conflicts batch detection on modules
  5. Return modules

Complexity: O(n), where n is the number of filesystem entries
```

---

## 4. Schema Specification (Schema Specification)

### 4.1 Overview

All modules **must** define `input_schema` and `output_schema` to support:
- LLM tool calling (MCP compatible)
- Runtime data validation
- Automatic API documentation generation
- Cross-language interoperability

### 4.2 Schema Format

**Must** be based on **JSON Schema Draft 2020-12** ([RFC unpublished draft](https://json-schema.org/draft/2020-12/json-schema-core)), extending with LLM-friendly fields.

**Compliance Requirements:**

| Requirement | Level | Description |
|------|------|------|
| Draft 2020-12 core vocabulary | **MUST** | type, properties, required, $ref, etc. |
| Draft 2020-12 validation vocabulary | **MUST** | minimum, maximum, pattern, enum, etc. |
| `$schema` declaration | **SHOULD** | Schema files **should** declare `$schema` field |
| `x-` extension prefix | **MUST** | Custom extension fields **must** be named with `x-` prefix |
| `additionalProperties` | **SHOULD** | input_schema **should** explicitly declare `additionalProperties: false` |

```yaml
# schemas/executor/validator/db_params.schema.yaml

$schema: "https://apcore.dev/schema/v1"
version: "1.0.0"
module_id: "executor.validator.db_params"

# Module description (LLM readable)
description: |
  Validates database operation parameters, checks table name format and SQL syntax safety.
  Suitable for pre-validation before executing SQL.

# Input Schema
input_schema:
  type: object
  properties:
    table:
      type: string
      pattern: "^[a-z][a-z0-9_]*$"
      description: "Target database table name"
      x-llm-description: "Database table name, only lowercase letters, numbers, and underscores allowed, must start with a letter"
      x-examples: ["user_info", "order_detail", "product_catalog"]

    sql:
      type: string
      description: "SQL statement"
      x-llm-description: "SQL statement to execute, will undergo safety checks"
      x-constraints: "Dangerous operations like DROP, TRUNCATE are not allowed"

    timeout:
      type: integer
      default: 30
      minimum: 1
      maximum: 300
      description: "Timeout in seconds"

  required: [table, sql]
  additionalProperties: false

# Output Schema
output_schema:
  type: object
  properties:
    valid:
      type: boolean
      description: "Whether validation passed"

    message:
      type: string
      description: "Validation result message"

    errors:
      type: array
      items:
        type: object
        properties:
          field:
            type: string
          code:
            type: string
          message:
            type: string
      description: "Error details list"

    warnings:
      type: array
      items:
        type: string
      description: "Warning messages"

  required: [valid]

# Error Schema (optional)
error_schema:
  type: object
  properties:
    code:
      type: string
      enum: [INVALID_TABLE, DANGEROUS_SQL, TIMEOUT, INTERNAL_ERROR]
    message:
      type: string
    details:
      type: object
  required: [code, message]
```

### 4.3 LLM Extension Fields

| Field | Type | Description |
|------|------|------|
| `x-llm-description` | string | Dedicated description for LLM (see usage guide below) |
| `x-examples` | array | Example values to help LLM understand value range |
| `x-constraints` | string | Business constraint description |
| `x-sensitive` | boolean | Mark sensitive fields (e.g., passwords), LLM should not log |

#### Relationship between `description` and `x-llm-description`

| Field | Audience | Purpose |
|------|------|------|
| `description` | All consumers (humans, AI, doc generators) | Universal field description, exported to AI protocols as field description |
| `x-llm-description` | AI/LLM only | Use only when AI description needs to be **significantly different** from human description |

**Use Cases:**

- Security warnings: AI needs additional security constraint hints (e.g., "only SELECT statements allowed")
- Usage guidance: Special behavior constraints when AI calls (e.g., "don't hard-code this value")
- Semantic supplementation: description is concise for humans, but AI needs more context to fill correctly

**Export Rules:**

- Adapters **SHOULD** prefer `x-llm-description`: If field has both `description` and `x-llm-description`, replace `description` with `x-llm-description` when exporting to AI protocols
- If field has only `description`, export as-is
- Strict mode export (§4.16) strips all `x-*` fields, always using `description`

**Anti-pattern:** Don't add `x-llm-description` to every field. For most fields, `description` is sufficient for both humans and AI, use `x-llm-description` only when there's a clear difference in needs.

### 4.4 Module Behavior Annotations (Annotations)

**Annotations are module-level behavior metadata** that help AI/LLM make invocation decisions.

```yaml
# Module behavior annotations specification
annotations:
  type: object
  properties:
    readonly:
      type: boolean
      default: false
      description: "Whether module is read-only (no side effects). true means no state modification."

    destructive:
      type: boolean
      default: false
      description: "Whether module has destructive operations. true means may delete/overwrite data."

    idempotent:
      type: boolean
      default: false
      description: "Whether module is idempotent. true means repeated calls with same parameters won't produce additional side effects."

    requires_approval:
      type: boolean
      default: false
      description: "Whether requires human approval. true means AI should seek user consent before calling."

    open_world:
      type: boolean
      default: true
      description: "Whether involves external systems. true means connects to external APIs/services/network."
```

**Annotations Design Principles:**

| Principle | Description |
|------|------|
| All optional | Use default values when not defined |
| Hints | Are hints rather than enforced constraints, AI uses as reference |
| Aligned with MCP | Field design compatible with MCP ToolAnnotations |
| Extensible | Can add new fields without breaking compatibility |

**AI usage of Annotations decision examples:**

```
readonly=true         → AI can call safely, no confirmation needed
destructive=true      → AI should warn user before calling
idempotent=true       → AI can safely retry failed calls
requires_approval=true → AI must seek user consent
open_world=true       → AI knows this call involves external systems, may be slow
```

### 4.5 Module Usage Examples (Examples)

**Examples provide concrete input/output examples** to help AI/LLM understand complex modules more accurately.

```yaml
# Module examples specification
examples:
  type: array
  items:
    type: object
    properties:
      title:
        type: string
        description: "Example title"

      description:
        type: string
        description: "Example description (optional)"

      inputs:
        type: object
        description: "Example input (must conform to input_schema)"

      output:
        type: object
        description: "Example output (optional, conforms to output_schema)"
    required: [title, inputs]
```

**Example:**

```yaml
examples:
  - title: "Send plain text email"
    description: "Send plain text email via SMTP"
    inputs:
      to: "user@example.com"
      subject: "Hello"
      body: "World"
    output:
      success: true
      message_id: "msg_123"

  - title: "Send HTML email"
    description: "Send HTML format email to multiple recipients via SMTP"
    inputs:
      to: ["user1@example.com", "user2@example.com"]
      subject: "Notification"
      html: "<h1>Hello</h1>"
      smtp_host: "smtp.example.com"
      smtp_port: 587
    output:
      success: true
      message_id: "msg_456"
```

**Examples Design Principles:**

| Principle | Description |
|------|------|
| All optional | Not providing examples doesn't affect module functionality |
| Inputs must be valid | inputs must conform to input_schema |
| Multi-scenario coverage | Recommend covering typical usage and edge cases |
| Aligned with Anthropic | Similar to Anthropic input_examples, improves LLM understanding |

### 4.6 Module Extension Metadata (Metadata)

**Metadata is a completely open dict** for custom extensions beyond the framework protocol.

```yaml
# Module metadata specification
metadata:
  type: object
  additionalProperties: true
  description: "Free extension metadata, framework doesn't validate content"
```

**Common usage examples:**

```yaml
metadata:
  # Performance hints
  cost_per_call: 0.001
  avg_latency_ms: 500
  max_latency_ms: 5000

  # Data sensitivity
  data_sensitivity: ["PII"]

  # Operations info
  owner: "email-team"
  sla: "99.9%"
  documentation_url: "https://docs.example.com/send-email"

  # Custom business fields
  billing_category: "communication"
```

**Metadata Design Principles:**

| Principle | Description |
|------|------|
| Completely free | Framework doesn't validate or interpret metadata content |
| Can be used by middleware | Middleware can read metadata for custom logic |
| Doesn't affect execution | metadata doesn't participate in module execution flow |
| Open extension | When custom needs arise, metadata is the first landing spot |

### 4.7 Three-layer Metadata Summary

```
┌───────────────────────────────────────────────────────────┐
│                  Module Metadata Three-layer Design        │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────────────────────────────────────┐     │
│  │  Core Layer (Required)                          │     │
│  │  input_schema / output_schema / description     │     │
│  │  → AI understands "what" the module does        │     │
│  └─────────────────────────────────────────────────┘     │
│                          ↓                                │
│  ┌─────────────────────────────────────────────────┐     │
│  │  Annotation Layer (Optional, type-safe)         │     │
│  │  annotations / examples / name / tags / version │     │
│  │  → AI understands "how to use" the module       │     │
│  └─────────────────────────────────────────────────┘     │
│                          ↓                                │
│  ┌─────────────────────────────────────────────────┐     │
│  │  Extension Layer (Optional, free dict)          │     │
│  │  metadata: dict[str, Any]                       │     │
│  │  → Custom extension needs (framework unconstrained)  │
│  └─────────────────────────────────────────────────┘     │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### 4.8 Description and Documentation Field Specification

#### 4.8.1 Design Principles

apcore adopts the **Progressive Disclosure** design pattern, referencing the [Claude Agent Skills standard](https://github.com/anthropics/skills), splitting module information into brief description and detailed documentation:

| Field | Required | Length Limit | Markdown | Purpose |
|------|--------|----------|----------|------|
| `description` | **Required** | ≤200 characters | No | Brief module function description for AI quick matching and understanding |
| `documentation` | Optional | ≤5000 characters | Yes | Detailed documentation including usage scenarios, constraints, configuration requirements |

**Core Philosophy:**

```
AI Module Discovery Flow:
├─ Phase 1: Module Discovery
│   └─ Read all modules' description (≤200 chars)
│   └─ Quickly determine candidate modules
│
├─ Phase 2: Invocation Decision
│   └─ Read candidate modules' Schema + Annotations
│   └─ Optionally read documentation (detailed info)
│   └─ Confirm parameters and behavior characteristics
│
└─ Phase 3: Execution Preparation
    └─ Read detailed documentation (documentation/docstring)
    └─ Learn detailed usage and examples
```

**Standard Correspondence:**

| apcore | Claude Skill | OpenAPI | JSON Schema |
|--------|-------------|---------|-------------|
| `description` | `description` | `summary` | `title` |
| `documentation` | `instructions` | `description` | `description` |

#### 4.8.2 Description Field

**MUST:**
- Length limit: ≤200 characters (approx. 100 Chinese characters)
- Content requirements: Explain "what it does" + "when to use" + "key characteristics"
- Avoid redundancy: Parameter info already defined in Schema need not be repeated

**SHOULD:**
- Highlight function verbs and usage scenarios
- Explain key behavior characteristics (e.g., idempotency, side effects)
- Single line or brief paragraph (≤3 lines)

**SHOULD NOT:**
- Include detailed usage examples (put in documentation or docstring)
- Repeat parameter lists from Schema
- Exceed 200 characters

**Recommended Format:**

```python
# Format 1: Single-line concise description (recommended)
description = "Send email to specified recipients. Uses SMTP protocol, non-idempotent operation, requires mail server configuration."

# Format 2: Structured brief description (suitable for complex modules)
description = """Send email to specified recipients.
Sends text/HTML emails via SMTP, non-idempotent operation. Use cases: notifications, verification codes, reports."""
```

**YAML format:**

```yaml
module_id: "executor.email.send_email"
description: "Send email to specified recipients. Uses SMTP protocol, non-idempotent operation, requires mail server configuration."
```

#### 4.8.3 Documentation Field

The `documentation` field is **optional**, used to provide detailed module documentation. Simple modules can use only the `description` field.

**SHOULD use documentation scenarios:**
- Module has complex configuration requirements
- Need to explain important usage constraints or limitations
- Have multiple usage scenarios to explain
- Need to provide detailed error handling explanation

**Format Requirements:**
- Supports Markdown format
- Length limit: ≤5000 characters
- Recommend using structured format (headings, lists, etc.)

**Recommended Content Structure:**

```markdown
## Functionality
Brief explanation of module's detailed functionality

## Configuration Requirements
- Required configuration items
- Dependent external services

## Usage Scenarios
- Scenario 1: ...
- Scenario 2: ...

## Limitations and Constraints
- Performance limits (e.g., rate limits)
- Data size limits
- Other constraints

## Notes
Important usage considerations
```

**Example:**

```python
class SendEmailModule(Module):
    description = "Send email to specified recipients. Uses SMTP protocol, non-idempotent operation, requires mail server configuration."

    documentation = """
# Functionality
Sends emails via SMTP protocol, supports plain text and HTML formats.

## Configuration Requirements
- Must configure SMTP server info in apcore.yaml
- Need to provide valid SMTP authentication credentials

## Usage Scenarios
- Send notification emails
- Send verification codes
- Send reports

## Limitations
- Gmail: 500 emails/day
- Attachment size: ≤25MB
- Timeout: 30 seconds

## Error Handling
Email send failures throw EmailSendError exception with detailed error info.
"""
```

**YAML format:**

```yaml
module_id: "executor.email.send_email"
description: "Send email to specified recipients. Uses SMTP protocol, non-idempotent operation, requires mail server configuration."
documentation: |
  # Functionality
  Sends emails via SMTP protocol, supports plain text and HTML formats.

  ## Configuration Requirements
  - Must configure SMTP server info in apcore.yaml
  - Need to provide valid SMTP authentication credentials

  ## Usage Scenarios
  - Send notification emails, verification codes, reports

  ## Limitations
  - Gmail: 500 emails/day
  - Attachment size: ≤25MB
```

#### 4.8.4 Relationship with Docstring

apcore modules can use three forms of documentation simultaneously:

| Location | Purpose | Audience | Format |
|------|------|------|------|
| `description` | Quick module understanding | AI module discovery phase | Plain text, ≤200 chars |
| `documentation` | Detailed usage documentation | AI invocation decision phase | Markdown, ≤5000 chars |
| Python docstring | Code-level documentation | Developers, IDE | reStructuredText/Markdown |

**Priority and Usage Recommendations:**

1. **description** (required): All modules must define
2. **documentation** (recommended): Complex modules should define
3. **docstring** (optional): For developer reading, IDE can display

**Example:**

```python
class SendEmailModule(Module):
    """Email sending module (Python docstring, for developers to read)

    This is Python docstring for developers to view in IDE.
    Can include more detailed technical implementation notes, source code comments, etc.
    """

    # Used in AI module discovery phase
    description = "Send email to specified recipients. Uses SMTP protocol, non-idempotent operation, requires mail server configuration."

    # Used in AI invocation decision phase (optional)
    documentation = """
    # Functionality
    Sends emails via SMTP protocol, supports plain text and HTML formats.

    ## Configuration Requirements
    ...
    """
```

#### 4.8.5 Backward Compatibility

To maintain backward compatibility, the framework will automatically handle old format modules:

**Automatic Migration Rules:**

```python
# Existing module (only description, ≤200 chars)
if description exists and len(description) <= 200:
    # Keep unchanged, fully compatible
    description = description
    documentation = None

# description exceeds 200 chars (not recommended, but will warn)
elif description exists and len(description) > 200:
    # Issue warning, suggest migration
    warn("description exceeds 200 chars, consider moving details to 'documentation' field")
    # Keep as-is, but may affect AI performance
```

**Migration Recommendations:**

For existing long descriptions (>200 chars):
1. Extract first 200 chars as new description
2. Move complete content to documentation field
3. Or manually rewrite as brief description + detailed documentation

#### 4.8.6 AI Usage Flow Example

```
User request: "Send a welcome email to new user"

Step 1: Module Discovery (read all descriptions)
  → Scan 100 modules' descriptions (total ~20,000 characters)
  → Identify candidate module: executor.email.send_email
  → description: "Send email to specified recipients. Uses SMTP protocol, non-idempotent operation..."

Step 2: Invocation Decision (read Schema + Annotations + documentation)
  → input_schema: {to, subject, body}
  → output_schema: {success, message_id}
  → annotations: {requires_approval: true, open_world: true}
  → documentation: "# Functionality\nSends emails via SMTP protocol..."
  → Decision: User confirmation needed

Step 3: Execution Preparation
  → Learn configuration requirements: Need SMTP server configuration
  → Learn limitations: Gmail 500 emails/day
  → Prepare parameters: {"to": "new_user@example.com", ...}
```

#### 4.8.7 Performance Impact

**Token consumption comparison:**

| Approach | Module Discovery Phase | Invocation Decision Phase | Total |
|------|-------------|-------------|------|
| **Progressive Disclosure** (recommended) | ~20K tokens<br/>(100 modules × 200 chars) | ~5-10K tokens<br/>(1-2 candidate modules × 5000 chars) | ~25-30K |
| Load all full docs | ~500K tokens<br/>(100 modules × 5000 chars) | 0 | ~500K |

**Advantages:**
- Reduce 94% of initial token consumption
- Speed up module discovery (load only necessary info)
- Load detailed info on-demand (load only candidate modules' documentation)
- Better AI performance and response speed

---

### 4.9 Schema Loading Strategy

```yaml
# apcore.yaml
schema:
  # Loading strategy
  strategy: "yaml_first"  # yaml_first | native_first | yaml_only

  # yaml_first: Load from YAML first, native implementation can override
  # native_first: Prefer native implementation (Pydantic/struct), YAML as fallback
  # yaml_only: Only use YAML (pure cross-language scenario)

  paths:
    yaml_schemas: "./schemas"

  # Validation options
  validation:
    strict: true                # Strict mode: disallow extra fields
    coerce_types: true          # Type coercion
```

### 4.10 Language-specific Schema Implementations

| Language | Native Implementation | YAML Loading Support |
|------|----------|--------------|
| Python | Pydantic v2 | YAML → Pydantic dynamic generation |
| Rust | serde + validator | YAML → JSONSchema runtime validation |
| Go | struct + go-playground/validator | YAML → JSONSchema runtime validation |
| Java | Jackson + Bean Validation | YAML → POJO code generation |
| TypeScript | Zod / TypeBox | YAML → Zod schema generation |

### 4.11 Schema References ($ref)

Implementations **must** support `$ref` references and **must** resolve according to the following algorithm. Implementations **must** detect and reject circular references.

```
Algorithm: resolve_ref(ref_string, current_file, schemas_dir, visited_refs)

Input:
  ref_string   — $ref value (e.g., "./common/error.schema.yaml#/definitions/ErrorDetail")
  current_file — Current Schema file path
  schemas_dir  — schemas root directory
  visited_refs — Set of visited refs (for loop detection)

Output:
  resolved_schema — Resolved Schema object

Preconditions:
  - ref_string is not empty

Steps:
  1. If ref_string ∈ visited_refs → Throw SCHEMA_CIRCULAR_REF error
  2. visited_refs ← visited_refs ∪ {ref_string}
  3. Parse ref_string into (file_part, json_pointer):
     a. If starts with "#" → file_part = current_file, json_pointer = ref_string[1:]
     b. If contains "#" → Split by "#" into file_part and json_pointer
     c. If starts with "apcore://" → Convert to file path under schemas_dir
     d. Otherwise → file_part is path relative to current_file directory
  4. schema_doc ← Load and parse YAML/JSON file for file_part
  5. resolved ← Locate target node in schema_doc using json_pointer
  6. If resolved still contains $ref → Recursively call resolve_ref(...)
  7. Return resolved

Complexity: O(d), where d is reference depth (implementations SHOULD limit max depth to 32)
```

Supports cross-file references for Schema reuse:

```yaml
schema_reference:
  # Reference formats
  formats:
    # Local reference (same file)
    local: "#/definitions/ErrorDetail"

    # Cross-file reference (relative path)
    relative: "./common/error.schema.yaml#/definitions/ErrorDetail"

    # Cross-file reference (Canonical ID)
    canonical: "apcore://common.types.error/ErrorDetail"

  # Reference resolution rules
  resolution:
    - "Local references look up in current file first"
    - "Relative paths are relative to current schema file directory"
    - "Canonical ID references look in schemas/ directory"

  # Example
  example:
    # schemas/executor/validator/db_params.schema.yaml
    input_schema:
      type: object
      properties:
        table:
          type: string
        options:
          $ref: "./common/db_options.schema.yaml#/definitions/DBOptions"

    # Common definitions
    definitions:
      ErrorDetail:
        type: object
        properties:
          field: { type: string }
          code: { type: string }
          message: { type: string }
```

### 4.12 Schema Version Evolution

```yaml
schema_evolution:
  # Compatibility rules
  backward_compatible:
    allowed:
      - "Add optional field (with default value)"
      - "Relax field constraints (e.g., reduce minLength)"
      - "Add enum options"
    forbidden:
      - "Remove required field"
      - "Change field type"
      - "Tighten field constraints"
      - "Remove enum options"

  # Version declaration
  versioning:
    in_schema: true
    format: "semver"
    example:
      version: "1.1.0"
      previous_version: "1.0.0"
      changes:
        - type: "added"
          field: "options.retry_count"
          description: "Added retry count configuration"

  # Deprecated field marking
  deprecation:
    marker: "x-deprecated"
    format:
      x-deprecated:
        since: "1.1.0"
        removal: "2.0.0"
        replacement: "options.max_retries"
        message: "Please use options.max_retries instead"
```

### 4.13 Annotation Conflict Rules

When both YAML metadata file (`*_meta.yaml`) and code define Annotations, conflicts **must** be resolved by the following priority:

1. **YAML metadata file** (highest priority) — Operations teams can override behavior annotations without modifying code
2. **Explicit definition in code** (secondary priority) — Developer defines on module class
3. **Default values** (lowest priority) — Default values provided by framework

Implementations **must** merge rather than replace when loading: If YAML only defines `readonly: true`, other fields **must** retain values from code or defaults.

### 4.14 Schema Validation Error Format

When Schema validation fails, implementations **must** return structured error information:

```yaml
schema_validation_error:
  type: object
  required: [code, message, errors]
  properties:
    code:
      type: string
      const: "SCHEMA_VALIDATION_ERROR"
    message:
      type: string
      description: "Human-readable error summary"
    errors:
      type: array
      items:
        type: object
        required: [path, message]
        properties:
          path:
            type: string
            description: "JSON Pointer to error field (e.g., '/table' or '/options/retry_count')"
          message:
            type: string
            description: "Detailed description of validation failure"
          constraint:
            type: string
            description: "Name of violated constraint (e.g., 'pattern', 'minimum', 'required')"
          expected:
            description: "Expected value or constraint"
          actual:
            description: "Actual value"
```

### 4.15 Edge Case Handling

Implementations **must** handle Schema edge cases according to the following table:

| Scenario | Behavior | Level |
|------|------|------|
| `$ref` depth exceeds `schema.max_ref_depth` | Throw `SCHEMA_CIRCULAR_REF` | **MUST** |
| `$ref` target path doesn't exist (404) | Throw `SCHEMA_NOT_FOUND` | **MUST** |
| Empty Schema `{}` | Treat as `type: object`, allow any properties | **MUST** |
| YAML/JSON syntax error | Throw `SCHEMA_PARSE_ERROR` | **MUST** |
| Unknown JSON Schema keyword (e.g., `x-custom`) | Ignore (forward compatible) | **MUST** |
| Circular reference detection (A → B → A) | Throw `SCHEMA_CIRCULAR_REF` | **MUST** |
| `required` field is empty array `[]` | Treat as no required fields | **MUST** |
| `enum` value contains `null` | Allow, `null` is valid enum value | **MUST** |
| Cross-file `$ref` loading timeout | Throw `SCHEMA_PARSE_ERROR` (cause: timeout) | **SHOULD** |

**Example — Circular Reference Detection:**

```yaml
# schemas/user.schema.yaml
$ref: "./team.schema.yaml#/definitions/team"

# schemas/team.schema.yaml
definitions:
  team:
    $ref: "./user.schema.yaml"  # Circular reference
```

**Behavior**: Maintain reference path stack in `resolve_ref()`, throw `SCHEMA_CIRCULAR_REF` when duplicate path is detected.

### 4.16 Strict Mode Export

OpenAI and Anthropic's `strict: true` mode requires JSON Schema to satisfy additional constraints. apcore defines `to_strict_schema()` conversion to transform standard apcore Schema to Strict Mode compatible format.

**Strict Mode Requirements:**

| Requirement | Description |
|------|------|
| `additionalProperties: false` | All nested `object` types **must** set this |
| All fields `required` | All fields in `properties` **must** appear in `required` array |
| Optional fields expressed with nullable | Originally optional fields become `required` + `type: ["original_type", "null"]` |
| No `x-*` extension fields | All `x-*` fields **must** be stripped |
| No `default` values | `default` fields **must** be removed |

**`to_strict_schema()` Conversion Rules:**

```
Input: apcore_schema (standard JSON Schema + x-* extensions)
Output: strict_schema (Strict Mode compatible JSON Schema)

Rules:
  1. Recursively traverse all type: "object" nodes:
     a. Set additionalProperties: false
     b. Add properties not in required to required
     c. For newly added required fields, change their type to [original_type, "null"]
  2. Remove all fields starting with "x-" (x-llm-description, x-examples, x-sensitive, x-constraints, etc.)
  3. Remove all default fields
  4. Recursively process nested objects (including array items, oneOf/anyOf/allOf sub-schemas)
```

**Example — Before/After Conversion:**

```yaml
# Before conversion (apcore standard Schema)
type: object
properties:
  to:
    type: string
    description: "Recipient email"
    x-examples: ["user@example.com"]
  cc:
    type: array
    items: { type: string }
    description: "CC list"
    default: []
required: [to]

# After conversion (Strict Mode)
type: object
properties:
  to:
    type: string
    description: "Recipient email"
  cc:
    type: ["array", "null"]
    items: { type: string }
    description: "CC list"
required: [to, cc]
additionalProperties: false
```

**Registry `export_schema()` Integration:**

`export_schema()` accepts optional `strict` parameter. When `strict=true`, automatically applies `to_strict_schema()` conversion before export.

```python
# Standard export
schema = registry.export_schema("executor.email.send_email")

# Strict Mode export (for OpenAI / Anthropic)
strict_schema = registry.export_schema("executor.email.send_email", strict=True)
```

For detailed algorithm pseudocode, see [algorithms.md A23](docs/spec/algorithms.md#a23-to_strict_schema--strict-mode-conversion).

### 4.17 Export Profiles

apcore defines standard export Profiles for adapter developers to follow. Profiles are **adapter layer configuration**, not core implementation—apcore framework itself doesn't implement adapters but provides unified conversion specifications.

| Profile | Characteristics | Typical Users |
|---------|------|-----------|
| `mcp` | Preserve `x-*` fields; map annotations → hints; contains inputSchema + outputSchema | MCP Server adapters |
| `openai` | Strip `x-*`; `strict: true`; replace `.` with `_` in id; parameters only | OpenAI Function Calling adapters |
| `anthropic` | Strip `x-*`; map examples → input_examples; contains input_schema | Anthropic Claude Tool adapters |
| `generic` | Full JSON Schema + all extensions (default) | Generic scenarios, debugging |

**Conversion Details for Each Profile:**

**`mcp` Profile:**
- Schema: Preserve as-is (with `x-*` extension fields)
- ID: Use as-is (`executor.email.send_email`)
- Annotations: `readonly` → `readOnlyHint`, `destructive` → `destructiveHint`, `idempotent` → `idempotentHint`, `open_world` → `openWorldHint`
- See Appendix D.1 MCP Mapping

**`openai` Profile:**
- Schema: Apply `to_strict_schema()` conversion (§4.16)
- ID: Replace `.` with `_` (e.g., `executor_email_send_email`)
- Replace `description` with `x-llm-description` then strip
- See Appendix D.3 OpenAI Mapping

**`anthropic` Profile:**
- Schema: Strip `x-*` fields
- ID: Replace `.` with `_`
- Replace `description` with `x-llm-description` then strip
- Examples: `module.examples[*].inputs` → `input_examples`
- See Appendix D.4 Anthropic Mapping

**`generic` Profile:**
- Schema: Full output, no conversions
- For apcore internal use, debugging, documentation generation

---

## 5. Module Specification (Module Specification)

### 5.1 Module File Structure

Each module consists of the following parts:

```
extensions/{layer}/{type}/{module_name}.{ext}      # Module implementation
extensions/{layer}/{type}/{module_name}_meta.yaml  # Module metadata (optional)
schemas/{canonical_id}.schema.yaml              # Schema definition
```

### 5.2 Metadata File and Entry Point Resolution

Implementations **must** resolve module entry points according to the following algorithm:

```
Algorithm: resolve_entry_point(meta_yaml, file_path, language)

Input:
  meta_yaml  — Metadata file content (may be null)
  file_path  — Module file path
  language   — File language (determined by extension)

Output:
  entry_point — { file: string, class_name: string }

Steps:
  1. If meta_yaml exists and contains entry_point field:
     a. Parse format "filename:ClassName"
     b. Return { file: filename, class_name: ClassName }
  2. Otherwise, auto-infer:
     a. file ← filename from file_path (without extension)
     b. class_name ← Convert file from snake_case to PascalCase
     c. If language == "python": Look for class inheriting Module in file
     d. If unique match found → Return that class
     e. If multiple matches found → Throw AMBIGUOUS_ENTRY_POINT error
     f. If no match found → Throw NO_MODULE_CLASS error
  3. Return entry_point
```

```yaml
# extensions/executor/validator/db_params_meta.yaml

# Note: module_id and group are auto-generated from directory, no manual specification needed

# Module description
description: "Database parameter validator"

# Entry point (optional, defaults to inference from filename)
entry_point: "db_params:DbParamsValidator"

# Allowed callers to this module (ACL)
allowed_callers:
  - "orchestrator.engine.*"        # Wildcard
  - "api.handler.task_submit"      # Exact match

# Dependencies on other modules (see 5.3 Dependency Management)
dependencies:
  - module_id: "common.util.sql_parser"
    version: ">=1.0.0"             # Version constraint (optional)
    optional: false                 # Whether optional dependency

# Tags (for categorization and search)
tags:
  - database
  - validation
  - security

# Version (optional)
version: "1.0.0"

# Behavior annotations (optional, help AI make invocation decisions)
annotations:
  readonly: false
  destructive: false
  idempotent: true
  requires_approval: false
  open_world: false

# Usage examples (optional, help AI understand complex modules)
examples:
  - title: "Validate SELECT statement"
    inputs:
      table: "user_info"
      sql: "SELECT * FROM user_info WHERE id = 1"
    output:
      valid: true
      message: "Validation passed"

  - title: "Detect dangerous SQL"
    inputs:
      table: "user_info"
      sql: "DROP TABLE user_info"
    output:
      valid: false
      errors:
        - field: "sql"
          code: "DANGEROUS_SQL"
          message: "SQL contains dangerous keyword: DROP"

# Extension metadata (optional, free dict)
metadata:
  owner: "database-team"
  avg_latency_ms: 5

# Deprecation marking (optional)
deprecated: false
deprecated_message: null
replacement: null

# Resource limits (optional, for sandbox isolation)
resources:
  timeout: 30000                   # Execution timeout (ms)
  memory_limit: null               # Memory limit (bytes), null=no limit
```

### 5.3 Dependency Management

Implementations **must** use topological sorting to resolve dependency order and **must** detect circular dependencies.

```
Algorithm: resolve_dependencies(modules)

Input:
  modules — Module set, each module contains { id, dependencies[] }

Output:
  load_order — List of module IDs sorted by dependency order

Steps:
  1. Build dependency graph: Map<module_id, Set<dependency_id>>
  2. Calculate in-degree: Map<module_id, int>
  3. queue ← All modules with in-degree 0
  4. load_order ← []
  5. While queue is not empty:
     a. current ← queue.dequeue()
     b. load_order.append(current)
     c. For each dependent of current:
        - in_degree[dependent] -= 1
        - If in_degree[dependent] == 0 → queue.enqueue(dependent)
  6. If len(load_order) < len(modules):
     - remaining ← Modules not added to load_order
     - Throw CIRCULAR_DEPENDENCY error with circular path
  7. Return load_order

Complexity: O(V + E), where V is number of modules, E is number of dependencies
```

```yaml
dependency_management:
  # Dependency declaration format
  declaration:
    simple: "common.util.sql_parser"           # Simple format: any version
    with_version:
      module_id: "common.util.sql_parser"
      version: ">=1.0.0,<2.0.0"                # Version constraint
      optional: false                           # Required dependency

  # Version constraint syntax (similar to npm/pip)
  version_constraints:
    - "1.0.0"        # Exact version
    - ">=1.0.0"      # Greater than or equal
    - "<2.0.0"       # Less than
    - ">=1.0.0,<2.0.0"  # Range
    - "^1.0.0"       # Compatible version (1.x.x)
    - "~1.0.0"       # Approximate version (1.0.x)

  # Dependency resolution rules
  resolution:
    strategy: "highest"            # highest | lowest | locked
    conflict_resolution: "error"   # error | use_highest | use_lowest

  # Circular dependencies
  circular_dependency: "error"     # Error when circular dependency detected

  # Optional dependencies
  optional_dependency:
    behavior: "skip_if_missing"    # Skip if missing, don't error
    check_method: "has_module(module_id) -> bool"
```

### 5.4 Multi-version Coexistence

```yaml
multi_version:
  # Whether to allow multiple versions of same module
  enabled: true

  # Version identification method
  identification:
    method: "file_suffix"          # Filename suffix
    pattern: "{name}_v{major}"     # e.g., db_params_v2.py

  # Version selection
  selection:
    default: "latest"              # Use latest version by default
    explicit: "module_id@version"  # Explicit specification: executor.validator.db_params@2

  # Examples
  examples:
    - file: "db_params.py"
      id: "executor.validator.db_params"
      version: "1.0.0"

    - file: "db_params_v2.py"
      id: "executor.validator.db_params_v2"
      version: "2.0.0"
      alias: "executor.validator.db_params@2"
```

### 5.5 Module Isolation (Optional)

```yaml
# [Implementation Phase: Phase 2]
module_isolation:
  # Isolation levels
  levels:
    none: "No isolation, shared process space"
    process: "Separate process"
    container: "Container isolation"

  # Resource limits
  resource_limits:
    timeout:
      type: integer
      unit: "ms"
      default: 30000
    memory:
      type: integer
      unit: "bytes"
      default: null              # null = no limit
    cpu:
      type: number
      unit: "cores"
      default: null

  # Sandbox configuration
  sandbox:
    network: true                # Allow network access
    filesystem: "readonly"       # readonly | readwrite | none
    allowed_paths: []            # Allowed access paths
```

### 5.6 Module Interface Protocol

All modules **must** implement the following interface. Using language-agnostic pseudocode signature definitions:

```
Interface: Module

  Required implementations:
    execute(inputs: Map<String, Any>, context: Context) → Map<String, Any>
      Precondition: inputs has passed input_schema validation
      Postcondition: Return value must conform to output_schema
      Exception: ModuleError

  Optional implementations:
    execute_async(inputs: Map<String, Any>, context: Context) → Future<Map<String, Any>>
    validate(inputs: Map<String, Any>) → ValidationResult
    on_load() → void
    on_unload() → void

  Required definitions:
    input_schema: SchemaDefinition    // Input Schema
    output_schema: SchemaDefinition   // Output Schema
    description: String               // Module description

  Optional definitions:
    name: String?                     // Human-readable name
    tags: List<String>                // Tag list
    version: String                   // Semantic version
    annotations: ModuleAnnotations?   // Behavior annotations
    examples: List<ModuleExample>     // Usage examples
    metadata: Map<String, Any>        // Extension metadata
```

**Cross-language Implementation Mapping:**

| Pseudocode Interface | Python | Rust | Go | Java | TypeScript |
|-----------|--------|------|----|------|------------|
| `Module` base class | `class Module(ABC)` | `trait Module` | `type Module interface` | `interface Module` | `abstract class Module` |
| `execute()` | `def execute(self, inputs, context)` | `fn execute(&self, inputs, context)` | `func (m) Execute(inputs, ctx)` | `Map execute(Map, Context)` | `execute(inputs, context)` |
| `input_schema` | `ClassVar[Type[BaseModel]]` | `type InputSchema: Serialize` | `InputSchema struct` | `Class<? extends Schema>` | `static inputSchema: ZodSchema` |
| `Map<String, Any>` | `dict[str, Any]` | `HashMap<String, Value>` | `map[string]any` | `Map<String, Object>` | `Record<string, unknown>` |

All modules **must** implement the following interface:

```yaml
module_interface:
  # Required methods
  required_methods:
    - name: "execute"
      description: "Execute module main logic"
      input: "Defined by input_schema"
      output: "Defined by output_schema"
      async_variant: "execute_async"  # Async version

  # Optional attributes
  optional_attributes:
    - name: "name"
      type: "string"
      description: "Human-readable module name"
      default: "Auto-generated from class name"

    - name: "tags"
      type: "list[string]"
      description: "Tags for categorization and search"
      default: "[]"

    - name: "version"
      type: "string"
      description: "Module version"
      default: "1.0.0"

    - name: "annotations"
      type: "ModuleAnnotations"
      description: "Behavior annotations, help AI make invocation decisions"
      default: "Default values (readonly=false, destructive=false, ...)"

    - name: "examples"
      type: "list[ModuleExample]"
      description: "Usage examples, help AI understand complex modules"
      default: "[]"

    - name: "metadata"
      type: "dict[str, Any]"
      description: "Free extension metadata"
      default: "{}"

  # Optional implementations
  optional_methods:
    - name: "validate"
      description: "Validate input only, don't execute"
      input: "Defined by input_schema"
      output: "{ valid: bool, errors: array }"

    - name: "describe"
      description: "Return module description (for LLM)"
      input: "None"
      output: "{ description: string, input_schema: object, output_schema: object, annotations: object, examples: array }"

  # Lifecycle hooks
  lifecycle_hooks:
    - name: "on_load"
      description: "Called when module loads"
    - name: "on_unload"
      description: "Called when module unloads"
```

### 5.7 Context Parameter Specification

Each module invocation passes a `context` parameter containing runtime context information. In cross-process scenarios, Context **must** support serialization for transport.

**Design Principle**: Only fields that the framework execution engine depends on are independent fields, everything else goes in `data` (referencing Go `context.Context`, OpenTelemetry Context design philosophy).

```yaml
context_schema:
  type: object
  properties:
    # ====== Framework engine dependencies (breaks if removed) ======

    trace_id:
      type: string
      format: uuid
      description: "Request trace ID (unique across full chain)"
      required: true

    caller_id:
      type: string
      nullable: true
      description: "Caller's Canonical ID (null for top-level calls)"
      example: "orchestrator.engine.task_flow"

    call_chain:
      type: array
      items:
        type: string
      description: "Complete call chain (for loop detection, depth limiting, ACL)"
      example: ["api.handler.task_submit", "orchestrator.engine.task_flow"]

    executor:
      description: "Executor reference (for calling other modules, runtime injected)"

    # ====== Almost all users need, semantically stable ======

    identity:
      type: object
      nullable: true
      description: "Caller identity (ACL engine depends on)"
      properties:
        id:
          type: string
          description: "Unique identifier"
        type:
          type: string
          enum: [user, service, agent, api_key, system]
          default: "user"
          description: "Identity type"
        roles:
          type: array
          items:
            type: string
          description: "Role list (ACL depends on)"
        attrs:
          type: object
          description: "Extension attributes (tenant_id, email, etc. business fields)"

    # ====== Everything else ======

    data:
      type: object
      description: "Shared pipeline state (reference passing, readable/writable along call chain)"
```

**Field Classification Rationale:**

| Field | Classification | Reason |
|------|------|------|
| `trace_id` | Framework engine dependency | Executor generates, middleware/logging depends on across chain |
| `caller_id` | Framework engine dependency | ACL engine determines "who is calling whom" |
| `call_chain` | Framework engine dependency | Loop detection, depth limiting, ACL path matching |
| `executor` | Framework engine dependency | Only channel for inter-module calls |
| `identity` | Widely needed | ACL is framework first-class citizen, needs standardized "who" |
| `data` | Universal bag | span_id, locale, pipeline intermediate state, etc. all mutable data |

**Context Serialization Specification (Cross-process Scenarios):**

In cross-process/cross-network invocation scenarios, Context **must** be serializable to JSON format:

```yaml
context_serialization:
  format: "JSON"
  rules:
    - "trace_id: MUST serialize as string"
    - "caller_id: MUST serialize as string | null"
    - "call_chain: MUST serialize as string[]"
    - "executor: MUST NOT serialize (runtime injected)"
    - "identity: MUST serialize as JSON object"
    - "data: SHOULD serialize, but MUST exclude non-serializable values"
    - "When data contains functions/connections etc. non-serializable values, MUST silently skip and log warning"
```

### 5.8 Async Module Specification

Async module state transitions **must** follow this state machine:

```
                    ┌──────────────────────────────────────────┐
                    │                                          │
                    ▼                                          │
  ┌──────┐    ┌─────────┐    ┌─────────┐    ┌───────────┐    │
  │ idle │───▶│ pending │───▶│ running │───▶│ completed │    │
  └──────┘    └─────────┘    └────┬────┘    └───────────┘    │
                                  │                            │
                                  ├───▶ ┌────────┐             │
                                  │     │ failed │             │
                                  │     └────────┘             │
                                  │                            │
                                  └───▶ ┌───────────┐          │
                                        │ cancelled │──────────┘
                                        └───────────┘
                                        (can resubmit)

  State transition rules:
    idle → pending       : When execute_async() is called
    pending → running    : When executor starts processing
    running → completed  : When execution succeeds
    running → failed     : When execution throws exception
    running → cancelled  : When cancel() is called
    cancelled → pending  : When resubmitted (MAY support)
    failed → pending     : When retrying (MAY support)

  Forbidden transitions:
    completed → *        : Completed tasks MUST NOT transition to other states
    idle → running       : MUST NOT skip pending state
```

For long-running modules, async execution mode is supported:

```yaml
async_module:
  # Async execution methods
  methods:
    # Start async task
    execute_async:
      input: "Defined by input_schema"
      output:
        type: object
        properties:
          task_id:
            type: string
            description: "Async task ID"
          status:
            type: string
            enum: [pending, running]

    # Query task status
    get_status:
      input:
        task_id: string
      output:
        type: object
        properties:
          status:
            type: string
            enum: [pending, running, completed, failed, cancelled]
          progress:
            type: number
            minimum: 0
            maximum: 100
          result:
            description: "Result when task completes (per output_schema)"
          error:
            description: "Error info when task fails"

    # Cancel task
    cancel:
      input:
        task_id: string
      output:
        type: object
        properties:
          success: boolean

  # Callback mechanism (optional)
  callbacks:
    on_progress:
      description: "Progress update callback"
    on_complete:
      description: "Task completion callback"
    on_error:
      description: "Task failure callback"
```

### 5.9 Inter-module Communication

Modules call other modules through `Executor`:

```yaml
module_communication:
  # Invocation methods
  methods:
    # Sync call
    sync_call:
      signature: "executor.call(module_id, inputs, context)"
      returns: "output or raises ModuleError"

    # Async call
    async_call:
      signature: "executor.call_async(module_id, inputs, context)"
      returns: "Future[output]"

  # Invocation constraints
  constraints:
    - "Must go through Executor, direct instantiation not allowed"
    - "Invocations subject to ACL rules"
    - "Call chain automatically recorded to context.call_chain"
    - "Avoid circular calls (framework detects and errors)"

  # Example (Python)
  example: |
    class MyModule(Module):
        def execute(self, inputs: dict, context: Context) -> dict:
            # Call other module
            result = context.executor.call(
                module_id="common.util.sql_parser",
                inputs={"sql": inputs["sql"]},
                context=context
            )
            return {"parsed": result}
```

### 5.10 Module Implementation Example (Python)

```python
# extensions/executor/validator/db_params.py

from apcore import Module
from pydantic import BaseModel, Field
from typing import Optional

class DBParamsInput(BaseModel):
    """Input Schema - Keep consistent with YAML"""
    table: str = Field(..., pattern=r"^[a-z][a-z0-9_]*$")
    sql: str
    timeout: int = Field(default=30, ge=1, le=300)

class DBParamsOutput(BaseModel):
    """Output Schema"""
    valid: bool
    message: Optional[str] = None
    errors: list[dict] = []
    warnings: list[str] = []

class DbParamsValidator(Module):
    """Database parameter validator"""

    # Schema declaration (optional if using YAML)
    input_schema = DBParamsInput
    output_schema = DBParamsOutput

    # Behavior annotations: AI knows this is readonly check, idempotent, no side effects
    annotations = ModuleAnnotations(
        readonly=True,
        destructive=False,
        idempotent=True,
        requires_approval=False,
        open_world=False
    )

    # Usage examples
    examples = [
        ModuleExample(
            title="Validate safe SQL",
            inputs={"table": "user_info", "sql": "SELECT * FROM user_info"},
            output={"valid": True, "message": "Validation passed", "errors": [], "warnings": []}
        )
    ]

    def execute(self, inputs: dict, context: Context) -> dict:
        """Execute validation"""
        # Input already validated by Schema
        params = DBParamsInput(**inputs)

        errors = []
        warnings = []

        # Business logic: Check dangerous SQL
        dangerous_keywords = ['DROP', 'TRUNCATE', 'DELETE']
        sql_upper = params.sql.upper()
        for kw in dangerous_keywords:
            if kw in sql_upper:
                errors.append({
                    "field": "sql",
                    "code": "DANGEROUS_SQL",
                    "message": f"SQL contains dangerous keyword: {kw}"
                })

        return DBParamsOutput(
            valid=len(errors) == 0,
            message="Validation passed" if not errors else "Validation failed",
            errors=errors,
            warnings=warnings
        ).model_dump()
```

### 5.11 Function-based Module Definition (Function-based Module Definition)

> Section name uses "Function-based Module Definition" rather than "Decorator Module Definition" because Decorator is language-specific syntax, while the core requirement is **semantic capability**—wrapping existing callables as standard modules.

#### 5.11.1 Normative Statement

Implementations **MUST** provide `module()` mechanism to wrap existing callables (functions or methods) as standard modules, auto-generating Schema from type information.

`module()` is the only core concept, available in both Decorator and function call forms:

- In languages supporting Decorator syntax (Python, TypeScript, Java annotations), **SHOULD** provide as Decorator form
- In languages not supporting Decorator (Go, C), **MUST** provide as function call form
- Both forms share the same parameter set, producing modules that **must** be completely equivalent in Registry/Executor/Schema behavior

#### 5.11.2 Cross-language Syntax Reference

| Language | Decorator Form | Function Call Form (Register Existing Code) |
|------|---------------|--------------------------|
| Python | `@module(id="email.send") def send(...)` | `module(service.send, id="email.send")` |
| TypeScript | `@module({id: "email.send"}) function send(...)` | `module(service.send, {id: "email.send"})` |
| Java | `@Module(id="email.send") public Map send(...)` | `Apcore.module(service::send, "email.send")` |
| Rust | `#[module(id = "email.send")] fn send(...)` | `apcore::module("email.send", \|inputs\| { ... })` |
| Go | — None | `apcore.Module("email.send", sendEmail)` / `apcore.Module("email.send", service.Send)` |
| C | — None | `apcore_module("email.send", &send_email)` |

#### 5.11.3 `module()` Parameter Signature

Both forms share the same parameter set (language-agnostic pseudocode):

```
module(callable?, options?) → Module

options:
  id: String?              // Module ID (optional, auto-generate if absent)
  description: String?     // Module description (optional, extract from docstring/comment if absent)
  annotations: Annotations? // Behavior annotations (optional)
  tags: List<String>?      // Tags (optional)
  version: String?         // Version (optional, default "1.0.0")
  metadata: Map?           // Extension metadata (optional)
```

- Decorator form: `callable` implicitly provided by decorator target, `options` as decorator parameters
- Function call form: `callable` explicitly passed (first parameter), `options` as subsequent parameters

#### 5.11.4 Type Inference and Schema Auto-generation

Implementations **must** auto-generate JSON Schema from function signatures according to the following algorithm:

```
Algorithm: generate_schema_from_function(callable)

Input:
  callable — Target function or method

Output:
  schema — { input_schema: JSONSchema, output_schema: JSONSchema, description: String }

Steps:
  1. Extract function parameter list params (exclude self/cls and context: Context)
  2. For each param:
     a. Get type annotation type_hint
     b. If type_hint missing → Throw FUNC_MISSING_TYPE_HINT error
     c. Map type_hint to JSON Schema type (see §5.11.5)
     d. Extract constraints from Annotated metadata, default values, etc.
     e. Extract description from docstring parameter comments
  3. Construct input_schema:
     a. type: "object"
     b. properties: Schema mapping of all parameters
     c. required: List of parameters without defaults
  4. Extract return type annotation return_type
     a. If return_type missing → Throw FUNC_MISSING_RETURN_TYPE error
     b. Map return_type to output_schema
  5. Extract description:
     a. docstring/comment first line → description
     b. Parameter comments → Each field's description
  6. Return { input_schema, output_schema, description }

Complexity: O(n), where n is number of parameters
```

#### 5.11.5 Language Type → JSON Schema Mapping Table

| Language Type | JSON Schema | Description |
|---------|-------------|------|
| `str` / `String` | `{ "type": "string" }` | String |
| `int` / `Integer` / `i32` | `{ "type": "integer" }` | Integer |
| `float` / `Double` / `f64` | `{ "type": "number" }` | Float |
| `bool` / `Boolean` | `{ "type": "boolean" }` | Boolean |
| `list[T]` / `Vec<T>` / `[]T` | `{ "type": "array", "items": <T> }` | Array |
| `dict[str, T]` / `Map<String, T>` | `{ "type": "object", "additionalProperties": <T> }` | Map |
| `Optional[T]` / `T \| None` | `<T>` + `"nullable": true` | Nullable type |
| `BaseModel` / `struct` / `@dataclass` | `{ "type": "object", "properties": {...} }` | Struct/object |
| `Literal["a", "b"]` / `enum` | `{ "type": "string", "enum": ["a", "b"] }` | Enum |
| `Annotated[T, Field(...)]` | `<T>` + constraint fields | Constrained type |

#### 5.11.6 Module ID Generation Rules

When `module()` doesn't specify `id` parameter, **must** auto-generate from function full path:

```
Rule: generate_module_id(callable)

Steps:
  1. Get callable's module path (e.g., "myapp.services.email")
  2. Get callable's name (e.g., "send_email")
  3. Combine as "{module_path}.{name}"
  4. Normalize to Canonical ID format (§2.7)
  5. If ID already exists → Throw duplicate_id error (§2.6)
```

#### 5.11.7 Description Extraction Rules

Implementations **must** extract module description by the following priority:

1. `module()`'s `description` parameter (highest priority)
2. Function docstring / comment first line
3. If neither above → Generate default description from function name

Parameter-level description:
- Extract from docstring's Args/Parameters section
- Extract from `Annotated[T, Field(description="...")]`
- Extract from language comments

#### 5.11.8 Sync/Async Function Support

| Function Type | Mapping |
|---------|------|
| `def func(...)` | `execute()` |
| `async def func(...)` | `execute_async()` |

Implementations **should** support both sync and async functions. Async functions **should** map to module's `execute_async()` method.

#### 5.11.9 Context Injection

When function parameters include `context: Context` type annotation, framework **must** auto-inject Context object, and this parameter **must not** appear in generated `input_schema`.

```python
# context parameter auto-injected, doesn't appear in Schema
@module(id="email.send")
def send_email(to: str, subject: str, body: str, context: Context) -> dict:
    print(f"trace_id: {context.trace_id}")
    return {"success": True}

# Generated input_schema only contains to, subject, body
```

#### 5.11.10 Equivalence Guarantee

Function-defined modules and class-defined modules **must** be completely equivalent in:

- Registration and discovery behavior in Registry
- Executor's invocation flow (Schema validation → ACL → Middleware → Execution)
- Schema format and validation logic
- Error handling and error codes
- Observability (tracing, logging, metrics)

#### 5.11.11 Error Codes

| Error Code | Description | Trigger Condition |
|--------|------|---------|
| `FUNC_MISSING_TYPE_HINT` | Function parameter missing type annotation | Parameter has no type annotation and can't be inferred |
| `FUNC_MISSING_RETURN_TYPE` | Function missing return type annotation | Return type has no annotation and can't be inferred |

#### 5.11.12 Examples

**Python — Decorator Form:**

```python
from apcore import module, Context
from typing import Annotated
from pydantic import Field

@module(id="email.send", tags=["email", "notification"])
def send_email(
    to: Annotated[str, Field(description="Recipient email")],
    subject: Annotated[str, Field(description="Email subject", max_length=200)],
    body: Annotated[str, Field(description="Email body")],
    context: Context
) -> dict:
    """Email sending module"""
    # Original business logic
    return {"success": True, "message_id": "msg_123"}
```

**Python — Function Call Form (Register Existing Code):**

```python
from apcore import module

class EmailService:
    def send(self, to: str, subject: str, body: str) -> dict:
        """Send email"""
        return {"success": True}

service = EmailService()
module(service.send, id="email.send")
```

**Go — Function Call Form:**

```go
package main

import "github.com/apcore/apcore-go"

func sendEmail(to string, subject string, body string) map[string]any {
    return map[string]any{"success": true}
}

func main() {
    apcore.Module("email.send", sendEmail)

    // Register existing struct method
    service := &EmailService{}
    apcore.Module("email.send_template", service.SendTemplate)
}
```

---

### 5.12 External Schema Binding (External Schema Binding)

#### 5.12.1 Normative Statement

Implementations **MUST** support external Schema binding files, allowing existing functions to be mapped as apcore modules through YAML configuration, achieving zero-code-modification integration.

#### 5.12.2 Binding File Format

Binding files **must** be in YAML format, containing a `bindings` array:

```yaml
# bindings/email.binding.yaml
bindings:
  - module_id: "email.send"
    target: "myapp.services.email:send_email"
    description: "Send email"
    input_schema:
      type: object
      properties:
        to:
          type: string
          description: "Recipient email"
        subject:
          type: string
          description: "Email subject"
        body:
          type: string
          description: "Email body"
      required: [to, subject, body]
    output_schema:
      type: object
      properties:
        success:
          type: boolean
        message_id:
          type: string

  - module_id: "email.send_template"
    target: "myapp.services.email:EmailService.send_template"
    description: "Send email using template"
    auto_schema: true  # Auto-generate Schema from type annotations
```

**Binding Item Field Definitions:**

| Field | Type | Required | Description |
|------|------|------|------|
| `module_id` | string | **MUST** | Module Canonical ID |
| `target` | string | **MUST** | Target callable (format: `module.path:callable_name`) |
| `description` | string | **SHOULD** | Module description |
| `input_schema` | object | Conditional | Input Schema (choose one with `auto_schema`) |
| `output_schema` | object | Conditional | Output Schema (choose one with `auto_schema`) |
| `auto_schema` | boolean | Conditional | Auto-generate Schema from type annotations (choose one with explicit Schema) |
| `schema_ref` | string | **MAY** | Reference external Schema file path |
| `annotations` | object | **MAY** | Behavior annotations |
| `tags` | array | **MAY** | Tags |
| `version` | string | **MAY** | Version |
| `metadata` | object | **MAY** | Extension metadata |

#### 5.12.3 Target Resolution Algorithm

Implementations **must** resolve `target` field according to the following algorithm:

```
Algorithm: resolve_target(target_string)

Input:
  target_string — Target callable path (e.g., "myapp.services.email:send_email")

Output:
  callable — Callable object

Preconditions:
  - target_string conforms to "module.path:callable_name" format

Steps:
  1. Split target_string by ":" into (module_path, callable_name)
     If no ":" → Throw BINDING_INVALID_TARGET error
  2. import module_path
     If import fails → Throw BINDING_MODULE_NOT_FOUND error
  3. If callable_name contains ".":
     a. Split by "." into (class_name, method_name)
     b. Find class_name in module
     c. Find method_name on class instance
     d. If any step fails → Throw BINDING_CALLABLE_NOT_FOUND error
  4. Otherwise:
     a. Find callable_name in module
     b. If not found → Throw BINDING_CALLABLE_NOT_FOUND error
  5. Validate result is callable
     If not callable → Throw BINDING_NOT_CALLABLE error
  6. Return callable

Complexity: O(1) (not counting module loading time)
```

#### 5.12.4 Schema Reference Support

Binding items **may** reference external Schema files via `schema_ref`, avoiding inlining complete Schema in binding file:

```yaml
bindings:
  - module_id: "email.send"
    target: "myapp.services.email:send_email"
    schema_ref: "../schemas/email.send.schema.yaml"
```

#### 5.12.5 `auto_schema` Mode

When `auto_schema: true`, implementations **must** reuse the `generate_schema_from_function` algorithm from §5.11.4 to auto-generate Schema from target callable's type annotations.

If target callable lacks sufficient type information, **must** throw `BINDING_SCHEMA_MISSING` error.

#### 5.12.6 Discovery Mechanism

Binding file discovery is controlled via `apcore.yaml` configuration:

```yaml
# apcore.yaml
bindings:
  dir: "./bindings"          # Default scan directory
  files:                      # Or specify file list
    - "./bindings/email.binding.yaml"
    - "./bindings/payment.binding.yaml"
  pattern: "*.binding.yaml"  # File matching pattern (default)
```

- If `bindings.dir` is configured, implementations **must** scan files matching `pattern` in that directory
- If `bindings.files` is configured, implementations **must** load specified file list
- If neither configured, implementations **should** default to scanning `bindings/` directory

#### 5.12.7 Validation Rules

Implementations **must** perform the following validations when loading binding files:

1. `module_id` conforms to Canonical ID format (§2.7)
2. `target` can be resolved to valid callable
3. Schema is valid (explicitly defined or auto_schema can generate)
4. `module_id` doesn't conflict with registered modules (§2.6)
5. Binding file itself conforms to `binding.schema.json` (see `schemas/binding.schema.json`)

#### 5.12.8 Error Codes

| Error Code | Description | Trigger Condition |
|--------|------|---------|
| `BINDING_INVALID_TARGET` | target format invalid | target doesn't conform to `module.path:callable_name` format |
| `BINDING_MODULE_NOT_FOUND` | Module path can't be imported | import module_path fails |
| `BINDING_CALLABLE_NOT_FOUND` | Can't find target callable | Can't find specified function/method in module |
| `BINDING_NOT_CALLABLE` | Target not callable | Resolved object is not callable |
| `BINDING_SCHEMA_MISSING` | Schema missing | No explicit Schema and auto_schema can't generate |

### 5.13 Edge Case Handling

Implementations **must** handle module edge cases according to the following table:

#### 5.13.1 execute() Return Value Edges

| Scenario | Behavior | Level |
|------|------|------|
| `execute()` returns `None` | Throw `MODULE_EXECUTE_ERROR` ("Return value cannot be None") | **MUST** |
| `execute()` returns non-Map/dict type | Throw `MODULE_EXECUTE_ERROR` ("Return value must be Map") | **MUST** |
| Return value doesn't match `output_schema` | Throw `SCHEMA_VALIDATION_ERROR` | **MUST** |
| `execute()` throws non-`ModuleError` exception | Wrap as `MODULE_EXECUTE_ERROR` (cause points to original exception) | **MUST** |
| `execute()` returns object with non-serializable objects | **Should** log warning but don't enforce check | **SHOULD** |

#### 5.13.2 Module Dependency Loading Failures

| Scenario | Behavior | Level |
|------|------|------|
| Module in `dependencies.requires` doesn't exist | Throw `DEPENDENCY_NOT_FOUND`, refuse loading | **MUST** |
| Module in `dependencies.optional` doesn't exist | Log INFO, continue loading | **MUST** |
| Module in `dependencies.requires` fails to load | Throw `MODULE_LOAD_ERROR`, refuse loading | **MUST** |
| Module in `dependencies.optional` fails to load | Log WARN, continue loading | **MUST** |
| Reverse dependency (A depends on B, B also depends on A) | Throw `CIRCULAR_DEPENDENCY` | **MUST** |
| Indirect circular dependency (A → B → C → A) | Throw `CIRCULAR_DEPENDENCY` | **MUST** |

#### 5.13.3 Module Lifecycle Edges

| Scenario | Behavior | Level |
|------|------|------|
| `on_load()` hook throws exception | Throw `MODULE_LOAD_ERROR` (cause points to original exception), terminate loading | **MUST** |
| `on_unload()` hook throws exception | Log ERROR, continue unload process | **MUST** |
| Module called before load completion | Wait for load completion or throw `MODULE_NOT_FOUND` | **SHOULD** |
| Module called after unload | Throw `MODULE_NOT_FOUND` | **MUST** |
| Repeated `discover()` of same module | If `metadata.yaml` unchanged, skip (idempotent) | **MUST** |
| Hot reload while module executing | See §11.7.3 Hot Reload Race Conditions | **MUST** |

**Note**:
- Dependency topological sorting uses algorithm A07 (§5.3)
- Circular dependency detection should complete in `discover()` phase, avoiding runtime failures

---

## 6. ACL Specification (ACL Specification)

### 6.1 ACL Files

```yaml
# acl/global_acl.yaml

$schema: "https://apcore.dev/acl/v1"
version: "1.0.0"

# Global rules
rules:
  # Rule 1: API layer can only call orchestration layer
  - id: "api_to_orchestrator"
    callers:
      - "api.*"
    targets:
      - "orchestrator.*"
    actions: [execute]
    effect: allow

  # Rule 2: Orchestration layer can call executor layer
  - id: "orchestrator_to_executor"
    callers:
      - "orchestrator.*"
    targets:
      - "executor.*"
    actions: [execute, validate]
    effect: allow

  # Rule 3: Forbid executor layer calling API layer
  - id: "deny_executor_to_api"
    callers:
      - "executor.*"
    targets:
      - "api.*"
    actions: ["*"]
    effect: deny
    priority: 100  # High priority

# Default policy
default_effect: deny

# Audit configuration
audit:
  enabled: true
  log_level: info
  include_denied: true
```

### 6.2 Rule Matching

Implementations **must** perform pattern matching according to the following algorithm:

```
Algorithm: match_pattern(pattern, module_id)

Input:
  pattern   — ACL pattern (e.g., "api.*", "*.validator.*", "executor.email.send_email")
  module_id — Module Canonical ID to match

Output:
  matched — boolean

Steps:
  1. If pattern == "*" → Return true
  2. If pattern doesn't contain "*":
     → Return pattern == module_id (exact match)
  3. Split pattern by "*" into segments
  4. Use greedy matching algorithm:
     a. pos ← 0
     b. For each segment (non-empty):
        - Find segment in module_id[pos:]
        - If not found → Return false
        - pos ← Found position + len(segment)
     c. If pattern doesn't end with "*" → module_id must end with last segment
  5. Return true

Complexity: O(m × n), where m is pattern length, n is module_id length
```

```yaml
rule_matching:
  # Wildcard support
  wildcards:
    - "*"           # Match any
    - "api.*"       # Match all under api
    - "*.validator.*"  # Match validator in any layer

  # Priority (higher number = higher priority)
  priority:
    default: 0
    explicit_deny: 100

  # Matching order
  order:
    1: "By priority descending"
    2: "deny takes precedence over allow"
    3: "Exact match takes precedence over wildcard"
```

### 6.3 Rule Evaluation Algorithm

Implementations **must** evaluate ACL rules according to the following algorithm:

```
Algorithm: evaluate_acl(caller_id, target_id, rules, default_effect)

Input:
  caller_id      — Caller module ID (null means external call, treated as "@external")
  target_id      — Called module ID
  rules          — Rule list (sorted by priority)
  default_effect — Default policy ("allow" | "deny")

Output:
  decision — { effect: "allow" | "deny", matched_rule: Rule | null }

Steps:
  1. effective_caller ← caller_id ?? "@external"
  2. Sort rules by priority descending (for same priority, maintain definition order)
  3. For same priority rules, deny comes before allow
  4. For each rule ∈ sorted_rules:
     a. caller_matched ← false
        For each pattern ∈ rule.callers:
          If match_pattern(pattern, effective_caller) → caller_matched ← true; break
     b. target_matched ← false
        For each pattern ∈ rule.targets:
          If match_pattern(pattern, target_id) → target_matched ← true; break
     c. If caller_matched and target_matched:
        → Return { effect: rule.effect, matched_rule: rule }
  5. Return { effect: default_effect, matched_rule: null }

Complexity: O(R × P), where R is number of rules, P is average patterns per rule
```

### 6.4 Pattern Specificity Scoring

When further distinguishing rules within same priority is needed, implementations **should** calculate pattern specificity score:

```
Algorithm: calculate_specificity(pattern)

Input:
  pattern — ACL pattern string

Output:
  score — Specificity score (integer, higher = more specific)

Steps:
  1. If pattern == "*" → Return 0
  2. segments ← Split pattern by "."
  3. score ← 0
  4. For each segment:
     a. If segment == "*" → score += 0
     b. If segment contains "*" (partial wildcard) → score += 1
     c. If segment doesn't contain "*" (exact match) → score += 2
  5. Return score
```

### 6.5 Edge Case Handling

| Scenario | Behavior | Level |
|------|------|------|
| `caller_id` is null | Treat as `@external` | **MUST** |
| `rules` is empty | Use `default_effect` | **MUST** |
| `callers` or `targets` in rule is empty array | Rule never matches | **MUST** |
| `actions` field missing | Treat as `["*"]` (match all actions) | **SHOULD** |
| Same rule matches both allow and deny | deny takes precedence | **MUST** |
| Module calls itself | Perform ACL check normally | **MUST** |

---

## 7. Error Handling Specification (Error Handling Specification)

### 7.1 Unified Error Format

All errors must follow unified format:

```yaml
error_format:
  type: object
  required: [code, message]
  properties:
    code:
      type: string
      description: "Error code (format: CATEGORY_SPECIFIC_ERROR)"
    message:
      type: string
      description: "Human-readable error message"
    details:
      type: object
      description: "Error details (optional)"
    cause:
      type: object
      description: "Original error (optional, for error chains)"
    trace_id:
      type: string
      description: "Trace ID"
    timestamp:
      type: string
      format: datetime
```

### 7.2 Framework Error Codes

```yaml
error_codes:
  # Module-related (MODULE_*)
  MODULE_NOT_FOUND:
    description: "Module doesn't exist"
    http_status: 404
  MODULE_LOAD_ERROR:
    description: "Module load failed"
    http_status: 500
  MODULE_EXECUTE_ERROR:
    description: "Module execution error"
    http_status: 500
  MODULE_TIMEOUT:
    description: "Module execution timeout"
    http_status: 504

  # Schema-related (SCHEMA_*)
  SCHEMA_NOT_FOUND:
    description: "Schema doesn't exist"
    http_status: 404
  SCHEMA_VALIDATION_ERROR:
    description: "Schema validation failed"
    http_status: 400
  SCHEMA_PARSE_ERROR:
    description: "Schema parse error"
    http_status: 500

  # Permission-related (ACL_*)
  ACL_DENIED:
    description: "Permission denied"
    http_status: 403
  ACL_RULE_ERROR:
    description: "ACL rule error"
    http_status: 500

  # Function module-related (FUNC_*)
  FUNC_MISSING_TYPE_HINT:
    description: "Function parameter missing type annotation"
    http_status: 500
  FUNC_MISSING_RETURN_TYPE:
    description: "Function missing return type annotation"
    http_status: 500

  # Binding-related (BINDING_*)
  BINDING_INVALID_TARGET:
    description: "Binding target format invalid"
    http_status: 500
  BINDING_MODULE_NOT_FOUND:
    description: "Binding target module path can't be imported"
    http_status: 500
  BINDING_CALLABLE_NOT_FOUND:
    description: "Binding target callable not found"
    http_status: 500
  BINDING_NOT_CALLABLE:
    description: "Binding target not callable"
    http_status: 500
  BINDING_SCHEMA_MISSING:
    description: "Binding Schema missing"
    http_status: 500

  # General errors (GENERAL_*)
  GENERAL_INVALID_INPUT:
    description: "Invalid input"
    http_status: 400
  GENERAL_INTERNAL_ERROR:
    description: "Internal error"
    http_status: 500
  GENERAL_NOT_IMPLEMENTED:
    description: "Feature not implemented"
    http_status: 501

  # Call chain-related (CALL_*)
  CALL_DEPTH_EXCEEDED:
    description: "Call chain depth exceeded limit"
    http_status: 508
  CIRCULAR_CALL:
    description: "Circular call detected"
    http_status: 508
  CALL_FREQUENCY_EXCEEDED:
    description: "Same module call frequency exceeded limit"
    http_status: 508
```

### 7.3 Error Propagation

Implementations **must** propagate errors according to the following algorithm:

```
Algorithm: propagate_error(error, module_id, context)

Input:
  error     — Original exception/error object
  module_id — Module ID where error occurred
  context   — Current execution context

Output:
  module_error — Standardized ModuleError object

Steps:
  1. If error is already ModuleError type:
     a. Keep original error.code
     b. Append current module_id to error.chain
     c. Return error
  2. Construct module_error:
     a. code ← Map based on error type:
        - SchemaValidationError → "SCHEMA_VALIDATION_ERROR"
        - ACLDeniedError → "ACL_DENIED"
        - TimeoutError → "MODULE_TIMEOUT"
        - Other → "MODULE_EXECUTE_ERROR"
     b. message ← Human-readable message from error
     c. details ← Extract structured details from error
     d. cause ← error (keep original error)
     e. trace_id ← context.trace_id
     f. module_id ← module_id
     g. call_chain ← Copy of context.call_chain
     h. timestamp ← Current UTC time (ISO 8601)
  3. Return module_error
```

```yaml
error_propagation:
  # Module errors
  module_error:
    - "Module-thrown errors **must** be wrapped as ModuleError"
    - "Original error **must** be saved in cause field"
    - "Error code prefixed with MODULE_"

  # Error context
  context:
    - "Error **must** contain trace_id"
    - "Error **should** contain occurrence location (module_id)"
    - "**Must** support error chain tracing"
```

### 7.4 Custom Error Codes

Modules can define their own error codes:

```yaml
custom_error_codes:
  # Naming convention
  naming:
    pattern: "{MODULE_PREFIX}_{ERROR_NAME}"
    module_prefix: "Last part of module ID in uppercase"
    examples:
      - module_id: "executor.validator.db_params"
        prefix: "DB_PARAMS"
        error_code: "DB_PARAMS_INVALID_TABLE"
        error_code: "DB_PARAMS_SQL_INJECTION"

  # Declare in Schema
  declaration:
    location: "error_schema.codes"
    example:
      error_schema:
        codes:
          DB_PARAMS_INVALID_TABLE:
            description: "Invalid table name"
            http_status: 400
          DB_PARAMS_SQL_INJECTION:
            description: "SQL injection detected"
            http_status: 400

  # Error code registration
  registration:
    - "Framework startup **must** collect all module error codes"
    - "**Must** detect error code conflicts"
    - "**Should** generate error code documentation"

  # Framework error code priority
  priority:
    - "Framework error codes (MODULE_/SCHEMA_/ACL_/GENERAL_) **must** be reserved, modules **must not** use them"
    - "Module custom error codes **must not** conflict with framework error codes"

  # Collision detection algorithm
  collision_detection: |
    Algorithm: detect_error_code_collisions(framework_codes, module_codes_map)

    Steps:
      1. all_codes ← Copy of framework_codes
      2. For each (module_id, codes) ∈ module_codes_map:
         For each code ∈ codes:
           a. If code ∈ framework_codes → Throw error: Module can't use framework reserved codes
           b. If code ∈ all_codes → Throw error: Error code already registered by other module
           c. all_codes ← all_codes ∪ {code}
      3. Return all_codes (complete error code registry)
```

### 7.5 Error Response Format

```yaml
# Standard error response
error_response:
  code: "DB_PARAMS_INVALID_TABLE"
  message: "Invalid table name format"
  details:
    field: "table"
    value: "User-Info"
    expected: "Only lowercase letters and underscores allowed"
  trace_id: "550e8400-e29b-41d4-a716-446655440000"
  timestamp: "2026-02-05T10:30:00Z"
  cause: null  # Or nested error object
```

### 7.6 Retry Semantics

Implementations **must not** default retry failed module invocations. Retry behavior **must** be explicitly controlled by caller or middleware.

**Retryability Classification:**

| Error Code | Retryable | Description |
|--------|--------|------|
| `MODULE_TIMEOUT` | **Yes** | Timeout may be temporary |
| `GENERAL_INTERNAL_ERROR` | **Yes** | Internal error may be transient |
| `MODULE_EXECUTE_ERROR` | **Depends** | Depends on module's `annotations.idempotent` |
| `SCHEMA_VALIDATION_ERROR` | **No** | Input error won't change with retry |
| `ACL_DENIED` | **No** | Permission insufficiency won't change with retry |
| `MODULE_NOT_FOUND` | **No** | Module non-existence won't change with retry |
| `MODULE_LOAD_ERROR` | **No** | Load errors typically need code fixes |
| `CALL_DEPTH_EXCEEDED` | **No** | Call chain structure issue, retry won't change |
| `CIRCULAR_CALL` | **No** | Call chain structure issue, retry won't change |
| `CALL_FREQUENCY_EXCEEDED` | **No** | Call chain structure issue, retry won't change |

Retry middleware (if implemented) **should**:
- Only retry errors marked as retryable
- Only auto-retry modules with `annotations.idempotent == true`
- Use exponential backoff strategy
- Set max retry count limit (**should** not exceed 5 times)

### 7.7 Error Hierarchy

Framework error codes **must** follow this hierarchy:

```
ApCoreError (root error)
├── ConfigError                    # Configuration-related errors
│   ├── CONFIG_INVALID             # Invalid configuration file
│   └── CONFIG_NOT_FOUND           # Configuration file not found
├── ModuleError                    # Module-related errors
│   ├── MODULE_NOT_FOUND           # Module doesn't exist
│   ├── MODULE_LOAD_ERROR          # Module load failed
│   ├── MODULE_EXECUTE_ERROR       # Module execution error
│   └── MODULE_TIMEOUT             # Module execution timeout
├── SchemaError                    # Schema-related errors
│   ├── SCHEMA_NOT_FOUND           # Schema doesn't exist
│   ├── SCHEMA_VALIDATION_ERROR    # Schema validation failed
│   ├── SCHEMA_PARSE_ERROR         # Schema parse error
│   └── SCHEMA_CIRCULAR_REF        # Schema circular reference
├── ACLError                       # Permission-related errors
│   ├── ACL_DENIED                 # Permission denied
│   └── ACL_RULE_ERROR             # ACL rule error
├── FuncError                      # Function module-related errors
│   ├── FUNC_MISSING_TYPE_HINT     # Function parameter missing type annotation
│   └── FUNC_MISSING_RETURN_TYPE   # Function missing return type annotation
├── BindingError                   # Binding-related errors
│   ├── BINDING_INVALID_TARGET     # target format invalid
│   ├── BINDING_MODULE_NOT_FOUND   # Module path can't be imported
│   ├── BINDING_CALLABLE_NOT_FOUND # Can't find target callable
│   ├── BINDING_NOT_CALLABLE       # Target not callable
│   └── BINDING_SCHEMA_MISSING     # Schema missing
├── DependencyError                # Dependency-related errors
│   ├── CIRCULAR_DEPENDENCY        # Circular dependency
│   └── DEPENDENCY_NOT_FOUND       # Dependent module doesn't exist
├── CallChainError                 # Call chain-related errors
│   ├── CALL_DEPTH_EXCEEDED        # Call depth exceeded limit
│   ├── CIRCULAR_CALL              # Circular call
│   └── CALL_FREQUENCY_EXCEEDED    # Call frequency exceeded limit
└── GeneralError                   # General errors
    ├── GENERAL_INVALID_INPUT      # Invalid input
    ├── GENERAL_INTERNAL_ERROR     # Internal error
    └── GENERAL_NOT_IMPLEMENTED    # Feature not implemented
```

Implementations **must** ensure all framework-thrown errors belong to this hierarchy. Module custom errors **should** inherit from `ModuleError`.

---

## 8. Configuration Specification (Configuration Specification)

### 8.1 Framework Configuration

apcore.yaml is the core configuration file of the framework. Implementations **must** validate configuration files according to the following JSON Schema.

**apcore.yaml Complete JSON Schema Definition:**

```yaml
# apcore.yaml — Complete configuration structure and constraints

$schema: "https://apcore.dev/config/v1"
version: "1.0.0"                    # MUST, configuration version

# Project information
project:
  name: "my-ai-project"             # MUST, project name (pattern: ^[a-z][a-z0-9_-]*$)
  version: "0.1.0"                   # SHOULD, project version (semver)

# Extension configuration
extensions:
  root: "./extensions"               # MUST, extension root directory
  auto_discover: true                # SHOULD, auto-discovery (default: true)
  lazy_load: true                    # MAY, lazy loading (default: true)
  follow_symlinks: false             # MUST NOT default true (default: false)
  max_depth: 8                       # SHOULD, max scan depth (default: 8, max: 16)
  ignore_patterns:                   # MAY, additional ignore patterns
    - "*.test.*"
    - "*.spec.*"

# Schema configuration
schema:
  root: "./schemas"                  # MUST, Schema directory
  strategy: "yaml_first"             # SHOULD, loading strategy (yaml_first|native_first|yaml_only)
  validation:
    strict: true                     # SHOULD, strict mode (default: true)
    coerce_types: true               # MAY, type coercion (default: true)
  max_ref_depth: 32                  # MAY, $ref max recursion depth (default: 32)

# ACL configuration
acl:
  root: "./acl"                      # MUST, ACL configuration directory
  default_effect: "deny"             # MUST, default policy (deny|allow, default: deny)
  audit:
    enabled: true                    # SHOULD, audit logging (default: true)
    log_level: "info"                # MAY, audit log level
    include_denied: true             # SHOULD, log denied calls

# Logging configuration
logging:
  level: "info"                      # SHOULD (trace|debug|info|warn|error|fatal)
  format: "json"                     # SHOULD (json|text, default: json)

# Observability configuration
observability:
  tracing:
    enabled: true                    # SHOULD (default: true)
    sampling_rate: 1.0               # MAY (0.0-1.0, default: 1.0)
    exporter: "stdout"               # MAY (stdout|otlp|jaeger)
  metrics:
    enabled: true                    # MAY (default: true)
    exporter: "stdout"               # MAY (stdout|prometheus|otlp)

# Middleware configuration
middleware:
  disabled: []                       # MAY, list of disabled built-in middleware

# Binding configuration
bindings:
  dir: "./bindings"                  # SHOULD, binding file directory (default: "./bindings")
  files: []                          # MAY, specified binding file list
  pattern: "*.binding.yaml"          # SHOULD, file matching pattern (default: "*.binding.yaml")

# ID Map configuration
id_map:
  auto_detect: true                  # SHOULD (default: true)
  overrides: {}                      # MAY, manual ID mapping overrides
```

#### 8.1.1 Default Values Summary

Implementations **must** follow these default value conventions:

| Configuration Item | Default Value | Valid Range | Description |
|--------|--------|---------|------|
| `extensions.root` | `"./extensions"` | Valid directory path | Extension root directory |
| `extensions.max_depth` | `8` | `1..16` | Scan depth limit |
| `schema.root` | `"./schemas"` | Valid directory path | Schema root directory |
| `schema.max_ref_depth` | `32` | `1..100` | `$ref` resolution depth limit |
| `acl.root` | `"./acl"` | Valid directory path | ACL file root directory |
| `acl.default_effect` | `"deny"` | `allow`/`deny` | Default behavior when no rule matches |
| `executor.timeout` | `60000` (ms) | `0..600000` | Global execution timeout (0 means no limit) |
| `executor.max_call_depth` | `32` | `1..1000` | Call chain max depth |
| `executor.max_module_repeat` | `3` | `1..100` | Max occurrences of same module in call chain |
| `observability.enabled` | `true` | `true`/`false` | Observability master switch |
| `observability.exporter` | `"stdout"` | `stdout`/`prometheus`/`otlp` | Default exporter |
| `bindings.dir` | `"./bindings"` | Valid directory path | Binding file directory |
| `bindings.pattern` | `"*.binding.yaml"` | glob pattern | Binding file matching pattern |
| `id_map.auto_detect` | `true` | `true`/`false` | Auto ID mapping detection |

**Note**:
- Configuration values exceeding ranges **must** be rejected in `validate_config()` (algorithm A12)
- `timeout = 0` means disable timeout, implementations **should** log WARN
- `max_call_depth` and `max_module_repeat` used for call chain safety checks (algorithm A20)

### 8.2 Environment Variable Override

Implementations **must** support overriding configuration file values through environment variables.

**Override Rules:**

| Priority | Source | Example |
|--------|------|------|
| 1 (Highest) | Environment variable | `APCORE_EXTENSIONS_ROOT=./ext` |
| 2 | Configuration file | `extensions.root: "./extensions"` |
| 3 (Lowest) | Default value | `"./extensions"` |

**Environment Variable Naming Convention:**

```
APCORE_{SECTION}_{KEY}

Rules:
  1. Prefix APCORE_ (uppercase)
  2. Nested levels separated by _
  3. All letters uppercase
  4. Hyphens converted to underscores

Examples:
  extensions.root        → APCORE_EXTENSIONS_ROOT
  schema.validation.strict → APCORE_SCHEMA_VALIDATION_STRICT
  acl.default_effect     → APCORE_ACL_DEFAULT_EFFECT
  logging.level          → APCORE_LOGGING_LEVEL
  observability.tracing.enabled → APCORE_OBSERVABILITY_TRACING_ENABLED
```

### 8.3 Configuration Validation Algorithm

Implementations **must** validate configuration at startup:

```
Algorithm: validate_config(config)

Input:
  config — Merged configuration object (env + file + defaults)

Output:
  validated_config — Validated configuration, or throw CONFIG_INVALID error

Steps:
  1. For each required field (MUST):
     If missing → Throw CONFIG_INVALID with missing field path
  2. Type validation:
     For each field, validate value type conforms to Schema definition
  3. Constraint validation:
     - extensions.root must be valid directory path
     - schema.root must be valid directory path
     - acl.default_effect must be "allow" or "deny"
     - observability.tracing.sampling_rate must be in [0.0, 1.0] range
     - extensions.max_depth must be in [1, 16] range
  4. Semantic validation:
     - If extensions.auto_discover == true and extensions.root doesn't exist → Warning
     - If schema.strategy == "yaml_only" and schema.root doesn't exist → Error
  5. Return validated_config
```

---

## 9. Observability Specification (Observability Specification)

### 9.1 Tracing

Based on OpenTelemetry specification:

```yaml
tracing:
  # Trace context
  context:
    trace_id:
      format: "uuid-v4 or w3c trace-id"
      propagation: "Must propagate in call chain"
    span_id:
      format: "16 character hexadecimal"
    parent_span_id:
      format: "16 character hexadecimal"

  # Span creation rules
  spans:
    - name: "module.execute"
      attributes: [module_id, method, duration_ms, success]

    - name: "module.validate"
      attributes: [module_id, valid, error_count]

  # Propagation methods
  propagation:
    - "Auto-propagate through context parameter"
    - "Use W3C Trace Context for HTTP calls"
```

### 9.2 Logging

```yaml
logging:
  levels: [trace, debug, info, warn, error, fatal]

  # Structured logging
  format:
    timestamp: "ISO 8601"
    level: string
    message: string
    trace_id: string
    module_id: string
    extra: object

  # Sensitive data
  sensitive_data:
    - "x-sensitive fields auto-redacted"
    - "Passwords, tokens not logged"
```

### 9.3 Metrics

```yaml
metrics:
  - name: "apcore_module_calls_total"
    type: counter
    labels: [module_id, method, status]

  - name: "apcore_module_duration_seconds"
    type: histogram
    labels: [module_id, method]

  - name: "apcore_module_errors_total"
    type: counter
    labels: [module_id, error_code]
```

### 9.4 Trace ID Format

trace_id **must** use UUID v4 format. In distributed scenarios, **recommended** to be compatible with W3C Trace Context standard.

```yaml
trace_id_spec:
  format: "uuid-v4"                    # MUST
  example: "550e8400-e29b-41d4-a716-446655440000"

  distributed:
    w3c_trace_context: "RECOMMENDED"   # Recommended for distributed scenarios
    traceparent_header: "traceparent: 00-{trace_id_hex}-{span_id_hex}-{flags}"

  generation:
    - "Auto-generated by Executor at top-level calls"
    - "Child calls inherit parent call's trace_id"
    - "MUST NOT allow externally provided unvalidated trace_id"
```

### 9.5 Sensitive Data Redaction

Implementations **must** redact fields marked as `x-sensitive` in logs and trace outputs.

```
Algorithm: redact_sensitive(data, schema)

Input:
  data   — Data object to redact
  schema — Corresponding JSON Schema (contains x-sensitive marking)

Output:
  redacted_data — Redacted data copy

Steps:
  1. redacted ← deep_copy(data)
  2. For each (field_name, field_schema) in schema.properties:
     a. If field_schema["x-sensitive"] == true:
        - If redacted[field_name] exists and is not null:
          redacted[field_name] ← "***REDACTED***"
     b. If field_schema.type == "object" and has properties:
        - Recurse: redacted[field_name] ← redact_sensitive(redacted[field_name], field_schema)
     c. If field_schema.type == "array" and items has x-sensitive:
        - Redact each element in array
  3. Return redacted

Complexity: O(n), where n is number of data fields
```

### 9.6 Sampling Strategy

Implementations **should** support the following sampling strategies:

| Strategy | Configuration Value | Description |
|------|--------|------|
| Full sampling | `sampling_rate: 1.0` | Record all calls (development environment **recommended**) |
| Proportional sampling | `sampling_rate: 0.1` | Record 10% of calls |
| Error-first | `sampling_strategy: "error_first"` | Always record error calls, successful calls by proportion |
| Off | `sampling_rate: 0.0` | Don't record trace info |

Sampling decision **must** be made at call chain root node, child calls **must** inherit parent call's sampling decision.

### 9.7 Span Naming Convention

Implementations **should** follow these Span naming conventions:

```yaml
span_naming:
  pattern: "apcore.{component}.{operation}"
  examples:
    module_execute: "apcore.module.execute"
    module_validate: "apcore.module.validate"
    acl_check: "apcore.acl.check"
    middleware_before: "apcore.middleware.before"
    middleware_after: "apcore.middleware.after"
    schema_validate: "apcore.schema.validate"
    registry_discover: "apcore.registry.discover"

  attributes:
    - "module_id"        # MUST
    - "method"           # MUST (execute|validate|describe)
    - "duration_ms"      # MUST
    - "success"          # MUST (boolean)
    - "error_code"       # SHOULD (when failed)
    - "caller_id"        # SHOULD
```

---

## 10. Extension Mechanism (Extension Mechanism)

### 10.1 Middleware/Interceptors

```yaml
middleware:
  order: "Onion model (first in, last out)"

  hooks:
    - name: "before"
      params: [module_id, inputs, context]
      can_modify: [inputs, context]
      can_abort: true

    - name: "after"
      params: [module_id, output, context]
      can_modify: [output]

    - name: "on_error"
      params: [module_id, error, context]
      can_retry: true
```

### 10.2 Middleware Registration and Priority

```yaml
middleware_registration:
  # Registration methods
  methods:
    # 1. Configuration file registration
    config:
      location: "apcore.yaml"
      example:
        middleware:
          - id: "logging"
            class: "apcore.middleware.LoggingMiddleware"
            priority: 100
            config:
              level: "info"

          - id: "tracing"
            class: "apcore.middleware.TracingMiddleware"
            priority: 90

          - id: "custom"
            class: "my_project.middleware.CustomMiddleware"
            priority: 50

    # 2. Code registration (runtime)
    code:
      example: |
        registry.add_middleware(
            id="custom",
            middleware=CustomMiddleware(),
            priority=50
        )

  # Priority rules
  priority:
    range: "0-1000"
    higher_first: true           # Higher number executes first
    default: 100

    # Recommended values
    recommended:
      framework: "900-1000"      # Framework built-in
      security: "800-899"        # Security-related
      logging: "700-799"         # Logging/tracing
      validation: "600-699"      # Additional validation
      custom: "0-599"            # User-defined

  # Execution order example
  execution_order:
    request: "[1000] → [900] → [800] → [100] → Module"
    response: "Module → [100] → [800] → [900] → [1000]"
```

### 10.3 Custom Extension Points

```yaml
extension_points:
  # Extension point definitions
  points:
    schema_loader:
      description: "Custom Schema loading method"
      interface: "load(module_id: str) -> Schema"
      use_case: "Load Schema from remote service/database; Binding file Schema loading"
      default: "YAMLSchemaLoader"

    id_converter:
      description: "Custom ID conversion rules"
      interface: "to_canonical(local_id, lang) -> str"
      use_case: "Support new programming language"
      default: "DefaultIDConverter"

    module_loader:
      description: "Custom module loading method"
      interface: "load(module_id: str) -> Module"
      use_case: "Remote loading, dynamic compilation; Function-based module loading; Binding file target resolution and module loading"
      default: "DirectoryModuleLoader"

    executor:
      description: "Custom execution method"
      interface: "execute(module, method, input, context) -> output"
      use_case: "Distributed execution, sandbox isolation"
      default: "LocalExecutor"

    acl_checker:
      description: "Custom permission checking"
      interface: "check(caller, target, action, context) -> bool"
      use_case: "Integrate external permission system"
      default: "YAMLACLChecker"

  # Extension point registration
  registration:
    config:
      location: "apcore.yaml"
      example:
        extensions:
          schema_loader: "my_project.loaders.RemoteSchemaLoader"
          executor: "my_project.executors.DistributedExecutor"

    code:
      example: |
        registry.set_extension(
            point="schema_loader",
            implementation=RemoteSchemaLoader(url="https://...")
        )

  # Extension point chaining (multiple implementations)
  chaining:
    enabled: true
    strategy: "first_success"    # first_success | all | fallback
    example:
      schema_loader:
        - "CacheSchemaLoader"    # Check cache first
        - "YAMLSchemaLoader"     # Load from file if cache miss
```

### 10.4 Framework Built-in Middleware

```yaml
builtin_middleware:
  # Must enable (cannot disable)
  required:
    - id: "schema_validation"
      description: "Input/output Schema validation"
      priority: 1000

    - id: "acl_check"
      description: "Permission check"
      priority: 999

  # Default enabled (can disable)
  default_enabled:
    - id: "tracing"
      description: "Trace context propagation"
      priority: 950

    - id: "logging"
      description: "Call logging"
      priority: 900

    - id: "metrics"
      description: "Metrics collection"
      priority: 890

    - id: "error_wrapper"
      description: "Error wrapping and formatting"
      priority: 800

  # Disable built-in middleware
  disable:
    config:
      middleware:
        disabled:
          - "metrics"            # Disable metrics collection
```

### 10.5 Middleware Execution State Machine

Middleware chain execution **must** follow this state machine:

```
                                     Error branch
  ┌──────┐    ┌────────┐    ┌─────────┐    ┌──────┐    ┌──────┐
  │ init │───▶│ before │───▶│ execute │───▶│ after│───▶│ done │
  └──────┘    └───┬────┘    └────┬────┘    └──┬───┘    └──────┘
                  │              │             │
                  │ abort        │ error       │ error
                  ▼              ▼             ▼
              ┌──────┐    ┌──────────┐    ┌──────┐
              │ done │    │ on_error │───▶│ done │
              └──────┘    └──────────┘    └──────┘

State descriptions:
  init     — Initialize middleware chain, prepare execution context
  before   — Execute all middleware before() in order
  execute  — Execute module's execute() method
  after    — Execute all middleware after() in reverse order
  on_error — Execute all middleware on_error() in reverse order
  done     — Execution complete, return result or error

Rules:
  - before phase: If middleware returns non-None → Use as replacement input
  - before phase: If middleware throws exception → Skip subsequent before and execute, enter on_error
  - after phase: If middleware returns non-None → Use as replacement output
  - on_error phase: If middleware returns non-None → Use as fallback result, stop error propagation
```

### 10.6 Extension Point Interface Formalization

Implementations **should** support the following extension points, each **must** define clear interface contract:

```
Extension Point: SchemaLoader
  load(module_id: String) → Schema
  supports(module_id: String) → Boolean
  priority: Integer

Extension Point: ModuleLoader
  load(module_id: String) → Module
  supports(file_path: String) → Boolean
  priority: Integer

Extension Point: IDConverter
  to_canonical(local_id: String, language: String) → String
  from_canonical(canonical_id: String, language: String) → String

Extension Point: ACLChecker
  check(caller_id: String, target_id: String, context: Context) → Boolean

Extension Point: Executor
  execute(module: Module, method: String, inputs: Map, context: Context) → Map
```

### 10.7 Extension Loading Order

Implementations **must** load extensions according to the following algorithm:

```
Algorithm: load_extensions(config, extension_points)

Steps:
  1. Sort each extension point's implementations by priority descending
  2. For each extension point:
     a. If strategy == "first_success": Try in order, first success takes effect
     b. If strategy == "all": Execute all implementations, merge results
     c. If strategy == "fallback": Try in order, try next on failure
  3. If extension point has no available implementation → Use framework default implementation
```

### 10.8 Edge Case Handling

Implementations **must** handle middleware edge cases according to the following table:

#### 10.8.1 on_error Cascade

| Scenario | Behavior | Level |
|------|------|------|
| `on_error()` itself throws exception | Log ERROR, continue to next `on_error()` in chain | **MUST** |
| `on_error()` returns non-`None` value | Stop propagation, use return value as module final output | **MUST** |
| `on_error()` returns `None` | Continue propagating error downward | **MUST** |
| All `on_error()` return `None` | Throw original error to caller | **MUST** |

#### 10.8.2 before() Edges

| Scenario | Behavior | Level |
|------|------|------|
| `before()` returns `None` | Keep `inputs` unchanged, continue chain | **MUST** |
| `before()` returns partial field dict | Merge into `inputs` (shallow merge) | **MUST** |
| `before()` returns non-dict type | Throw `GENERAL_INTERNAL_ERROR` | **MUST** |
| `before()` throws `ModuleError` | Trigger `on_error()` chain, skip module execution | **MUST** |
| `before()` modifies `context.data` | Allowed, modifications visible to subsequent middleware and module | **MUST** |

#### 10.8.3 after() Edges

| Scenario | Behavior | Level |
|------|------|------|
| `after()` returns `None` | Keep `result` unchanged, continue chain | **MUST** |
| `after()` returns partial field dict | Merge into `result` (shallow merge) | **MUST** |
| `after()` throws `ModuleError` | Trigger `on_error()` chain, replace original result | **MUST** |
| `after()` returns value not matching `output_schema` | Trigger `SCHEMA_VALIDATION_ERROR` | **MUST** |

#### 10.8.4 Timeout Related

| Scenario | Behavior | Level |
|------|------|------|
| Timeout occurs in `before()` phase | Throw `MODULE_TIMEOUT`, trigger `on_error()` chain | **MUST** |
| Timeout occurs in `execute()` phase | Throw `MODULE_TIMEOUT`, trigger `on_error()` chain | **MUST** |
| Timeout occurs in `after()` phase | Throw `MODULE_TIMEOUT`, trigger `on_error()` chain | **MUST** |
| `on_error()` handling timeout itself times out | Log ERROR, stop `on_error()` chain, throw original `MODULE_TIMEOUT` | **MUST** |

**Note**:
- Timeout timer should start at first `before()` call
- Timeout enforcement algorithm see §11.7.4 and algorithms.md A22

---

## 11. SDK Implementation Guide (SDK Implementation Guide)

### 11.1 Required Core Components

| Component | Responsibility | Phase |
|------|------|------|
| `IDConverter` | Canonical ID ↔ Local ID conversion | Phase 1 |
| `SchemaLoader` | Load YAML Schema | Phase 1 |
| `Registry` | Module discovery, registration, loading | Phase 1 |
| `Executor` | Module invocation, Schema validation | Phase 1 |
| `ACLChecker` | Permission checking | Phase 2 |
| `MiddlewareManager` | Middleware management | Phase 2 |
| `TracingProvider` | Trace context | Phase 2 |
| `MetricsCollector` | Metrics collection | Phase 2 |

### 11.2 Core Component Interface Contracts

Following are formalized interface definitions for each core component (language-agnostic pseudocode). All SDK implementations **must** provide equivalent implementations of these interfaces.

```
Interface: IDConverter
  /**
   * Convert language-native ID to Canonical ID
   * @param local_id  — Native format ID (e.g., "executor::validator::db_params")
   * @param language  — Source language identifier (python|rust|go|java|typescript)
   * @return canonical_id — Dot-separated snake_case format
   * @throws INVALID_ID — If local_id doesn't conform to language naming convention
   */
  to_canonical(local_id: String, language: String) → String

  /**
   * Convert Canonical ID to language-native ID
   * @param canonical_id — Dot-separated snake_case format
   * @param language     — Target language identifier
   * @return local_id    — Language-native format
   */
  from_canonical(canonical_id: String, language: String) → String

Interface: SchemaLoader
  /**
   * Load specified module's Schema definition
   * @param module_id — Canonical ID
   * @return schema   — Parsed Schema object (contains input_schema, output_schema, description)
   * @throws SCHEMA_NOT_FOUND — If Schema file doesn't exist
   * @throws SCHEMA_INVALID   — If Schema format is invalid
   */
  load(module_id: String) → Schema

  /**
   * Validate Schema itself for validity
   * @param schema — Schema object to validate
   * @return errors — Error list, empty list means valid
   */
  validate(schema: Schema) → List<ValidationError>

Interface: Registry
  /**
   * Scan extension directory, discover and register all modules
   * @param config — Framework configuration
   * @throws EXTENSION_ROOT_NOT_FOUND — If extension root directory doesn't exist
   */
  discover(config: Config) → void

  /**
   * Get specified module
   * @param module_id — Canonical ID
   * @return module   — Module instance
   * @throws MODULE_NOT_FOUND — If module not registered
   */
  get(module_id: String) → Module

  /**
   * List all registered module IDs
   * @return ids — Canonical ID list
   */
  list() → List<String>

  /**
   * Get module description info (for AI/LLM use)
   * @param module_id — Canonical ID
   * @return description — Complete description including Schema, Annotations, Examples
   */
  describe(module_id: String) → ModuleDescription

Interface: Executor
  /**
   * Execute module method
   * @param module_id — Canonical ID
   * @param method    — Method name (execute|validate|describe)
   * @param inputs    — Input parameters (conform to input_schema)
   * @param context   — Execution context
   * @return output   — Output result (conform to output_schema)
   * @throws INPUT_VALIDATION_FAILED  — Input validation failed
   * @throws OUTPUT_VALIDATION_FAILED — Output validation failed
   * @throws ACL_DENIED               — Permission denied
   * @throws MODULE_EXECUTION_ERROR   — Module execution exception
   */
  execute(module_id: String, method: String, inputs: Map, context: Context) → Map

Interface: ACLChecker
  /**
   * Check invocation permission
   * @param caller_id — Caller module ID or identity
   * @param target_id — Target module ID
   * @param context   — Execution context
   * @return allowed  — Whether allowed
   */
  check(caller_id: String, target_id: String, context: Context) → Boolean

Interface: MiddlewareManager
  /**
   * Register middleware
   * @param id         — Middleware identifier
   * @param middleware  — Middleware instance
   * @param priority   — Priority (0-1000, higher executes first)
   */
  add(id: String, middleware: Middleware, priority: Integer) → void

  /**
   * Execute middleware chain in priority order
   * @param module_id — Target module ID
   * @param inputs    — Input parameters
   * @param context   — Execution context
   * @param next      — Next execution function (module's execute)
   * @return output   — Final output
   */
  run_chain(module_id: String, inputs: Map, context: Context, next: Function) → Map

Interface: TracingProvider
  /**
   * Create new Span
   * @param name       — Span name (follows §9.7 naming convention)
   * @param context    — Execution context (contains trace_id, parent_span_id)
   * @return span      — Span object
   */
  start_span(name: String, context: Context) → Span

  /**
   * End Span and record result
   * @param span       — Span to end
   * @param attributes — Additional attributes
   */
  end_span(span: Span, attributes: Map) → void

Interface: MetricsCollector
  /**
   * Record counter metric
   * @param name   — Metric name
   * @param labels — Label key-value pairs
   * @param value  — Increment value (default 1)
   */
  increment(name: String, labels: Map, value: Integer) → void

  /**
   * Record histogram metric
   * @param name   — Metric name
   * @param labels — Label key-value pairs
   * @param value  — Observed value
   */
  observe(name: String, labels: Map, value: Float) → void
```

### 11.3 Cross-language Implementation Requirements

| Requirement | Python | Rust | Go | Java | TypeScript |
|------|--------|------|----|------|------------|
| Schema validation library | Pydantic v2 | serde + validator | struct + validator | Jackson + Bean Validation | Zod / Ajv |
| Async model | asyncio | tokio | goroutine | CompletableFuture / Virtual Threads | async/await (Promise) |
| Package management | pyproject.toml + uv/pip | Cargo.toml | go.mod | Maven / Gradle | package.json |
| JSON Schema support | MUST | MUST | MUST | MUST | MUST |
| YAML parsing | MUST | MUST | MUST | MUST | MUST |
| Directory as ID | MUST | MUST | MUST | MUST | MUST |
| ID Map cross-language conversion | MUST | MUST | MUST | MUST | MUST |
| ACL engine | MUST | MUST | MUST | MUST | MUST |
| Middleware onion model | MUST | MUST | MUST | MUST | MUST |
| OpenTelemetry integration | SHOULD | SHOULD | SHOULD | SHOULD | SHOULD |
| Structured logging | MUST | MUST | MUST | MUST | MUST |
| Error code specification | MUST | MUST | MUST | MUST | MUST |

### 11.4 Consistency Testing Requirements

Each SDK implementation **must** pass the following consistency test suite to ensure cross-language behavior consistency:

```
Consistency Test Suite:

1. ID Conversion Tests:
   - Directory path → Canonical ID (10+ cases, including edge cases)
   - Cross-language ID conversion (5+ cases per language)
   - Invalid ID detection (format errors, too long, reserved words)

2. Schema Validation Tests:
   - Valid input passes validation
   - Invalid input rejected (type error, missing field, extra field)
   - x-sensitive field marking recognition
   - $ref reference resolution (including circular reference detection)

3. ACL Tests:
   - Wildcard matching (*, **)
   - deny takes precedence over allow
   - Default policy takes effect
   - Identity type matching (user, service, agent, api_key, system)

4. Middleware Tests:
   - Onion model execution order
   - before phase modifying input
   - after phase modifying output
   - on_error phase fallback handling
   - Priority sorting correctness

5. Executor Tests:
   - Normal execution flow (input validation → ACL → middleware → execute → output validation)
   - Error propagation and error codes
   - Context propagation (trace_id, call_chain)
   - Circular call detection
   - Call depth limiting

6. Observability Tests:
   - trace_id is valid UUID v4
   - Sensitive data redaction
   - Structured log format
```

### 11.5 Implementation Roadmap

```
Phase 1: Core MVP
├── IDMap (Directory as ID + cross-language conversion)
├── SchemaLoader (YAML → language-native Schema)
├── Registry (directory scanning, registration)
└── Executor (invocation, validation)

Phase 2: Core Complete
├── ACLChecker (permission checking)
├── MiddlewareManager (middleware)
├── ErrorHandler (error handling)
└── ObservabilityProvider (tracing, logging, metrics)

Phase 3: CLI & DX
├── CLI tools (init, create, run)
├── Schema validation tools
└── Developer documentation

Phase 4: Advanced
├── Module hot reloading
├── OpenTelemetry integration
└── Performance optimization
```

### 11.6 Language-specific Guidelines

#### Python
- Schema: Pydantic v2
- Async: asyncio
- Package management: pyproject.toml + uv/pip

#### Rust
- Schema: serde + validator
- Async: tokio
- Package management: Cargo.toml

#### Go
- Schema: struct + validator
- Async: goroutine
- Package management: go.mod

#### Java
- Schema: Jackson + Bean Validation
- Async: CompletableFuture / Virtual Threads
- Package management: Maven / Gradle

### 11.7 Concurrency Model Specification

This section defines apcore's concurrency model and thread safety requirements, ensuring SDK implementers correctly implement the framework in multi-threaded/coroutine environments.

#### 11.7.1 Module Instance Lifecycle

**Singleton Model (MUST)**:

- Each `module_id` **must** correspond to unique module instance (singleton)
- Instance created at `discover()` or first invocation, destroyed at `unregister()` or app shutdown
- `on_load()` hook **must** be called only once during instance lifecycle

**Reentrancy (MUST)**:

- `execute()` method **must** support concurrent reentrant calls (thread-safe)
- Module internal state (if any) **should** use thread-safe mechanisms (locks, atomic variables, etc.) for protection
- Implementations **must not** assume `execute()` calls are serial

**Example — Thread-safe counter module:**

```python
# Python example (similar for other languages)
import threading

class CounterModule:
    def __init__(self):
        self._count = 0
        self._lock = threading.Lock()

    def on_load(self, context):
        # Called only once
        print("Counter module loaded")

    def execute(self, inputs, context):
        # Multi-thread safe
        with self._lock:
            self._count += 1
            return {"count": self._count}
```

**Lifecycle Guarantees:**

| Phase | Call Count | Concurrency | Description |
|------|---------|--------|------|
| `__init__()` | 1 time | Single-thread | Instance creation |
| `on_load()` | 1 time | Single-thread | Module initialization |
| `execute()` | 0..N times | **Multi-thread** | Business logic |
| `on_unload()` | 0..1 time | Single-thread | Resource cleanup |

#### 11.7.2 Context.data Sharing Semantics

**Reference Sharing (MUST)**:

- `context.data` **must** be the same dict/Map object across entire call chain (reference sharing)
- Parent module modifications to `context.data` visible to child modules, vice versa
- When `derive()` creates new Context, `data` field **must** copy reference (not deep copy)

**Isolation (MUST)**:

- Different top-level `call()` invocations **must** use independent `context.data` instances
- Concurrently executing call chains **must not** share `context.data` (avoid race conditions)

**Example — Context.data Sharing:**

```python
# Top-level calls
context1 = Context(data={})
executor.call("module_a", {}, context1)  # context1.data independent

context2 = Context(data={})
executor.call("module_b", {}, context2)  # context2.data independent

# Sharing within call chain
# module_a.execute():
context.data["key"] = "value"  # Write
sub_context = context.derive("module_c")
executor.call("module_c", {}, sub_context)

# module_c.execute():
print(context.data["key"])  # Reads "value" (reference sharing)
```

**Concurrent Access Protection (SHOULD)**:

- If `context.data` might be accessed by multiple threads (e.g., async middleware), **should** use thread-safe Map implementation
- Python's `dict` is partially thread-safe in CPython (GIL protected), but **should** avoid relying on implementation details

#### 11.7.3 Hot Reload Race Conditions

**Problem**: During `unregister()`, module might be executing in other threads.

**Safe Unload Algorithm (MUST)**:

```
Algorithm: safe_unregister(module_id, registry)

Steps:
  1. Mark module as "unloading" state
  2. Remove from registry (new call() will throw MODULE_NOT_FOUND)
  3. Wait for all executing calls to complete:
     - Maintain reference count (number of executing calls)
     - Block until count reaches zero or timeout (default 30 seconds)
  4. Call on_unload() hook
  5. Release module instance

Return:
  - If successfully unloaded → true
  - If timeout → Log ERROR, force unload, return false
```

**For detailed algorithm see algorithms.md A21 — safe_unregister()**

**Operation Concurrency Table:**

| Operation A | Operation B | Behavior | Description |
|--------|--------|------|------|
| `call()` | `unregister()` | If A already started execution, continue to completion; if not started, throw `MODULE_NOT_FOUND` | **MUST** |
| `register()` | `register()` same ID | Latter throws `GENERAL_INVALID_INPUT` | **MUST** |
| `unregister()` | `unregister()` same ID | Idempotent, succeed silently | **MUST** |
| `get()` | `unregister()` | If get executes first, return instance; if unregister executes first, throw `MODULE_NOT_FOUND` | **SHOULD** |

#### 11.7.4 Timeout Enforcement

**Cooperative Cancellation (SHOULD)**:

- SDK **should** prefer cooperative cancellation mechanism (e.g., Python `asyncio.CancelledError`, Go `context.Context`)
- Module **should** check cancellation signal and actively exit

**Forced Termination (MAY)**:

- If module doesn't respond to cancellation signal, SDK **may** forcibly terminate execution thread/coroutine
- After forced termination **must** log ERROR including `module_id` and timeout duration

**Timeout Enforcement Algorithm (MUST)**:

```
Algorithm: enforce_timeout(module_id, inputs, context, timeout_ms)

Steps:
  1. Start timer (from first before() middleware)
  2. Concurrent execution:
     a. Main task: execute_with_middleware(module_id, inputs, context)
     b. Timeout monitor: sleep(timeout_ms)
  3. If main task completes first → Cancel timer, return result
  4. If timeout triggers first:
     - Send cancellation signal (cooperative)
     - Wait maximum grace_period (default 5 seconds)
     - If still hasn't exited → Forcibly terminate (if supported)
     - Throw MODULE_TIMEOUT error

Return:
  - Success → Module output
  - Failure → MODULE_TIMEOUT error
```

**For detailed algorithm see algorithms.md A22 — enforce_timeout()**

**Timeout Levels:**

| Level | Scope | Default | Description |
|------|------|--------|------|
| Global timeout | before + execute + after | 60000ms | Configuration item `executor.timeout` |
| ACL check timeout | ACL rule evaluation | 1000ms | Separate timing |
| Schema validation timeout | Input/output validation | Included in global timeout | Not separately timed |

#### 11.7.5 Middleware Chain Atomicity

**Call-level Isolation (MUST)**:

- Each `call()`'s middleware chain execution **must** not interleave with other `call()`'s chain execution
- Middleware chain's before → execute → after sequence **must** complete atomically (not interrupted by other calls)

**Instance Sharing (MUST)**:

- Middleware instances shared at application level (similar to module singleton)
- Middleware **must** be thread-safe, support concurrent calls

**Example — Middleware Concurrent Execution:**

```
Thread 1: before1 → before2 → execute(A) → after2 → after1
Thread 2:                before1 → before2 → execute(B) → after2 → after1
                       ↑ Interleaving allowed (different calls)

# Forbidden:
Thread 1: before1 → before2 → [interrupted by Thread 2]
Thread 2:                      before1 → ...
```

**State Isolation (SHOULD)**:

- Middleware **should** store call-level state through `context.data` (e.g., request ID, timer)
- **Must not** use middleware instance variables to store call-level state (causes race conditions)

#### 11.7.6 Sync/Async Mixing

**Bridging Strategy (MUST)**:

Implementations **must** support mixed calls of sync and async modules, bridging according to these rules:

| Caller | Called Module | Bridging Strategy | Description |
|--------|-----------|---------|------|
| Sync | Sync | Direct call | No overhead |
| Sync | Async | Block and wait (await) | Sync caller blocks until async completes |
| Async | Sync | Thread pool offload | Avoid blocking event loop |
| Async | Async | Direct await | No overhead |

**Language Mapping:**

| Language | Sync Model | Async Model | Bridging Mechanism |
|------|---------|---------|---------|
| Python | Regular function | `async def` | `asyncio.run()` / `run_in_executor()` |
| JavaScript | Blocking code (rare) | Promise / async/await | Direct await (JS default async) |
| Rust | Regular function | `async fn` | `block_on()` / `spawn_blocking()` |
| Go | goroutine | goroutine + channel | Naturally supported (goroutines lightweight) |
| Java | Thread | CompletableFuture | `join()` / `supplyAsync()` |

**Performance Considerations:**

- Sync→Async bridging blocks caller thread, **should** avoid frequent use in async contexts
- Async→Sync bridging needs thread pool, **should** configure reasonable thread pool size (default CPU cores × 2)

#### 11.7.7 Resource Cleanup Guarantees

Implementations **must** guarantee resource cleanup according to this table:

| Exit Scenario | `on_unload()` Called | `finally` Block Executed | File/Network Closed | Memory Released | Level |
|---------|-------------------|-----------------|--------------|---------|------|
| Normal completion | ✅ Called | ✅ Executed | ✅ Guaranteed | ✅ GC reclaimed | **MUST** |
| Timeout (cooperative cancel) | ✅ Called | ✅ Executed | ✅ Guaranteed | ✅ GC reclaimed | **MUST** |
| Timeout (forced termination) | ⚠️ May not call | ⚠️ May not execute | ⚠️ May leak | ✅ GC reclaimed (eventually) | **MAY** |
| Process crash/kill | ❌ Not called | ❌ Not executed | ❌ Leak | ❌ Lost | N/A |

**Best Practices:**

- Modules **should** use RAII (Resource Acquisition Is Initialization) pattern to manage resources
- **Should** acquire resources in `on_load()`, release in `on_unload()`
- **Should** avoid opening long-term resources in `execute()` (e.g., database connections), use connection pools instead

**Example — Resource Cleanup:**

```python
class DatabaseModule:
    def on_load(self, context):
        self.pool = create_connection_pool()  # Long-term resource

    def execute(self, inputs, context):
        with self.pool.get_connection() as conn:  # Short-term resource, auto-released
            return conn.query(inputs["sql"])

    def on_unload(self):
        self.pool.close()  # Cleanup long-term resource
```

---

## 12. Versioning (Versioning)

### 12.1 Version Number Specification

```
{major}.{minor}.{patch}[-{prerelease}]

major: Incompatible API changes
minor: Backward-compatible feature additions
patch: Backward-compatible bug fixes
prerelease: draft, alpha, beta, rc
```

### 12.2 Compatibility Promise

- **Within major version**: Protocol backward compatible
- **Schema evolution**: Support version declaration, old version Schema readable by new SDK
- **Deprecation policy**: Keep at least 2 minor versions for deprecation period

### 12.3 Version Negotiation Algorithm

When SDK loads configuration or Schema, **must** perform version negotiation:

```
Algorithm: negotiate_version(declared_version, sdk_version)

Input:
  declared_version — Version declared in configuration/Schema (e.g., "1.2.0")
  sdk_version      — Current SDK's supported highest version (e.g., "1.3.0")

Output:
  effective_version — Effective version number, or throw VERSION_INCOMPATIBLE error

Steps:
  1. Parse declared_version as (major_d, minor_d, patch_d)
  2. Parse sdk_version as (major_s, minor_s, patch_s)
  3. If major_d ≠ major_s:
     → Throw VERSION_INCOMPATIBLE ("Major version incompatible")
  4. If minor_d > minor_s:
     → Throw VERSION_INCOMPATIBLE ("SDK version too low, please upgrade")
  5. If minor_d < minor_s:
     → Issue DEPRECATION_WARNING (if minor_s - minor_d > 2)
     → effective_version ← declared_version (backward compatibility mode)
  6. If minor_d == minor_s:
     → effective_version ← max(declared_version, sdk_version)
  7. Return effective_version
```

### 12.4 Schema Migration

When Schema version changes, implementations **should** support automatic migration:

```
Algorithm: migrate_schema(schema, from_version, to_version)

Input:
  schema       — Original Schema object
  from_version — Original version number
  to_version   — Target version number

Output:
  migrated_schema — Migrated Schema, or throw MIGRATION_FAILED error

Steps:
  1. If from_version == to_version → Return schema (no migration needed)
  2. migration_path ← Find migration path from from_version to to_version
     (e.g., 1.0 → 1.1 → 1.2, each step has corresponding migration function)
  3. If migration_path empty → Throw MIGRATION_FAILED ("No available migration path")
  4. current_schema ← deep_copy(schema)
  5. For each (step_from, step_to, migrate_fn) in migration_path:
     a. current_schema ← migrate_fn(current_schema)
     b. Validate current_schema conforms to step_to version's Schema specification
     c. If validation fails → Throw MIGRATION_FAILED with step info
  6. Return current_schema

Migration types:
  - add_field:     Add field (provide default value)
  - rename_field:  Rename field (keep old name as alias)
  - remove_field:  Remove field (mark as deprecated for at least 2 minor versions)
  - change_type:   Type change (only allowed in major version changes)
```

### 12.5 Backward/Forward Compatibility Matrix

| Change Type | Backward Compatible | Forward Compatible | Version Impact | Description |
|----------|----------|----------|----------|------|
| Add optional field | Yes | Yes | patch/minor | New SDK ignores unknown fields, old SDK uses defaults |
| Add required field | No | No | major | Old SDK can't provide new required field |
| Remove optional field | Yes | No | minor | Old SDK passes removed field, new SDK ignores |
| Remove required field | Yes | No | minor | Old SDK still passes field, new SDK ignores |
| Expand field type (e.g., string → string\|number) | Yes | No | minor | Old SDK only sends string, new SDK accepts more types |
| Narrow field type (e.g., string\|number → string) | No | Yes | major | Old SDK might send number, new SDK rejects |
| Add error code | Yes | No | minor | Old SDK might not recognize new error code |
| Remove error code | No | Yes | major | Old SDK's dependent error code disappears |
| Add extension point | Yes | Yes | minor | Doesn't affect existing extension points |
| Modify extension point interface | No | No | major | All extension point implementations need adaptation |
| Add middleware Hook | Yes | Yes | minor | Old middleware doesn't implement new Hook |
| Modify middleware Hook signature | No | No | major | All middleware needs adaptation |

**Compatibility Rules:**

1. **SDK must** ignore unknown configuration fields and Schema properties (forward compatibility foundation)
2. **SDK must** provide reasonable defaults for all new fields (backward compatibility foundation)
3. **SDK should** gracefully handle unknown error codes
4. **SDK must not** remove published public APIs in minor/patch versions

---

## 13. Appendix

### A. Complete Example Project Structure

```
my-ai-project/
├── apcore.yaml              # Framework configuration
├── extensions/                   # Extension directory
│   ├── api/
│   │   └── handler/
│   │       ├── task_submit.py
│   │       └── task_submit_meta.yaml
│   ├── orchestrator/
│   │   └── engine/
│   │       ├── task_flow.py
│   │       └── task_flow_meta.yaml
│   └── executor/
│       ├── validator/
│       │   ├── db_params.py
│       │   └── db_params_meta.yaml
│       └── handler/
│           ├── db_task.py
│           └── db_task_meta.yaml
├── schemas/                      # Schema definitions
│   ├── api.handler.task_submit.schema.yaml
│   ├── orchestrator.engine.task_flow.schema.yaml
│   ├── executor.validator.db_params.schema.yaml
│   └── executor.handler.db_task.schema.yaml
├── acl/                          # Permission configuration
│   └── global_acl.yaml
└── tests/
```

### B. JSON Schema Validation Files

See `.schema.json` files in `schemas/` directory.

### C. Reference Implementations

- **apcore (this project)**: Python reference implementation
- **apflow**: Task orchestration application example based on apcore

### D. Module Exposure Methods Reference (Non-core, Reference Only)

apcore modules can be exposed in multiple forms for external invocation. Following are common AI protocol mapping references, **apcore doesn't provide adapter implementations**.

#### D.1 MCP (Model Context Protocol) Mapping

```yaml
# Module → MCP Tool mapping
mcp_mapping:
  tool:
    name: "{module.id}"
    description: "{module.description}"
    inputSchema: "{module.input_schema}"
    outputSchema: "{module.output_schema}"

    # annotations mapping
    annotations:
      readOnlyHint: "{module.annotations.readonly}"
      destructiveHint: "{module.annotations.destructive}"
      idempotentHint: "{module.annotations.idempotent}"
      openWorldHint: "{module.annotations.open_world}"

  # Example
  example:
    # apcore Module
    module:
      id: "executor.email.send_email"
      description: "Send emails via SMTP"
      input_schema: { ... }
      annotations:
        readonly: false
        destructive: false
        idempotent: false
        open_world: true

    # Map to MCP Tool
    mcp_tool:
      name: "executor.email.send_email"
      description: "Send emails via SMTP"
      inputSchema: { ... }
      outputSchema: { ... }
      annotations:
        readOnlyHint: false
        destructiveHint: false
        idempotentHint: false
        openWorldHint: true
```

#### D.2 A2A (Agent-to-Agent) Mapping

```yaml
# Module → A2A Skill mapping
a2a_mapping:
  skill:
    id: "{module.id}"
    name: "{module.name}"
    description: "{module.description}"
    tags: "{module.tags}"
    examples: "{module.examples[*].title}"
    inputSchema: "{module.input_schema}"
    outputSchema: "{module.output_schema}"
    inputModes: ["application/json"]
    outputModes: ["application/json"]
```

#### D.3 OpenAI Function Calling Mapping

```yaml
# Module → OpenAI Function mapping
openai_mapping:
  function:
    name: "{module.id.replace('.', '_')}"  # OpenAI doesn't support dots
    description: "{module.description}"
    parameters: "{module.input_schema}"
    strict: true

  # annotations mapping (OpenAI Agents SDK)
  agents_sdk:
    needs_approval: "{module.annotations.requires_approval}"
```

#### D.4 Anthropic Claude Tool Mapping

```yaml
# Module → Anthropic Tool mapping
anthropic_mapping:
  tool:
    name: "{module.id.replace('.', '_')}"
    description: "{module.description}"
    input_schema: "{module.input_schema}"
    input_examples: "{module.examples[*].inputs}"
```

#### D.5 LangChain Tool Mapping

```python
# Module → LangChain Tool
from langchain.tools import StructuredTool

def module_to_langchain_tool(module):
    return StructuredTool.from_function(
        func=module.execute,
        name=module.id,
        description=module.description,
        args_schema=module.input_schema,  # Pydantic model
        tags=module.tags,
        metadata=module.metadata,
    )
```

### E. Module Definition Methods Comparison

apcore supports three module definition methods to meet different scenario needs:

| Dimension | Class-based (Class Definition) | Function-based (Functional) | External Binding (External Binding) |
|------|---------------------|------------------------|---------------------------|
| **Definition Method** | Inherit Module base class | `@module` / `module()` | YAML binding file |
| **Schema Source** | Pydantic Model / YAML | Type annotation auto-generation | YAML explicit definition / auto_schema |
| **Code Invasiveness** | High (need inherit base class) | Low (add decorator or function call) | Zero (no source code modification) |
| **Applicable Scenarios** | New module development | Existing function/method wrapping | Existing application zero-modification integration |
| **Lifecycle Hooks** | on_load / on_unload | Not supported | Not supported |
| **Advanced Features** | Full support | Partial support (annotations, tags, etc.) | Full support (via YAML configuration) |
| **Cross-language** | Each language implements base class | Each language implements module() | Universal YAML format |

**Cross-language Syntax Reference:**

| Language | Class-based | Function-based (Decorator) | Function-based (Function Call) | External Binding |
|------|------------|--------------------------|-------------------------|-----------------|
| Python | `class M(Module)` | `@module(id=...)` | `module(fn, id=...)` | YAML |
| TypeScript | `class M extends Module` | `@module({id: ...})` | `module(fn, {id: ...})` | YAML |
| Java | `class M implements Module` | `@Module(id=...)` | `Apcore.module(fn, id)` | YAML |
| Rust | `impl Module for M` | `#[module(id=...)]` | `apcore::module(id, fn)` | YAML |
| Go | `type M struct` + `Module` interface | — | `apcore.Module(id, fn)` | YAML |
| C | `apcore_module_t` struct | — | `apcore_module(id, fn)` | YAML |

---

## Revision History

| Version | Date | Change Description |
|------|------|----------|
| 1.0.0-draft | 2026-02-05 | Initial draft |
| 1.1.0-draft | 2026-02-07 | Added §5.11 Function-based Module Definition, §5.12 External Schema Binding, Appendix E Module Definition Methods Comparison |
| 1.2.0-draft | 2026-02-09 | Revised §4.3 supplemented x-llm-description usage guide; Added §4.16 Strict Mode Export, §4.17 Export Profile |
