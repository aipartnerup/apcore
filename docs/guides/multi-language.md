# apcore — Multi-Language Development Guide

> Use YAML Schema as a shared contract to develop apcore modules in Python, Rust, Go, Java, TypeScript, and other languages.

## 1. Overview

One of the core designs of apcore is **cross-language interoperability**. Using YAML Schema as a shared contract, SDKs in different languages can load the same Schema definitions to achieve consistent module interfaces.

**Cross-Language Architecture:**

```
                    YAML Schema (Shared Contract)
                          │
          ┌───────┬───────┼───────┬───────┐
          ▼       ▼       ▼       ▼       ▼
       Python   Rust     Go     Java    TypeScript
       Pydantic serde   struct  Jackson  Zod/Ajv
          │       │       │       │       │
          └───────┴───────┴───────┴───────┘
                          │
                    Canonical ID
                (Unified Addressing System)
```

**Core Principles:**

| Principle | Description |
|------|------|
| **Schema is the Source of Truth** | YAML Schema files define the input/output structure of modules, with each language SDK generating or loading types from them |
| **Canonical ID is the Unified Address** | All languages use the same dot-separated snake_case ID to reference modules |
| **Type Mapping has Standards** | Clear mapping table from JSON Schema types to language-specific types |
| **Behavior Through Annotations** | Module behavior annotations (readonly, destructive, etc.) are universal across languages |

---

## 2. Cross-Language Development Model

### 2.1 How apcore Supports Multiple Languages

apcore achieves cross-language support through the following mechanisms:

1. **YAML Schema Sharing**: Module `input_schema` and `output_schema` are defined in YAML files, read by all language SDKs
2. **Canonical ID**: Module IDs use language-agnostic dot-separated snake_case format (e.g., `executor.validator.db_params`)
3. **ID Map**: Converts local naming conventions of each language (PascalCase, camelCase, etc.) to Canonical ID
4. **JSON Schema Draft 2020-12**: Schema is based on international standards, with mature validation libraries available in all languages

```
Development Workflow:
  1. Define YAML Schema → schemas/executor/validator/db_params.schema.yaml
  2. Each language SDK loads the Schema
  3. Implement Module interface (execute method)
  4. Framework automatically handles Schema validation, ACL checks, etc.
```

### 2.2 YAML Schema as a Shared Contract

```yaml
# schemas/executor/email/send_email.schema.yaml
# This file is shared by SDKs in all languages

$schema: "https://apcore.dev/schema/v1"
version: "1.0.0"
module_id: "executor.email.send_email"

description: |
  Email sending module.
  Sends emails via SMTP protocol.

input_schema:
  type: object
  properties:
    to:
      oneOf:
        - type: string
          format: email
        - type: array
          items:
            type: string
            format: email
      description: "Recipient(s)"
    subject:
      type: string
      maxLength: 200
      description: "Email subject"
    body:
      type: string
      description: "Email body (plain text)"
    html:
      type: string
      description: "Email body (HTML)"
    smtp_host:
      type: string
      description: "SMTP server address"
    smtp_port:
      type: integer
      description: "SMTP server port (default 587)"
      default: 587
  required: [to, subject]
  additionalProperties: false

output_schema:
  type: object
  properties:
    success:
      type: boolean
      description: "Whether the send was successful"
    message_id:
      type: string
      description: "Message ID"
  required: [success]
```

### 2.3 Canonical ID as Unified Addressing

Canonical ID is the globally unique identifier for a module, using a language-agnostic format:

```
Format: dot-separated snake_case
Regex: ^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$
Max length: 128 characters

Examples:
  executor.validator.db_params
  orchestrator.engine.task_flow
  api.handler.task_submit
  common.util.sql_parser
```

Each language uses the same Canonical ID to reference modules:

```python
# Python
result = executor.call("executor.email.send_email", inputs, context)
```

```rust
// Rust
let result = executor.call("executor.email.send_email", &inputs, &context)?;
```

