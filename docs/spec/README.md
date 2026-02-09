# Framework Specification

> The official technical specification of apcore, intended for SDK implementers.

This directory contains the official specification documents for the apcore framework. These documents define the algorithms, type mappings, and conformance requirements that all language SDK implementations must follow. The specification documents use RFC 2119 keywords (MUST, SHOULD, MAY, etc.) to distinguish the level of obligation for each requirement.

## Document List

| Document | Description |
|----------|-------------|
| [algorithms.md](./algorithms.md) | Core algorithm reference, a unified index summarizing 17+ pseudocode algorithms |
| [type-mapping.md](./type-mapping.md) | Cross-language type mapping, defining the standard mapping from JSON Schema types to native types in each language |
| [conformance.md](./conformance.md) | Conformance definitions, specifying implementation conformance levels, test suite requirements, and declaration specifications |

## Recommended Reading Order

1. **Conformance Definitions** -- First understand the conformance levels and clarify implementation goals
2. **Type Mapping** -- Master the type correspondences for the target language
3. **Algorithm Reference** -- Implement core algorithms one by one

If you are a module developer rather than an SDK implementer, these documents are for reference only. For day-to-day development, please refer to the [Usage Guides](../guides/).
