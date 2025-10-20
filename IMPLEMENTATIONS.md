# IMPLEMENTATIONS.md — Tools Following Strict JSON

This page tracks tools, libraries, and frameworks that implement the Strict JSON Manifesto rules.

## Table of Contents

- [Status](#status)
- [Implementation Criteria](#implementation-criteria)
- [Verification Checklist](#verification-checklist)
- [By Language](#by-language)
- [How to Submit](#how-to-submit)
- [Template](#template)
- [Resources](#resources)

---

## Status

**Currently seeking community implementations.**

We're building a list of tools that follow Strict JSON principles. If you're developing one, we want to hear from you.

- **Official Implementations**: Tools developed by manifesto maintainers
- **Community Implementations**: Tools built by community members
- **In Progress**: Tools under active development

---

## Implementation Criteria

Any tool claiming to follow the **Strict JSON Manifesto** MUST:

### Core Requirements

- **Enforce all 16 rules** — No selective enforcement
- **Generate code (not reflection)** — Compile-time when possible, runtime when necessary
- **Fail fast with clear errors** — Include field name, line/column, expected vs actual
- **Support compact JSON only** — No beautified output in production
- **Have documentation** — Rules followed, usage examples, limitations
- **Type safety** — Make illegal states unrepresentable
- **No polymorphic typing** — No `Object` type, no arbitrary class instantiation
- **Input validation** — Enforce size limits (10MB JSON, 10k array, 1MB string, 10 nesting levels)

### Quality Requirements

- **Well-tested** — Unit tests demonstrate rule enforcement
- **Error handling** — Clear, actionable error messages with suggestions
- **Performance** — Minimal allocations, fast parsing
- **Language-appropriate** — Follows language conventions and idioms

### Documentation Requirements

- **README** — Quick start, basic usage, which rules enforced
- **Rules enforcement** — Document how each rule is validated
- **Examples** — Working code examples for common use cases
- **Limitations** — Known limitations or language-specific behaviors

---

## Verification Checklist

Before claiming "Strict JSON Compliant," verify:

**Rule Enforcement:**
- [ ] Rule 1: Numeric values (unquoted, wrapper types)
- [ ] Rule 2: Boolean values (unquoted true/false only)
- [ ] Rule 3: String handling (exact preservation)
- [ ] Rule 4: Array handling (explicit declaration, no auto-wrap)
- [ ] Rule 5: Null values (forbidden in JSON)
- [ ] Rule 6: Date/Time (ISO 8601 UTC only)
- [ ] Rule 7: Field mapping (exact or single custom)
- [ ] Rule 8: Type system (supported/unsupported types)
- [ ] Rule 9: Nested objects (follow same rules)
- [ ] Rule 10: Error handling (fail fast, clear messages)
- [ ] Rule 11: Nesting depth (max 10 levels)
- [ ] Rule 12: Circular references (prevented)
- [ ] Rule 13: Security (no polymorphic typing, input limits)
- [ ] Rule 14: Performance (generated code, minimal allocations)
- [ ] Rule 15: JSON compliance (RFC 8259)
- [ ] Rule 16: Transmission format (compact only)

**Code Quality:**
- [ ] No reflection (generated or deterministic code)
- [ ] Input validation with configurable limits
- [ ] Clear error messages with suggestions
- [ ] Handles all supported types correctly
- [ ] Handles edge cases (empty arrays, null fields, max limits)

**Documentation:**
- [ ] README with quick start
- [ ] Rules verification list
- [ ] Code examples
- [ ] Error handling guide

---

## By Language

### Java

#### In Progress / Help Wanted

- **Annotation Processor** — Compile-time code generation
  - Status: Planned
  - Rules: All 16
  - Help: Seeking maintainer

- **Spring Boot Starter** — Ready-to-use Spring Boot integration
  - Status: Planned
  - Features: Auto-configuration, exception handling
  - Help: Seeking maintainer

#### Reference Implementation

- [Strict JSON Java Processor](https://github.com/strict-json/processor-java) (coming soon)

### TypeScript / JavaScript

#### In Progress / Help Wanted

- **Zod Integration** — Runtime validation with Zod schemas
  - Status: Planned
  - Features: Type inference, error messages
  - Help: Seeking maintainer

- **Runtime Validator** — Standalone validator for browser/Node.js
  - Status: Planned
  - Features: No dependencies, compact
  - Help: Seeking maintainer

#### Reference Implementation

- [Strict JSON TypeScript](https://github.com/strict-json/typescript) (coming soon)

### Kotlin

#### In Progress / Help Wanted

- **Code Generator** — Annotation processor for Kotlin
  - Status: Planned
  - Features: Data class generation, sealed classes
  - Help: Seeking maintainer

- **Data Class Helper** — Extensions for Kotlin data classes
  - Status: Planned
  - Features: Validation DSL, error handling
  - Help: Seeking maintainer

#### Reference Implementation

- [Strict JSON Kotlin](https://github.com/strict-json/kotlin) (coming soon)

### Go

#### In Progress / Help Wanted

- **JSON Unmarshaller** — Custom JSON unmarshalling
  - Status: Planned
  - Features: Struct tags, validation hooks
  - Help: Seeking maintainer

#### Reference Implementation

- [Strict JSON Go](https://github.com/strict-json/go) (coming soon)

### Python

#### In Progress / Help Wanted

- **Pydantic Integration** — Strict JSON validation with Pydantic
  - Status: Planned
  - Features: Type hints, validators
  - Help: Seeking maintainer

- **Dataclass Decorator** — Python dataclass enhancement
  - Status: Planned
  - Features: Validation, error handling
  - Help: Seeking maintainer

#### Reference Implementation

- [Strict JSON Python](https://github.com/strict-json/python) (coming soon)

### .NET / C#

#### In Progress / Help Wanted

- **System.Text.Json Extensions** — Integration with .NET's JSON library
  - Status: Planned
  - Features: Custom converters, validation
  - Help: Seeking maintainer

- **Newtonsoft.Json (Json.NET) Integration**
  - Status: Planned
  - Features: Custom serialization settings
  - Help: Seeking maintainer

#### Reference Implementation

- [Strict JSON .NET](https://github.com/strict-json/dotnet) (coming soon)

---

## How to Submit

### Step 1: Verify Compliance

Before submitting, verify your tool:

1. Review [Verification Checklist](#verification-checklist)
2. Run test suite covering all 16 rules
3. Document which rules are enforced
4. List any language-specific variations or limitations

### Step 2: Prepare Documentation

Create documentation with:
- **README.md** — Quick start, overview
- **RULES_ENFORCED.md** — How each rule is validated
- **EXAMPLES.md** — Working code samples
- **LIMITATIONS.md** — Known gaps or language-specific behaviors

### Step 3: Open a Pull Request

Add your tool to the appropriate language section:

```markdown
#### [Your Tool Name]

- **Status**: Active / Maintained / Experimental
- **Repository**: https://github.com/...
- **Rules Enforced**: All 16 / List specific rules
- **Features**:
  - Feature 1
  - Feature 2
- **Language Versions**: Java 8+, JDK 17+, etc.
- **Maintainer**: @github-username or organization
- **Documentation**: Link to docs
```

---

## Template

Use this template for your implementation submission:

```markdown
### [Tool Name]

**Status**: [Active / Maintained / Experimental]

**Repository**: [Link]

**Description**: 1-2 sentence description

**Rules Enforced**:
- [x] All 16 rules
- [ ] Specific rules (list if not all)

**Language & Versions**:
- Language: [Java / TypeScript / Go / etc.]
- Min version: [e.g., Java 8, TS 4.5+]
- Dependencies: [None / Minimal / List]

**Key Features**:
- Feature 1
- Feature 2
- Feature 3

**Installation**:
```bash
# Installation command
```

**Quick Example**:
```java/typescript/python
// Quick usage example
```

**Documentation**: [Link to docs]

**Maintainer**: [@username](https://github.com/username) or [Organization](https://github.com/org)

**Tests**: [Link to test suite] | Coverage: X%

**Known Limitations**:
- Limitation 1
- Limitation 2
```

---

## Resources

### For Implementers

- [RULES.md](./RULES.md) — Complete rule specifications
- [Language-Agnostic Principles](./RULES.md#language-agnostic-principles) — Multi-language guidance
- [CONTRIBUTING.md](./CONTRIBUTING.md) — How to propose changes
- [README.md](./README.md) — Manifesto overview

### Reference Implementations (Coming Soon)

- Java reference implementation
- TypeScript reference implementation
- Go reference implementation
- Python reference implementation
- .NET reference implementation

### Related Standards & Tools

- [JSON Schema](https://json-schema.org/) — Data validation
- [RFC 8259](https://tools.ietf.org/html/rfc8259) — JSON standard
- [Zod](https://zod.dev/) — TypeScript-first schema validation
- [Pydantic](https://pydantic-docs.helpmanual.io/) — Python data validation

---

## Support

### Questions?

- **How to implement Rule X in language Y?** → Open issue: "Implementation help: Rule X in Go"
- **How do I submit my tool?** → Follow [How to Submit](#how-to-submit)
- **Tool doesn't follow all rules** → That's OK! Document which rules you support
- **Want to collaborate on reference implementation?** → Open issue: "Help wanted: Reference implementation for [language]"

### Call to Action

We're seeking maintainers for reference implementations in:
- ✓ Java
- ✓ TypeScript
- ✓ Go
- ✓ Python
- ✓ .NET / C#
- ✓ Kotlin
- ✓ Rust (future)
- ✓ Ruby (future)

**Interested?** Open an issue or reach out.

---

## In the Wild

_Coming soon: Companies and projects using Strict JSON implementations_

If your organization is using a Strict JSON tool, add yourself here via PR.

---

**Help us build this ecosystem. Your implementation matters.**