```go
// Go
result, err := executor.Call("executor.email.send_email", inputs, ctx)
```

```java
// Java
Map<String, Object> result = executor.call("executor.email.send_email", inputs, context);
```

```typescript
// TypeScript
const result = await executor.call("executor.email.send_email", inputs, context);
```

---

## 3. YAML Schema Sharing

### 3.1 Schema Files as the Source of Truth

In multi-language projects, YAML Schema files are the **Single Source of Truth**. Each language SDK should generate or load type definitions from Schema files, rather than writing them manually.

```
schemas/
├── executor/
│   ├── email/
│   │   └── send_email.schema.yaml
│   └── validator/
│       └── db_params.schema.yaml
├── orchestrator/
│   └── engine/
│       └── task_flow.schema.yaml
└── common/
    ├── error.schema.yaml
    └── pagination.schema.yaml
```

### 3.2 How Each Language SDK Consumes Schema

| Language | Consumption Method | Advantages | Disadvantages |
|------|----------|------|------|
| Python | Runtime loading → Pydantic dynamic generation | Flexible, no compilation needed | Slightly slower startup |
| Rust | Compile-time code generation → serde struct | Type-safe, zero runtime overhead | Requires build step |
| Go | Compile-time code generation → struct | Type-safe | Requires build step |
| Java | Compile-time code generation → POJO | Type-safe, good IDE support | Requires build step |
| TypeScript | Runtime loading → Zod schema | Flexible, good type inference | Runtime overhead |

### 3.3 Code Generation vs Runtime Loading

**Code Generation (Recommended for compiled languages):**

```bash
# Generate Rust struct from YAML Schema
apcore codegen --lang rust --schema schemas/ --output src/generated/

# Generate Go struct from YAML Schema
apcore codegen --lang go --schema schemas/ --output pkg/generated/

# Generate Java POJO from YAML Schema
apcore codegen --lang java --schema schemas/ --output src/main/java/generated/
```

**Runtime Loading (Recommended for dynamic languages):**

```python
# Python: Load YAML Schema at runtime
from apcore import SchemaLoader

loader = SchemaLoader(schemas_dir="./schemas")
schema = loader.load("executor.email.send_email")
# schema.input_schema -> Pydantic model class
# schema.output_schema -> Pydantic model class
```

```typescript
// TypeScript: Load YAML Schema at runtime
import { SchemaLoader } from '@apcore/sdk';

const loader = new SchemaLoader({ schemasDir: './schemas' });
const schema = await loader.load('executor.email.send_email');
// schema.inputSchema -> Zod schema
// schema.outputSchema -> Zod schema
```

---

## 4. SDK Patterns for Each Language

### 4.1 Python: Pydantic Integration

```python
# extensions/executor/email/send_email.py

from apcore import Module, ModuleAnnotations, ModuleExample, Context
from pydantic import BaseModel, Field
from typing import Literal, Optional, Union

class SendEmailInput(BaseModel):
    """Email sending input"""
    to: Union[str, list[str]] = Field(..., description="Recipient(s)")
    subject: str = Field(..., description="Email subject", max_length=200)
    body: Optional[str] = Field(None, description="Email body (plain text)")
    html: Optional[str] = Field(None, description="Email body (HTML)")
    smtp_host: Optional[str] = Field(None, description="SMTP server address")
    smtp_port: Optional[int] = Field(587, description="SMTP server port")

class SendEmailOutput(BaseModel):
    """Email sending output"""
    success: bool = Field(..., description="Whether the send was successful")
    message_id: Optional[str] = Field(None, description="Message ID")

class SendEmail(Module):
    """Email sending module"""

    input_schema = SendEmailInput
    output_schema = SendEmailOutput
    description = "Send email via SMTP"

    annotations = ModuleAnnotations(
        readonly=False,
        destructive=False,
        idempotent=False,
        open_world=True
    )

    def execute(self, inputs: dict, context: Context) -> dict:
        params = SendEmailInput(**inputs)
        # Implement sending logic...
        return SendEmailOutput(
            success=True,
            message_id="msg_123"
        ).model_dump()
```

