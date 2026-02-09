# Creating Modules Guide

> Create an apcore module from scratch.

## Four Module Definition Methods

apcore supports four ways to create modules, choose the one that suits your scenario:

| Method | Use Case | Code Invasiveness | Jump |
|------|---------|-----------|------|
| **Class-based** (class definition) | New module development | High (inherit base class) | [Quick Start](#quick-start) |
| **`@module` decorator** | Functions where you can modify source | Low (add one line decorator) | [module() Registration](#module-registration) |
| **`module()` function call** | Wrapping existing classes/methods | Very low (no changes to original function) | [module() Registration](#module-registration) |
| **External Binding** | Zero-modification integration of existing apps | Zero (no source changes) | [External Schema Binding](#external-schema-binding-yaml) |

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
└── schemas/              # Schema definitions (optional, for cross-language use)
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
    message_id: str | None = Field(None, description="Email ID")


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

### 3. Module ID Auto-generation

```
File path: extensions/executor/email/send_email.py
Module ID:  executor.email.send_email
```

**No configuration needed, the file path is the ID.**

---

This document is a concise English summary. For the complete guide and full examples, see `creating-modules.md`.

---

## Next Steps

- [Schema Definition Details](./schema-definition.md) - Complete Schema usage
- [ACL Configuration Guide](./acl-configuration.md) - Configure module access permissions
- [Module Interface Definition](../api/module-interface.md) - API reference
- [Adapter Development Guide](./adapter-development.md) - Framework adapter development
