# Approval System

## Overview

The Approval System provides runtime enforcement of the `requires_approval` annotation. When a module declares `requires_approval=true` and an `ApprovalHandler` is configured on the Executor, the handler is invoked at **Step 4.5** of the execution pipeline — after ACL checks pass and before input validation begins. This allows human or automated review of sensitive operations before they execute.

The Approval System is architecturally separate from the ACL System. ACL answers "who is allowed to call this module?" while Approval answers "does this particular invocation need sign-off before proceeding?"

## Requirements

- Provide a pluggable `ApprovalHandler` protocol that SDK implementations can satisfy with custom logic.
- Enforce the approval gate at Executor Step 4.5, after ACL (Step 4) and before Input Validation (Step 5).
- Skip the approval gate entirely when no `ApprovalHandler` is configured, or when the module does not declare `requires_approval=true`.
- Support synchronous approval flows (Phase A) where `request_approval()` blocks until a decision is returned.
- Optionally support asynchronous approval flows (Phase B) where a `pending` status is returned with an `approval_id`, and execution resumes when the client retries with an `_approval_token`.
- Raise structured errors (`APPROVAL_DENIED`, `APPROVAL_TIMEOUT`, `APPROVAL_PENDING`) that map cleanly to protocol error codes.
- Ship built-in handlers for common cases: `AlwaysDenyHandler` (safe default), `AutoApproveHandler` (testing), and `CallbackApprovalHandler` (custom function).

## Technical Design

### Approval Gate (Executor Step 4.5)

The approval gate is inserted between ACL Enforcement (Step 4) and Input Validation (Step 5) in the Executor's pipeline. The algorithm:

1. Check if `approval_handler` is configured on the Executor.
2. If not configured, skip to Step 5.
3. Check if the target module declares `requires_approval=true` in its annotations.
4. If not, skip to Step 5.
5. Build an `ApprovalRequest` containing `module_id`, `arguments`, `caller` identity, and the module's `annotations`.
6. Call `approval_handler.request_approval(request)`.
7. If `approved` → proceed to Step 5.
8. If `denied` → raise `ApprovalDeniedError` with the handler's reason.
9. If `timeout` → raise `ApprovalTimeoutError`.
10. If `pending` (Phase B only) → raise `ApprovalPendingError` with `approval_id`.

### ApprovalHandler Protocol

```python
class ApprovalHandler(Protocol):
    async def request_approval(self, request: ApprovalRequest) -> ApprovalResult:
        """Request approval for a module invocation. Returns the decision."""
        ...
```

Implementations receive an `ApprovalRequest` and return an `ApprovalResult`. The handler may block (waiting for human input via UI, Slack, etc.) or return immediately (auto-approve for testing).

### Data Types

**ApprovalRequest** carries the invocation context to the handler:

| Field | Type | Description |
|-------|------|-------------|
| `module_id` | `str` | Canonical module ID |
| `arguments` | `dict` | Input arguments for the call |
| `caller` | `Identity \| None` | Caller identity from context |
| `annotations` | `ModuleAnnotations` | Full annotation set of the module |

**ApprovalResult** carries the handler's decision:

| Field | Type | Description |
|-------|------|-------------|
| `status` | `str` | One of: `approved`, `denied`, `timeout`, `pending` |
| `reason` | `str \| None` | Human-readable explanation |
| `approval_id` | `str \| None` | Phase B: token for async resume |

### Error Types

| Error | Code | HTTP | When |
|-------|------|------|------|
| `ApprovalDeniedError` | `APPROVAL_DENIED` | 403 | Handler returns `denied` |
| `ApprovalTimeoutError` | `APPROVAL_TIMEOUT` | 408 | Handler returns `timeout` |
| `ApprovalPendingError` | `APPROVAL_PENDING` | 202 | Handler returns `pending` (Phase B) |

All three extend the base `ApprovalError`, which extends `ModuleError`.

### Built-in Handlers

| Handler | Behavior | Use Case |
|---------|----------|----------|
| `AlwaysDenyHandler` | Always returns `denied` | Safe default when approval is required but no handler is configured |
| `AutoApproveHandler` | Always returns `approved` | Testing and development |
| `CallbackApprovalHandler` | Delegates to a user-provided callback function | Custom approval logic |

### Protocol Bridge Handlers

Protocol bridges (apcore-mcp, apcore-a2a) provide their own `ApprovalHandler` implementations that use protocol-native mechanisms:

- **`ElicitationApprovalHandler`** (apcore-mcp) — Uses the MCP elicitation protocol to present an approval prompt to the AI client, which relays it to the human user.
- **`A2AApprovalHandler`** (apcore-a2a, future) — Uses the A2A interaction protocol.
- **`WebhookApprovalHandler`** — Sends approval requests to an HTTP endpoint and waits for a callback.

### Phased Implementation

| Phase | Scope | Requirement |
|-------|-------|-------------|
| **Phase A** | Synchronous approval: handler blocks until decision | **MUST** implement for conformance |
| **Phase B** | Asynchronous approval: `pending` + `approval_id` + retry with `_approval_token` | **MAY** implement |

## Key Files

| File | Purpose |
|------|---------|
| `approval.py` | `ApprovalHandler` protocol, `ApprovalRequest`, `ApprovalResult`, built-in handlers |
| `errors.py` | `ApprovalError`, `ApprovalDeniedError`, `ApprovalTimeoutError`, `ApprovalPendingError` |
| `executor.py` | Step 4.5 integration in `call()`, `call_async()`, `stream()` |

## Dependencies

### Internal
- **Executor** — The approval gate is embedded in the Executor pipeline at Step 4.5.
- **Module Annotations** — The `requires_approval` field on `ModuleAnnotations` triggers the gate.
- **Context / Identity** — Caller identity is passed to the handler for access-aware approval decisions.

## Testing Strategy

- **Unit tests** verify that the approval gate fires only when both an `ApprovalHandler` is configured and the module declares `requires_approval=true`.
- **Handler tests** confirm each built-in handler returns the expected `ApprovalResult` status.
- **Error mapping tests** verify that `denied` → `ApprovalDeniedError`, `timeout` → `ApprovalTimeoutError`, `pending` → `ApprovalPendingError`.
- **Skip tests** confirm the gate is skipped when no handler is set, or when the module does not require approval.
- **Integration tests** run a full pipeline execution with approval handlers to verify end-to-end behavior including error propagation to callers.
- Test naming follows the `test_<unit>_<behavior>` convention.