### 4.2 Rust: serde + jsonschema

```rust
// extensions/executor/email/send_email.rs

use apcore::{Module, Context, ModuleAnnotations, ModuleError};
use serde::{Deserialize, Serialize};
use serde_json::Value;

#[derive(Deserialize, Debug)]
pub struct SendEmailInput {
    pub to: StringOrArray,
    pub subject: String,
    pub body: Option<String>,
    pub html: Option<String>,
    pub smtp_host: Option<String>,
    pub smtp_port: Option<u16>,
}

#[derive(Serialize, Debug)]
pub struct SendEmailOutput {
    pub success: bool,
    pub message_id: Option<String>,
}

// Support for string or string[] input
#[derive(Deserialize, Debug)]
#[serde(untagged)]
pub enum StringOrArray {
    Single(String),
    Multiple(Vec<String>),
}

pub struct SendEmail;

impl Module for SendEmail {
    fn execute(
        &self,
        inputs: Value,
        context: &Context,
    ) -> Result<Value, ModuleError> {
        let params: SendEmailInput = serde_json::from_value(inputs)
            .map_err(|e| ModuleError::validation(e.to_string()))?;

        // Implement SMTP sending logic...

        let output = SendEmailOutput {
            success: true,
            message_id: Some("msg_123".to_string()),
        };

        serde_json::to_value(output)
            .map_err(|e| ModuleError::internal(e.to_string()))
    }

    fn annotations(&self) -> ModuleAnnotations {
        ModuleAnnotations {
            readonly: false,
            destructive: false,
            idempotent: false,
            open_world: true,
            requires_approval: false,
        }
    }
}
```

### 4.3 Go: encoding/json + gojsonschema

```go
// extensions/executor/email/send_email.go

package email

import (
    "encoding/json"
    "github.com/apcore/apcore-go/pkg/apcore"
)

// SendEmailInput email sending input
type SendEmailInput struct {
    To       interface{} `json:"to" validate:"required"` // string or []string
    Subject  string      `json:"subject" validate:"required,max=200"`
    Body     *string     `json:"body,omitempty"`
    HTML     *string     `json:"html,omitempty"`
    SMTPHost *string     `json:"smtp_host,omitempty"`
    SMTPPort *int        `json:"smtp_port,omitempty"`
}

// SendEmailOutput email sending output
type SendEmailOutput struct {
    Success   bool    `json:"success"`
    MessageID *string `json:"message_id,omitempty"`
}

// SendEmail email sending module
type SendEmail struct{}

func (m *SendEmail) Execute(
    inputs map[string]interface{},
    ctx *apcore.Context,
) (map[string]interface{}, error) {
    // Parse input
    data, err := json.Marshal(inputs)
    if err != nil {
        return nil, apcore.NewModuleError("MODULE_EXECUTE_ERROR", err.Error())
    }

    var params SendEmailInput
    if err := json.Unmarshal(data, &params); err != nil {
        return nil, apcore.NewValidationError(err.Error())
    }

    // Implement sending logic...

    msgID := "msg_123"
    output := SendEmailOutput{
        Success:   true,
        MessageID: &msgID,
    }

    // Convert to map
    result, _ := json.Marshal(output)
    var resultMap map[string]interface{}
    json.Unmarshal(result, &resultMap)
    return resultMap, nil
}

func (m *SendEmail) Annotations() apcore.ModuleAnnotations {
    return apcore.ModuleAnnotations{
        Readonly:    false,
        Destructive: false,
        Idempotent:  false,
        OpenWorld:   true,
    }
}

func (m *SendEmail) Description() string {
    return "Send email via SMTP"
}
```

### 4.4 Java: Jackson + json-schema-validator

