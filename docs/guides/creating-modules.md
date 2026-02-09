# Creating Modules Guide

> Build an apcore module from scratch.

## Four Module Definition Approaches

apcore supports four ways to create modules - choose the one that fits your scenario:

| Approach | Use Case | Code Intrusiveness | Jump To |
|------|---------|-----------|------|
| **Class-based** (Class Definition) | New module development | High (inherits base class) | [Quick Start](#quick-start) |
| **`@module` Decorator** | Functions where you can modify source code | Low (add one line decorator) | [module() Registration](#module-registration) |
| **`module()` Function Call** | Wrapping existing classes/methods | Very Low (no changes to original function) | [module() Registration](#module-registration) |
| **External Binding** (External Binding) | Zero-modification integration of existing apps | None (no source code changes) | [External Schema Binding](#external-schema-binding-yaml) |

---

## Quick Start

### 1. Create Project Structure

```bash
my-project/
├── apcore.yaml           # Framework configuration
├── extensions/           # Extensions directory
│   └── executor/         # Execution layer
│       └── email/        # Email functionality
│           └── send_email.py   # Module file
└── schemas/              # Schema definitions (optional, used for cross-language)
```

### 2. Create Module File

```python
# extensions/executor/email/send_email.py

from pydantic import BaseModel, Field
from apcore import Module, Context


# Step 1: Define input Schema
class SendEmailInput(BaseModel):
    """Input parameters for sending email"""
    to: str = Field(..., description="Recipient email address")
    subject: str = Field(..., description="Email subject")
    body: str = Field(..., description="Email body")


# Step 2: Define output Schema
class SendEmailOutput(BaseModel):
    """Output result for sending email"""
    success: bool = Field(..., description="Whether successful")
    message_id: str | None = Field(None, description="Message ID")


# Step 3: Create module
class SendEmailModule(Module):
    """Send email module"""  # This is the description

    input_schema = SendEmailInput
    output_schema = SendEmailOutput

    def execute(self, inputs: dict, context: Context) -> dict:
        # Implement sending logic
        to = inputs["to"]
        subject = inputs["subject"]
        body = inputs["body"]

        # ... send email ...
        message_id = "msg_123"

        return {"success": True, "message_id": message_id}
```

### 3. Module ID Auto-Generation

```
File path: extensions/executor/email/send_email.py
Module ID:  executor.email.send_email
```

**No configuration needed - the file path is the ID.**

---

## Detailed Steps

### Step 1: Design Schema

**First think about module inputs and outputs:**

| Question | Example (Send Email) |
|------|------------------|
| What inputs are needed? | to, subject, body, cc |
| What outputs are returned? | success, message_id, error |
| What constraints exist? | to must be email format, subject max 200 chars |

**Define Input Schema:**

```python
from pydantic import BaseModel, Field
from typing import Literal

class SendEmailInput(BaseModel):
    """Input parameters - each field must have description"""

    to: str = Field(
        ...,                              # ... means required
        description="Recipient email address",       # AI uses this to understand
        pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$"  # Email format validation
    )

    subject: str = Field(
        ...,
        description="Email subject",
        max_length=200                    # Length limit
    )

    body: str = Field(
        ...,
        description="Email body, supports plain text or HTML"
    )

    cc: list[str] = Field(
        default=[],                       # Optional fields must have defaults
        description="CC list"
    )

    priority: Literal["low", "normal", "high"] = Field(
        default="normal",
        description="Email priority"
    )
```

**Define Output Schema:**

```python
class SendEmailOutput(BaseModel):
    """Output result"""

    success: bool = Field(
        ...,
        description="Whether email was sent successfully"
    )

    message_id: str | None = Field(
        None,
        description="Message ID when send is successful"
    )

    error: str | None = Field(
        None,
        description="Error message when send fails"
    )

    sent_at: str | None = Field(
        None,
        description="Send time, ISO 8601 format"
    )
```

---

### Step 2: Implement Module

```python
from apcore import Module, Context
from datetime import datetime


class SendEmailModule(Module):
    """
    Send email module

    Send emails via SMTP or API, supports HTML format.
    """

    # Associate Schema
    input_schema = SendEmailInput
    output_schema = SendEmailOutput

    # Optional: Module metadata
    name = "Send Email"
    tags = ["email", "notification"]
    version = "1.0.0"

    def execute(self, inputs: dict, context: Context) -> dict:
        """
        Execute email sending

        Args:
            inputs: Input parameters (already validated)
            context: Call context

        Returns:
            Send result
        """
        # Method 1: Use dict directly
        to = inputs["to"]
        subject = inputs["subject"]

        # Method 2: Convert to Pydantic object (get type hints)
        params = self.input_schema(**inputs)

        try:
            # Execute send
            message_id = self._send_email(
                to=params.to,
                subject=params.subject,
                body=params.body,
                cc=params.cc
            )

            return {
                "success": True,
                "message_id": message_id,
                "error": None,
                "sent_at": datetime.now().isoformat()
            }

        except Exception as e:
            return {
                "success": False,
                "message_id": None,
                "error": str(e),
                "sent_at": None
            }

    def _send_email(self, to: str, subject: str, body: str, cc: list[str]) -> str:
        """Internal method: actual sending logic"""
        # Implement specific sending logic here
        # Can use smtplib, etc.
        return "msg_" + datetime.now().strftime("%Y%m%d%H%M%S")
```

---

### Step 3: Place Files

**Organize directories by functional layers:**

```
extensions/
├── api/                    # API entry layer
│   └── handler/
│       └── user_api.py
│
├── orchestrator/           # Orchestration layer
│   └── workflow/
│       └── user_register.py
│
├── executor/               # Execution layer
│   ├── email/
│   │   ├── send_email.py       → executor.email.send_email
│   │   └── send_template.py    → executor.email.send_template
│   ├── sms/
│   │   └── send_sms.py         → executor.sms.send_sms
│   └── database/
│       └── query.py            → executor.database.query
│
└── common/                 # Common components
    └── util/
        └── validator.py        → common.util.validator
```

**Layer Recommendations:**

| Layer | Responsibility | Examples |
|---|------|------|
| `api` | External request entry | HTTP handler, GraphQL resolver |
| `orchestrator` | Business orchestration, flow control | Registration flow, order processing |
| `executor` | Concrete execution, external calls | Send email, call API, query database |
| `common` | Common utilities | Validators, formatters |

---

### Step 4: Use Module

```python
from apcore import Registry, Executor

# 1. Create Registry and discover modules
registry = Registry(extensions_dir="./extensions")
registry.discover()

# 2. Create Executor
executor = Executor(registry)

# 3. Call module
result = executor.call(
    module_id="executor.email.send_email",
    inputs={
        "to": "user@example.com",
        "subject": "Hello",
        "body": "World"
    }
)

print(result)
# {"success": True, "message_id": "msg_123", ...}
```

---

## Advanced Usage

### Using Context

```python
class SendEmailModule(Module):
    def execute(self, inputs: dict, context: Context) -> dict:
        # Get call chain information
        print(f"Trace ID: {context.trace_id}")
        print(f"Caller: {context.caller_id}")
        print(f"Call Chain: {context.call_chain}")

        # Get identity information (if available)
        if context.identity:
            print(f"Identity: {context.identity.id} ({context.identity.type})")

        # Use shared data
        custom_data = context.data.get("my_data")

        # ... execute logic ...
```

### Calling Other Modules

```python
class UserRegisterModule(Module):
    """User registration module"""

    def execute(self, inputs: dict, context: Context) -> dict:
        # Create user
        user_id = self._create_user(inputs)

        # Call send email module
        email_result = context.executor.call(
            module_id="executor.email.send_email",
            inputs={
                "to": inputs["email"],
                "subject": "Hello",
                "body": "World"
            },
            context=context  # Pass context to maintain call chain
        )

        return {
            "user_id": user_id,
            "email_sent": email_result["success"]
        }
```

### Async Modules

```python
import aiohttp

class SendEmailModule(Module):
    """Send email module with async support"""

    input_schema = SendEmailInput
    output_schema = SendEmailOutput

    # Define a single async execute; the framework auto-detects it
    async def execute(self, inputs: dict, context: Context) -> dict:
        """Async version (preferred)"""
        params = self.input_schema(**inputs)

        async with aiohttp.ClientSession() as session:
            # Send asynchronously
            message_id = await self._send_async(session, params)

        return {
            "success": True,
            "message_id": message_id,
            "error": None
        }
```

### Resource Management

```python
class DatabaseModule(Module):
    """Database module that manages connections"""

    _pool: Any = None  # Connection pool

    def on_load(self) -> None:
        """Create connection pool when module loads"""
        self._pool = create_connection_pool(
            host="localhost",
            database="mydb"
        )

    def on_unload(self) -> None:
        """Close connection pool when module unloads"""
        if self._pool:
            self._pool.close()

    def execute(self, inputs: dict, context: Context) -> dict:
        # Use connection pool
        with self._pool.get_connection() as conn:
            result = conn.execute(inputs["sql"])
            return {"rows": result}
```

---

## Common Patterns

### Pattern 1: Simple Executor

```python
class CalculatorModule(Module):
    """Simple calculator - no side effects"""

    class Input(BaseModel):
        a: float = Field(..., description="First number")
        b: float = Field(..., description="Second number")
        op: Literal["+", "-", "*", "/"] = Field(..., description="Operator")

    class Output(BaseModel):
        result: float = Field(..., description="Calculation result")

    input_schema = Input
    output_schema = Output

    def execute(self, inputs: dict, context: Context) -> dict:
        a, b, op = inputs["a"], inputs["b"], inputs["op"]
        ops = {"+": a + b, "-": a - b, "*": a * b, "/": a / b}
        return {"result": ops[op]}
```

### Pattern 2: External API Call

```python
class WeatherModule(Module):
    """Get weather information - calls external API"""

    class Input(BaseModel):
        city: str = Field(..., description="City name")

    class Output(BaseModel):
        temperature: float = Field(..., description="Temperature (Celsius)")
        description: str = Field(..., description="Weather description")

    input_schema = Input
    output_schema = Output

    def execute(self, inputs: dict, context: Context) -> dict:
        import requests

        response = requests.get(
            "https://api.weather.com/v1/current",
            params={"city": inputs["city"]}
        )
        data = response.json()

        return {
            "temperature": data["temp"],
            "description": data["desc"]
        }
```

### Pattern 3: Data Validator

```python
class EmailValidatorModule(Module):
    """Email format validator"""

    class Input(BaseModel):
        email: str = Field(..., description="Email to validate")

    class Output(BaseModel):
        valid: bool = Field(..., description="Whether valid")
        reason: str | None = Field(None, description="Reason if invalid")

    input_schema = Input
    output_schema = Output

    def execute(self, inputs: dict, context: Context) -> dict:
        import re

        email = inputs["email"]
        pattern = r"^[\w\.-]+@[\w\.-]+\.\w+$"

        if re.match(pattern, email):
            return {"valid": True, "reason": None}
        else:
            return {"valid": False, "reason": "Invalid email format"}
```

### Pattern 4: Executor with Retry

```python
from tenacity import retry, stop_after_attempt, wait_exponential

class ReliableSendModule(Module):
    """Reliable send with retry"""

    input_schema = SendEmailInput
    output_schema = SendEmailOutput

    def execute(self, inputs: dict, context: Context) -> dict:
        try:
            return self._execute_with_retry(inputs)
        except Exception as e:
            return {"success": False, "message_id": None, "error": str(e)}

    @retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
    def _execute_with_retry(self, inputs: dict) -> dict:
        # This method will automatically retry
        message_id = self._send(inputs)
        return {"success": True, "message_id": message_id, "error": None}
```

---

## Best Practices

### 1. Schema Design

```python
# ✅ Good design: fields have descriptions and constraints
class GoodInput(BaseModel):
    email: str = Field(..., description="User email", pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    age: int = Field(..., description="User age", ge=0, le=150)

# ❌ Bad design: missing descriptions and constraints
class BadInput(BaseModel):
    email: str
    age: int
```

### 2. Error Handling

```python
# ✅ Good practice: return structured errors
def execute(self, inputs: dict, context: Context) -> dict:
    try:
        result = self._do_work(inputs)
        return {"success": True, "data": result, "error": None}
    except ValidationError as e:
        return {"success": False, "data": None, "error": f"Parameter error: {e}"}
    except ExternalAPIError as e:
        return {"success": False, "data": None, "error": f"External service error: {e}"}

# ❌ Bad practice: throw exceptions directly
def execute(self, inputs: dict, context: Context) -> dict:
    return self._do_work(inputs)  # Exception propagates up
```

### 3. Single Responsibility

```python
# ✅ Good design: each module does one thing
class SendEmailModule(Module): ...      # Only sends email
class ValidateEmailModule(Module): ...  # Only validates email
class RenderTemplateModule(Module): ... # Only renders template

# ❌ Bad design: one module does too much
class EmailModule(Module):
    def execute(self, inputs: dict, context: Context) -> dict:
        # Validation, rendering, sending all together
        self._validate(inputs)
        html = self._render(inputs)
        return self._send(html)
```

---

## Testing Guide

### Basic Module Testing

```python
# test_send_email.py
import pytest
from unittest.mock import MagicMock
from apcore import Context, Identity

def create_test_context(**kwargs):
    """Create test Context"""
    return Context(
        trace_id="test-trace-id",
        caller_id=kwargs.get("caller_id"),
        call_chain=kwargs.get("call_chain", []),
        executor=kwargs.get("executor", MagicMock()),
        identity=kwargs.get("identity", Identity(id="test", type="user")),
        data=kwargs.get("data", {})
    )

class TestSendEmailModule:
    def setup_method(self):
        self.module = SendEmailModule()
        self.context = create_test_context()

    def test_successful_send(self):
        """Test successful send"""
        result = self.module.execute(
            inputs={
                "to": "user@example.com",
                "subject": "Test",
                "body": "Hello"
            },
            context=self.context
        )
        assert result["success"] is True

    def test_invalid_input(self):
        """Test invalid input"""
        with pytest.raises(Exception):
            self.module.execute(
                inputs={"to": "", "subject": ""},
                context=self.context
            )

    def test_calls_other_module(self):
        """Test inter-module calls (Mock Executor)"""
        mock_executor = MagicMock()
        mock_executor.call.return_value = {"result": "ok"}
        context = create_test_context(executor=mock_executor)

        result = self.module.execute(
            inputs={...},
            context=context
        )

        mock_executor.call.assert_called_once_with(
            module_id="target.module",
            inputs={...},
            context=context
        )
```

### Debugging Tips

1. **Use trace_id to track call chain**: Search for `trace_id` in logs to trace complete call path
2. **Check call_chain**: `context.call_chain` shows the complete path of current call
3. **Pre-validate Schema**: Use `executor.validate()` to check if inputs are valid before execution
4. **Middleware debugging**: Add `LoggingMiddleware` to view inputs/outputs of each call

### Performance Guide

| Recommendation | Explanation |
|------|------|
| Reuse connections | Create connection pool in `on_load()`, close in `on_unload()` |
| Avoid blocking | Long operations **should** use `execute_async()` |
| Control data size | `context.data` is shared along call chain, avoid storing large amounts of data |
| Set timeouts | External calls **must** set reasonable timeout |
| Idempotent design | Modules marked as `idempotent=True` should ensure repeated calls are safe |

---

## module() Registration

> For existing functions or methods, wrap them as standard apcore modules using the `@module` decorator or `module()` function call. See [PROTOCOL_SPEC §5.11](../../PROTOCOL_SPEC.md) for detailed specification.

### @module Decorator (Simple Example)

**Before (regular function):**

```python
def send_email(to: str, subject: str, body: str) -> dict:
    """Send email"""
    # Business logic...
    return {"success": True, "message_id": "msg_123"}
```

**After (apcore module):**

```python
from apcore import module

@module(id="email.send", tags=["email", "notification"])
def send_email(to: str, subject: str, body: str) -> dict:
    """Send email"""
    # Business logic completely unchanged
    return {"success": True, "message_id": "msg_123"}
```

Add one line of `@module` decorator, and the function automatically becomes an apcore module:
- Schema is auto-generated from type annotations
- Description is auto-extracted from docstring
- Module is auto-registered to Registry

### module() Function Call (Register existing class methods)

```python
from apcore import module

# Existing business code (no modifications)
class EmailService:
    def send(self, to: str, subject: str, body: str) -> dict:
        """Send email"""
        return {"success": True}

    def send_template(self, template_id: str, data: dict) -> dict:
        """Send using template"""
        return {"success": True}

# Register via module() without changing original code
service = EmailService()
module(service.send, id="email.send")
module(service.send_template, id="email.send_template")
```

### Advanced Example (Annotated + async)

```python
from apcore import module, Context, ModuleAnnotations
from typing import Annotated
from pydantic import Field

@module(
    id="email.send",
    annotations=ModuleAnnotations(open_world=True, idempotent=False),
    tags=["email"]
)
async def send_email(
    to: Annotated[str, Field(description="Recipient email", pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$")],
    subject: Annotated[str, Field(description="Email subject", max_length=200)],
    body: Annotated[str, Field(description="Email body")],
    cc: Annotated[list[str], Field(description="CC list")] = [],
    context: Context = None
) -> dict:
    """
    Send email module

    Send emails asynchronously via SMTP.
    """
    # async def automatically maps to execute_async
    print(f"trace_id: {context.trace_id}")
    return {"success": True, "message_id": "msg_123"}
```

### ID Generation Rules

- When `id` parameter is specified: use it directly
- When not specified: auto-generate from function's `__module__` + `__qualname__`

### Description Extraction

1. `description` parameter (highest priority)
2. First line of function docstring
3. Default description generated from function name

### Limitations

| Feature | Class-based | module() |
|------|------------|----------|
| Lifecycle hooks (on_load/on_unload) | Supported | Not supported |
| Custom validate() | Supported | Not supported |
| Schema source | Pydantic Model | Auto-generated from type annotations |
| Execution context | `self` + `context` | `context` parameter injection |

---

## External Schema Binding (YAML)

> For scenarios where you cannot modify existing source code at all, use YAML binding files to map functions to apcore modules. See [PROTOCOL_SPEC §5.12](../../PROTOCOL_SPEC.md) for detailed specification.

### Complete Binding File Example

```yaml
# bindings/email.binding.yaml
bindings:
  - module_id: "email.send"
    target: "myapp.services.email:send_email"
    description: "Send email"
    tags: ["email", "notification"]
    annotations:
      open_world: true
      idempotent: false
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
      required: [success]

  - module_id: "email.send_template"
    target: "myapp.services.email:EmailService.send_template"
    description: "Send email using template"
    auto_schema: true
```

### auto_schema Mode

When the target function has complete type annotations, you can use `auto_schema: true` to auto-generate Schema:

```yaml
bindings:
  - module_id: "email.send"
    target: "myapp.services.email:send_email"
    auto_schema: true    # Auto-generate from send_email's type annotations
```

Equivalent to `module(send_email, id="email.send")`, but requires no source code modifications.

### Discovery Mechanism Configuration

```yaml
# apcore.yaml
bindings:
  dir: "./bindings"              # Scan directory (default)
  pattern: "*.binding.yaml"      # File matching pattern
  # Or specify file list
  files:
    - "./bindings/email.binding.yaml"
    - "./bindings/payment.binding.yaml"
```

### Multiple Binding Files Management

```
my-project/
├── bindings/
│   ├── email.binding.yaml       # Email-related modules
│   ├── payment.binding.yaml     # Payment-related modules
│   └── user.binding.yaml        # User-related modules
└── apcore.yaml
```

Each binding file is organized by business domain. The framework automatically scans all `*.binding.yaml` files in the `bindings/` directory.

---

## Approach Selection Comparison

| Consideration | Class-based | `@module` Decorator | `module()` Function Call | External Binding |
|------|------------|-----------------|-------------------|-----------------|
| **New development** | Recommended | Usable | Usable | Not recommended |
| **Wrap existing functions** | Not recommended (requires rewrite) | Recommended | Recommended | Usable |
| **Cannot modify source** | Impossible | Impossible | Impossible | Recommended |
| **Need lifecycle management** | Recommended | Not supported | Not supported | Not supported |
| **Cross-language unified config** | Not applicable | Not applicable | Partially applicable | Recommended |
| **Schema flexibility** | Highest (Pydantic) | Medium (type annotations) | Medium (type annotations) | High (hand-written YAML) |

---

## Next Steps

- [Schema Definition Details](./schema-definition.md) - Complete Schema usage
- [ACL Configuration Guide](./acl-configuration.md) - Configure module access permissions
- [Module Interface Definition](../api/module-interface.md) - API reference
- [Adapter Development Guide](./adapter-development.md) - Framework adapter development