```java
// extensions/executor/email/SendEmail.java

package com.example.extensions.executor.email;

import com.apcore.Module;
import com.apcore.Context;
import com.apcore.ModuleAnnotations;
import com.apcore.ModuleError;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.Map;
import java.util.Optional;

public class SendEmail implements Module {

    private static final ObjectMapper mapper = new ObjectMapper();

    // Input type
    public static class Input {
        @JsonProperty("to")
        public Object to; // String or List<String>

        @JsonProperty("subject")
        public String subject;

        @JsonProperty("body")
        public Optional<String> body = Optional.empty();

        @JsonProperty("html")
        public Optional<String> html = Optional.empty();

        @JsonProperty("smtp_host")
        public Optional<String> smtpHost = Optional.empty();

        @JsonProperty("smtp_port")
        public Optional<Integer> smtpPort = Optional.empty();
    }

    // Output type
    public static class Output {
        @JsonProperty("success")
        public boolean success;

        @JsonProperty("message_id")
        public String messageId;
    }

    @Override
    public Map<String, Object> execute(
            Map<String, Object> inputs,
            Context context
    ) throws ModuleError {
        Input params = mapper.convertValue(inputs, Input.class);

        // Implement sending logic...

        Output output = new Output();
        output.success = true;
        output.messageId = "msg_123";

        return mapper.convertValue(output, Map.class);
    }

    @Override
    public String getDescription() {
        return "Send email via SMTP";
    }

    @Override
    public ModuleAnnotations getAnnotations() {
        return new ModuleAnnotations.Builder()
            .readonly(false)
            .destructive(false)
            .idempotent(false)
            .openWorld(true)
            .build();
    }
}
```

### 4.5 TypeScript: Zod or Ajv

```typescript
// extensions/executor/email/sendEmail.ts

import { Module, Context, ModuleAnnotations } from '@apcore/sdk';
import { z } from 'zod';

// Define Schema using Zod (keep consistent with YAML Schema)
const SendEmailInputSchema = z.object({
  to: z.union([
    z.string().email(),
    z.array(z.string().email())
  ]).describe('Recipient(s)'),
  subject: z.string().max(200).describe('Email subject'),
  body: z.string().optional().describe('Email body (plain text)'),
  html: z.string().optional().describe('Email body (HTML)'),
  smtp_host: z.string().optional().describe('SMTP server address'),
  smtp_port: z.number().optional().describe('SMTP server port'),
});

const SendEmailOutputSchema = z.object({
  success: z.boolean().describe('Whether the send was successful'),
  message_id: z.string().optional().describe('Message ID'),
});

type SendEmailInput = z.infer<typeof SendEmailInputSchema>;
type SendEmailOutput = z.infer<typeof SendEmailOutputSchema>;

export class SendEmail extends Module {
  static inputSchema = SendEmailInputSchema;
  static outputSchema = SendEmailOutputSchema;

  description = 'Send email via SMTP';

  annotations: ModuleAnnotations = {
    readonly: false,
    destructive: false,
    idempotent: false,
    openWorld: true,
    requiresApproval: false,
  };

  async execute(
    inputs: Record<string, unknown>,
    context: Context
  ): Promise<Record<string, unknown>> {
    const params = SendEmailInputSchema.parse(inputs);

    // Implement sending logic...

    const output: SendEmailOutput = {
      success: true,
      message_id: 'msg_123',
    };

    return output;
  }
}
```

---

## 5. ID Map Configuration

### 5.1 When ID Map is Needed

ID Map is used to convert local naming conventions of each language to Canonical ID.

**Scenarios requiring ID Map:**

| Scenario | Description |
|------|------|
| Rust modules | Rust uses `::` separator and PascalCase struct names |
| Java modules | Java uses `.` separator and PascalCase class names, with package names |
| Mixed-language projects | Modules in different languages need to call each other |
| Custom mapping | When automatic conversion rules don't meet requirements |

**Scenarios NOT requiring ID Map:**

| Scenario | Description |
|------|------|
| Pure Python projects | Python already uses snake_case and `.` separator |
| Pure Go projects | Go also uses `.` separator |
| All modules follow standard naming | Auto-detection is sufficient |

### 5.2 Configuration Example

```yaml
# apcore.yaml

id_map:
  # Auto-detect: determine language by file extension
  auto_detect: true

  # Language rules (defaults are usually sufficient)
  languages:
    python:
      extensions: [".py"]
      separator: "."
      file_case: "snake_case"
      class_case: "PascalCase"

    rust:
      extensions: [".rs"]
      separator: "::"
      file_case: "snake_case"
      struct_case: "PascalCase"

    go:
      extensions: [".go"]
      separator: "."
      file_case: "snake_case"
      struct_case: "PascalCase"

    java:
      extensions: [".java"]
      separator: "."
      file_case: "PascalCase"
      class_case: "PascalCase"

    typescript:
      extensions: [".ts", ".tsx"]
      separator: "."
      file_case: "camelCase"
      class_case: "PascalCase"

  # Special mappings (override auto rules)
  overrides:
    "executor.validator.db_params":
      java:
        class: "com.mycompany.DbParamsValidator"
        package: "com.mycompany.validators"
      rust:
        module: "executor::validator::db_params"
        struct: "DbParamsValidator"
```

### 5.3 Mapping from Language Naming Conventions to Canonical ID

```
Python:
  File: extensions/executor/validator/db_params.py
  Class: DbParamsValidator
  → Canonical ID: executor.validator.db_params

Rust:
  File: extensions/executor/validator/db_params.rs
  Module path: executor::validator::db_params
  Struct: DbParamsValidator
  → Canonical ID: executor.validator.db_params

Go:
  File: extensions/executor/validator/db_params.go
  Package: validator
  Struct: DbParamsValidator
  → Canonical ID: executor.validator.db_params

Java:
  File: extensions/executor/validator/DbParams.java
  Package: com.example.extensions.executor.validator
  Class: DbParamsValidator
  → Canonical ID: executor.validator.db_params

TypeScript:
  File: extensions/executor/validator/dbParams.ts
  Class: DbParamsValidator
  → Canonical ID: executor.validator.db_params
```

---

## 6. Type Mapping Reference

### 6.1 Basic Type Mapping Quick Reference

| JSON Schema Type | Python | Rust | Go | Java | TypeScript |
|-----------------|--------|------|----|------|------------|
| `string` | `str` | `String` | `string` | `String` | `string` |
| `integer` | `int` | `i64` | `int64` | `long` / `Long` | `number` |
| `number` | `float` | `f64` | `float64` | `double` / `Double` | `number` |
| `boolean` | `bool` | `bool` | `bool` | `boolean` / `Boolean` | `boolean` |
| `null` | `None` | `Option::None` | `nil` | `null` | `null` |
| `object` | `dict[str, Any]` | `HashMap<String, Value>` | `map[string]any` | `Map<String, Object>` | `Record<string, unknown>` |
| `array` | `list[T]` | `Vec<T>` | `[]T` | `List<T>` | `T[]` |

### 6.2 Format Type Mapping

| JSON Schema Format | Python | Rust | Go | Java | TypeScript |
|-----------------|--------|------|----|------|------------|
| `format: date-time` | `datetime` | `chrono::DateTime<Utc>` | `time.Time` | `Instant` / `ZonedDateTime` | `Date` / `string` |
| `format: date` | `date` | `chrono::NaiveDate` | `time.Time` | `LocalDate` | `string` |
| `format: uuid` | `uuid.UUID` | `uuid::Uuid` | `uuid.UUID` | `UUID` | `string` |
| `format: email` | `str` | `String` | `string` | `String` | `string` |
| `format: uri` | `str` | `String` / `url::Url` | `string` / `*url.URL` | `URI` | `string` |

### 6.3 Composite Type Mapping

| JSON Schema | Python | Rust | Go | Java | TypeScript |
|------------|--------|------|----|------|------------|
| `enum: [...]` | `Literal[...]` | `enum` | `string` (const) | `enum` | `union type` |
| `oneOf` / `anyOf` | `Union[A, B]` | `enum { A(A), B(B) }` | `interface{}` | `Object` | `A \| B` |
| `T \| null` | `Optional[T]` | `Option<T>` | `*T` | `@Nullable T` | `T \| null` |
| `additionalProperties: T` | `dict[str, T]` | `HashMap<String, T>` | `map[string]T` | `Map<String, T>` | `Record<string, T>` |

For complete type mapping reference, see [Type Mapping Specification](../spec/type-mapping.md).

---

## 7. Common Pitfalls

### 7.1 Integer Precision Issues (JavaScript/JSON 53-bit Limitation)

JavaScript's `Number` type uses IEEE 754 double-precision floating-point, with a safe integer range of `-2^53 + 1` to `2^53 - 1` (i.e., `Number.MAX_SAFE_INTEGER = 9007199254740991`).

**Problem:**

```json
// JSON returned from backend
{"order_id": 9007199254740993}

// After JavaScript parsing
JSON.parse('{"order_id": 9007199254740993}')
// → { order_id: 9007199254740992 }  Precision loss!
```

**Solution:**

```yaml
# Use string to transmit large integers in Schema
properties:
  order_id:
    type: string
    pattern: "^[0-9]+$"
    description: "Order ID (string format to avoid precision loss)"
    x-llm-description: "Large integer transmitted as string to avoid JavaScript precision loss"
```

**Handling by Language:**

| Language | Recommendation |
|------|------|
| Python | `int` has unlimited precision, no special handling needed |
| Rust | Use `i64` or `i128`, serialize using `String` |
| Go | Use `int64`, be careful with large numbers in JSON serialization |
| Java | Use `long` or `BigInteger`, Jackson handles by default |
| TypeScript | Use `bigint` or `string`, recommend `string` for transmission |

### 7.2 DateTime and Timezone Handling

**Problem:** Different languages handle timezones differently by default, which can lead to time discrepancies.

```yaml
# Schema recommends using ISO 8601 + UTC
properties:
  created_at:
    type: string
    format: date-time
    description: "Creation time (ISO 8601, UTC)"
    x-examples: ["2026-02-07T10:30:00Z"]
```

**Timezone Handling by Language:**

```python
# Python: Always use UTC
from datetime import datetime, timezone

now = datetime.now(timezone.utc)
iso_str = now.isoformat()  # "2026-02-07T10:30:00+00:00"
```

```rust
// Rust: Use chrono's Utc
use chrono::Utc;

let now = Utc::now();
let iso_str = now.to_rfc3339(); // "2026-02-07T10:30:00+00:00"
```

```go
// Go: Use time.UTC
now := time.Now().UTC()
isoStr := now.Format(time.RFC3339) // "2026-02-07T10:30:00Z"
```

```java
// Java: Use Instant
import java.time.Instant;

Instant now = Instant.now();
String isoStr = now.toString(); // "2026-02-07T10:30:00Z"
```

```typescript
// TypeScript: Use toISOString()
const now = new Date();
const isoStr = now.toISOString(); // "2026-02-07T10:30:00.000Z"
```

**Best Practices:**

| Rule | Description |
|------|------|
| Store in UTC | All timestamps unified in UTC for storage and transmission |
| Display in local timezone | Convert to local timezone only at UI layer |
| Always include timezone info | Avoid using "naive" time (time without timezone) |
| Use ISO 8601 format | Uniformly use `2026-02-07T10:30:00Z` format |

### 7.3 Unicode Normalization Differences

**Problem:** Different operating systems and languages have different Unicode string normalization forms.

```
"cafe\u0301" (e + combining accent) vs "caf\u00e9" (precomposed e)
These two are visually identical but have different bytes.
```

**Recommended Practice:**

```yaml
# Annotate Unicode handling requirements in Schema
properties:
  name:
    type: string
    description: "Username"
    x-constraints: "MUST use NFC normalization form"
```

**Normalization by Language:**

```python
# Python
import unicodedata
normalized = unicodedata.normalize("NFC", raw_string)
```

```rust
// Rust
use unicode_normalization::UnicodeNormalization;
let normalized: String = raw_string.nfc().collect();
```

```go
// Go
import "golang.org/x/text/unicode/norm"
normalized := norm.NFC.String(rawString)
```

```java
// Java
import java.text.Normalizer;
String normalized = Normalizer.normalize(rawString, Normalizer.Form.NFC);
```

```typescript
// TypeScript
const normalized = rawString.normalize('NFC');
```

### 7.4 Null vs Undefined vs Missing Fields

**Problem:** `null` in JSON, missing fields, and language-specific concepts (like JavaScript's `undefined`) have different semantics.

```json
// Three different situations
{"name": null}      // Field exists, value is null
{"name": ""}        // Field exists, value is empty string
{}                  // Field does not exist (missing)
```

**Handling in Schema:**

```yaml
properties:
  name:
    type: ["string", "null"]   # Allow string or null
    description: "Username"

required: ["name"]  # Field must exist (but value can be null)
```

**Mapping by Language:**

| JSON State | Python | Rust | Go | Java | TypeScript |
|-----------|--------|------|----|------|------------|
| `"value"` | `"value"` | `Some("value")` | `"value"` | `"value"` | `"value"` |
| `null` | `None` | `None` | `nil` | `null` | `null` |
| Missing | `KeyError` | Deserialization failure / `None`(serde default) | Zero value | `null` | `undefined` |

**Best Practices:**

```yaml
# Clearly distinguish between required and optional

# Required field: must exist and not be null
required_field:
  type: string

# Optional field: can be absent, not null when present
optional_field:
  type: string
  default: "default value"

# Nullable field: must exist, but can be null
nullable_field:
  type: ["string", "null"]
```

### 7.5 Floating-Point Precision

**Problem:** IEEE 754 floating-point numbers may produce different precision results in different languages.

```python
# Python
0.1 + 0.2  # 0.30000000000000004
```

**Recommended Practice:**

- Use `integer` for monetary fields (in cents) or `string` (precise decimal)
- When precise comparison is needed, use epsilon tolerance

```yaml
# Monetary handling in Schema
properties:
  amount_cents:
    type: integer
    description: "Amount (unit: cents)"
    minimum: 0
    x-llm-description: "Amount as integer in cents, e.g., 1999 represents 19.99 dollars"

  # Or use string
  amount:
    type: string
    pattern: "^\\d+\\.\\d{2}$"
    description: "Amount (string format, precise to cents)"
    x-examples: ["19.99", "100.00"]
```

### 7.6 Enum Value Serialization Differences

**Problem:** Different languages have different serialization conventions for enums.

```yaml
# Uniformly use snake_case in Schema
properties:
  status:
    type: string
    enum: ["pending", "in_progress", "completed", "failed"]
```

| Language | Local Convention | Recommendation |
|------|----------|------|
| Python | `snake_case` | Use directly |
| Rust | `PascalCase` | serde `rename_all = "snake_case"` |
| Go | `PascalCase` | JSON tag `json:"status"` |
| Java | `UPPER_SNAKE_CASE` | `@JsonValue` annotation |
| TypeScript | `camelCase` | Use values defined in Schema directly |

---

## Next Steps

- [Schema Definition Guide](./schema-definition.md) - Deep dive into Schema definitions
- [Module Testing Guide](./testing-modules.md) - Cross-language testing strategies
- [Creating Modules Guide](./creating-modules.md) - Complete module creation tutorial
- [Architecture Design](../architecture.md) - Overall system architecture
