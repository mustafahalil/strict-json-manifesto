# Strict JSON Manifesto — Detailed Rules

This document specifies the exact rules that implementations following the Strict JSON Manifesto must enforce. These are non-negotiable principles for any tool or library claiming to follow this manifesto.

**Table of Contents**
- [Language-Agnostic Principles](#language-agnostic-principles)
- [Rule 1: Numeric Values](#rule-1-numeric-values)
- [Rule 2: Boolean Values](#rule-2-boolean-values)
- [Rule 3: String Handling](#rule-3-string-handling)
- [Rule 4: Array Handling](#rule-4-array-handling)
- [Rule 5: Null Values](#rule-5-null-values)
- [Rule 6: Date/Time Handling](#rule-6-datetime-handling)
- [Rule 7: Field Mapping](#rule-7-field-mapping)
- [Rule 8: Type System](#rule-8-type-system)
- [Rule 9: Nested Objects](#rule-9-nested-objects)
- [Rule 10: Error Handling](#rule-10-error-handling)
- [Rule 11: Nesting Depth Limits](#rule-11-nesting-depth-limits)
- [Rule 12: Circular References](#rule-12-circular-references)
- [Rule 13: Security](#rule-13-security)
- [Rule 14: Performance](#rule-14-performance)
- [Rule 15: JSON Compliance](#rule-15-json-compliance)
- [Rule 16: Data Transmission Format](#rule-16-data-transmission-format)

---

## Language-Agnostic Principles

These rules apply to **any programming language**. The principles are universal and language-independent. Examples in this document use **Java syntax** for consistency, but the same rules apply to TypeScript, Go, C#, Python, Kotlin, and all other languages.

### Key Concept: WHAT vs HOW

Each rule defines **WHAT must be done** (universal principle).  
**HOW to do it** depends on language-specific capabilities and idioms.

### Multi-Language Examples

**Rule 1: Numeric Values (Wrapper Types)**

| Principle | Java | TypeScript | Go | Python | .NET |
|-----------|------|------------|----|---------|----|
| Unquoted numbers | `Integer`, `Long`, `Double` | `number`, `bigint` | `int64`, `float64` | `int`, `float` | `int`, `long`, `decimal` |
| No scientific notation | Enforce in parser | Enforce in parser | Enforce in parser | Enforce in parser | Enforce in parser |
| Wrapper types | `Integer` not `int` | Number type | Interface, not primitive | Type hints | Nullable types |

**Rule 2: Boolean Values (Nullable/Wrapper)**

| Language | Correct | Wrong |
|----------|---------|-------|
| Java | `Boolean active;` | `boolean active;` |
| TypeScript | `active: boolean \| null;` | `active: boolean;` |
| Go | `*bool` or `optional` | `bool` (no null) |
| Python | `active: bool \| None` | `active: bool` |
| .NET | `bool?` | `bool` |

**Rule 4: Array Handling (Explicit Declaration)**

| Language | Correct | Wrong |
|----------|---------|-------|
| Java | `List<String> tags;` | `String[] tags;` or `ArrayList` |
| TypeScript | `tags: string[];` or `tags: Array<string>;` | `tags: string \| string[];` |
| Go | `tags []string` | `tags interface{}` |
| Python | `tags: List[str]` | `tags: list` (untyped) |
| .NET | `List<string> tags;` | `object[] tags;` |

**Rule 6: Date/Time Handling (ISO 8601 UTC Only)**

| Language | Correct | Wrong |
|----------|---------|-------|
| Java | `Instant timestamp;` | `Date` or `LocalDateTime` |
| TypeScript | `timestamp: Date;` with ISO input | Custom date parsing |
| Go | `time.Time` with UTC | Unix timestamp |
| Python | `datetime.datetime` with UTC | Milliseconds or string formats |
| .NET | `DateTimeOffset` with UTC | `DateTime` without offset |

### Implementation Pattern

1. **Parse Rule**: Define validation logic (universal)
2. **Type Mapping**: Choose language-appropriate type
3. **Enforce**: Add compile-time or runtime checks
4. **Error**: Fail fast with clear messages

### Example: Implementing Rule 1 (Numeric Values)
```
UNIVERSAL RULE: Numbers must be unquoted, use wrapper types

Java Implementation:
  ✓ private Integer age;     // Wrapper type
  ✗ private int age;         // Primitive rejected
  ✗ "age": "25"              // Quoted number rejected

TypeScript Implementation:
  ✓ age: number;             // Type-safe number
  ✓ age?: number;            // Optional allowed
  ✗ age: string;             // String rejected
  ✗ age: any;                // Any type rejected

Go Implementation:
  ✓ Age *int64              // Pointer = nullable
  ✓ Age int64               // Non-null if required
  ✗ Age interface{}         // Dynamic type rejected

Python Implementation:
  ✓ age: int | None         // Union with None
  ✓ age: Optional[int]      // Type hints
  ✗ age: Any                // Any type rejected
```

### When Rules Conflict with Language Features

**Principle**: Follow the Strict JSON manifesto spirit, adapt to language idioms.

- **Go has no null** → Use pointers for nullable fields
- **Python is dynamically typed** → Use type hints + runtime validation
- **TypeScript has any type** → Avoid it, use strict union types
- **JavaScript lacks static types** → Use runtime validators (Zod, JSON Schema)

### Reference for Implementers

When building a tool in your language:

1. Read the rule (language-independent principle)
2. Map to your language's type system
3. Implement validation at appropriate layer (compile-time or runtime)
4. Fail fast when rules violated
5. Provide clear error messages

---
---

## Rule 1: Numeric Values

### 1.1 JSON Format

**MUST:** Numbers in JSON MUST be unquoted  
**MUST NOT:** Quoted numbers are forbidden

```json
✓ VALID
{
  "age": 25,
  "price": 150.00,
  "balance": 1234.56,
  "bigNumber": 999999999999999
}

✗ INVALID
{
  "age": "25",           // ERROR: quoted number
  "price": "150.00",     // ERROR: quoted number
  "invalid": "abc"       // ERROR: not a number
}
```

### 1.2 Type Selection

**MUST:** Use appropriate numeric types based on value

| JSON Value | Recommended Type | Language Examples |
|------------|------------------|-------------------|
| 123 | 32-bit integer | Java: `Integer`, Kotlin: `Int`, TypeScript: `number` |
| 2147483648 | 64-bit long | Java: `Long`, Kotlin: `Long`, TypeScript: `bigint` |
| 123.45 | Floating point | Java: `Double`, Kotlin: `Double`, TypeScript: `number` |
| Very large decimals | Arbitrary precision | Java: `BigDecimal`, Kotlin: `BigDecimal` |

**MUST:** Use wrapper/nullable types, not primitives

```java
// ✓ CORRECT
private Integer age;
private Double price;
private Long id;

// ✗ WRONG
private int age;           // Primitive - not allowed
private double price;      // Primitive - not allowed
```

### 1.3 No Scientific Notation

**MUST NOT:** Scientific notation in JSON  
**FORBIDDEN:** Numbers like `1.23e10`, `5E-3`, `2.5e+2`

```json
✓ VALID
{
  "small": 0.0000123,
  "large": 123000000000,
  "precise": 123.456789
}

✗ INVALID
{
  "small": 1.23e-5,      // ERROR: scientific notation
  "large": 1.23e10,      // ERROR: scientific notation
  "number": 2.5E+2,      // ERROR: scientific notation
  "value": 5e3           // ERROR: scientific notation
}
```

**Why?** Scientific notation adds ambiguity and parsing complexity. Use explicit decimal representation instead.

### 1.4 String Representation Preservation

**MUST:** For numeric fields used in specific contexts (IDs, codes), preserve string representation

```json
{
  "customerId": 123456,
  "productCode": "00123"
}
```

Applications should provide access to both numeric and string representations where needed for business logic.

---

## Rule 2: Boolean Values

### 2.1 JSON Format

**MUST:** Booleans in JSON MUST be unquoted `true` or `false`  
**MUST NOT:** Quoted booleans, numeric representations, or string values

```json
✓ VALID
{
  "active": true,
  "enabled": false,
  "isAdmin": true
}

✗ INVALID
{
  "active": "true",       // ERROR: quoted boolean
  "enabled": 1,           // ERROR: numeric not allowed
  "isAdmin": "yes",       // ERROR: string not allowed
  "flag": "false"         // ERROR: quoted boolean
}
```

### 2.2 Type Requirements

**MUST:** Boolean fields use nullable/wrapper types

```java
// ✓ CORRECT
private Boolean active;
private Boolean enabled;

// ✗ WRONG
private boolean active;    // Primitive - not allowed
```

### 2.3 Tri-State Logic

**If application requires tri-state (true/false/unknown):**
- Use nullable Boolean (null = unknown)
- Do NOT use strings or custom types
- Document the semantics clearly

```java
// ✓ ACCEPTABLE
private Boolean featureEnabled;  // true, false, or null (unknown)
```

---

## Rule 3: String Handling

### 3.1 Absolute Preservation

**MUST:** String values MUST be preserved exactly as received  
**MUST NOT:** No trimming, normalization, or modification

```json
{
  "name": "  John Doe  ",        // Preserved: "  John Doe  "
  "code": "00123",               // Preserved: "00123"
  "text": "Line1\n\nLine2",      // Preserved with newlines
  "empty": "",                   // Preserved: empty string
  "unicode": "José María 🚀"     // Preserved: Unicode maintained
}
```

**Why?** If the sender included whitespace or leading zeros, they had a reason. Removing them is data loss.

### 3.2 Unicode Support

**MUST:** Full UTF-8 support required  
**MUST:** Unicode escape sequences handled correctly  
**MUST:** Emoji and special characters preserved

```java
// ✓ CORRECT - All representations work
private String name;           // "José María"
private String emoji;          // "🚀"
private String escaped;        // "\u0048\u0065\u006C\u006C\u006F" = "Hello"
```

### 3.3 No Auto-Conversion

**MUST NOT:** Attempt to convert strings to other types automatically

```json
// Input JSON
{"count": "123"}

// ✗ WRONG - Don't auto-convert string to number
// This should cause an error or be explicit
```

---

## Rule 4: Array Handling

### 4.1 Explicit Declaration

**MUST:** Array/collection fields MUST be explicitly declared  
**Default:** Single values only (non-array)

```java
// ✓ CORRECT - Explicit array declaration
private List<String> tags;
private Set<Integer> scores;

// ✗ WRONG - Implicit array/single value handling
// Fields should not accept both single values and arrays
```

### 4.2 Strict Array Format Required

**MUST:** Array/List fields MUST always receive values in array format `[]`  
**MUST NOT:** Single values for array fields (no auto-wrapping)

```json
✓ VALID
{
  "tags": ["new", "sale"],      // Array with multiple values
  "items": ["single"],          // Array with single value
  "scores": []                  // Empty array
}

✗ INVALID
{
  "tags": "new",               // ERROR: single value, not array
  "items": "value",            // ERROR: must use ["value"]
  "scores": null               // ERROR: use [], not null
}
```

**Why?** Clear type contract. No ambiguity. Developer immediately knows what to expect.

### 4.3 No Auto-Wrapping

**MUST NOT:** Implementations must NOT auto-wrap single values to arrays  
**MUST:** If JSON provides `"tags": "value"` for array field, reject with error

```java
// ✗ DON'T DO THIS
// Don't auto-convert "tags": "value" to ["value"]
// This violates the contract

// ✓ CORRECT
// If JSON has "tags": "value" for List<String> tags field:
//   ERROR: Field 'tags' expected array [] but got string
```

### 4.3 Supported Collection Types

**MUST Support:**
- `List<T>` — Preserves order and duplicates
- `Set<T>` — Removes duplicates

**SHOULD Support:**
- List implementations (`ArrayList`, `LinkedList`) — Use through `List` interface only

**MUST NOT Support:**
- `Map<String, Object>` — Type safety lost. Use typed classes instead
- `Map<String, ?>` with untyped values — All values must be explicitly typed
- Untyped collections — All generics required
- Custom collection types — Standard JDK collections only

**Use Typed Classes Instead of Map<String, Object>:**

```java
// ✗ WRONG - Type ambiguity
private Map<String, Object> metadata;

// ✓ CORRECT - Explicit structure
private class Metadata {
    private String key;
    private String value;
    private Long timestamp;
}
private Metadata metadata;

// ✓ CORRECT - If you need typed key-value pairs
private class Attributes {
    private Map<String, String> values;  // Explicitly typed values
}
private Attributes attributes;
```

**Why?** `Object` defeats type safety. Make the structure explicit in a class definition — it's clearer, safer, and follows the manifesto's "Make Illegal States Unrepresentable" principle.

---

## Rule 5: Null Values

### 5.1 Missing Field Behavior

**Default (Lenient Mode):** Missing fields become null  
**Strict Mode (Optional):** Missing fields throw error

```java
// JSON: {"name": "John"}  (age missing)

// Lenient mode (default)
private String name;      // "John"
private Integer age;      // null

// Strict mode (if supported)
private String name;      // "John"
private Integer age;      // ERROR: Field 'age' is required
```

### 5.2 Explicit Null Values

**MUST NOT:** `null` values in JSON are forbidden  
**Must:** Omit field instead

```json
✓ VALID - Omit missing field
{
  "name": "John"
}

✗ INVALID - Don't use null
{
  "name": "John",
  "age": null          // ERROR: null not allowed
}
```

### 5.3 Default Values

**MUST:** Missing fields default to null when appropriate  
**MUST NOT:** No automatic default values (empty strings, zero, false)

```java
// Missing fields always become null
private String name = null;      // Missing -> null
private Integer age = null;      // Missing -> null
private Boolean active = null;   // Missing -> null

// ✗ WRONG - Don't auto-provide defaults
// private String name = "";       // Incorrect default
// private int age = 0;            // Incorrect default
```

---

## Rule 6: Date/Time Handling

### 6.1 Format Standard

**MUST:** Only ISO 8601 UTC format accepted  
**Format:** `YYYY-MM-DDTHH:mm:ss[.fff]Z`

```json
✓ VALID
{
  "timestamp": "2024-12-25T14:30:00Z",
  "created": "2024-12-25T14:30:00.123Z",
  "updated": "2025-01-15T09:45:30.456Z"
}

✗ INVALID
{
  "timestamp": "2024-12-25T14:30:00+03:00",    // Timezone not UTC
  "created": "2024-12-25",                     // Date only
  "date": "Dec 25, 2024",                      // Natural language
  "time": "14:30:00",                          // Time only
  "timestamp": 1735126200000                   // Milliseconds
}
```

### 6.2 Type Requirements

**MUST:** Use language-appropriate temporal types

```java
// ✓ CORRECT
private Instant timestamp;              // Java - ISO 8601 parsing
private OffsetDateTime created;         // With offset info
private LocalDateTime updated;          // When offset not needed

// Language equivalents
// TypeScript: Date (with ISO string input)
// Python: datetime.datetime
// Go: time.Time
// Kotlin: Instant
```

### 6.3 Timezone Handling

**MUST:** All dates interpreted as UTC  
**MUST:** No timezone conversion without explicit requirement

```
"2024-12-25T14:30:00Z" = UTC time 14:30:00
Not converted to local time automatically
Application handles conversion if needed
```

---

## Rule 7: Field Mapping

### 7.1 Default Mapping

**MUST:** Exact field name matching (case-sensitive)  
**MUST NOT:** No automatic case conversion

```json
{
  "userName": "john",
  "user_name": "jane",
  "UserName": "bob"
}
```

```java
// ✓ CORRECT - Each maps to specific field
private String userName;       // Matches: "userName" only
private String user_name;      // Matches: "user_name" only
private String UserName;       // Matches: "UserName" only
```

### 7.2 Custom Mapping

**If supported:** Single custom mapping per field

```java
// ✓ EXAMPLE (if implementation supports this)
// Pseudo-code - actual syntax depends on implementation
private String firstName;      // Maps to: "first_name" in JSON
private String email;          // Maps to: "email_address" in JSON
```

**MUST NOT:** Multiple aliases for one field

```java
// ✗ WRONG
// Don't support: "email" OR "mail" OR "email_address"
// Support only ONE mapping per field
```

### 7.3 Unknown Fields

**Default:** Unknown fields ignored  
**Strict Mode:** Unknown fields may cause error

```json
// Input
{"name": "John", "unknownField": "value"}

// Lenient (default): Processes "name", ignores "unknownField"
// Strict mode: May error on unknown field
```

---

## Rule 8: Type System

### 8.1 Supported Types

Implementations MUST support:

**Primitive/Scalar Types:**
- String
- Integer (32-bit)
- Long (64-bit)
- Double (floating point)
- Float (floating point)
- Boolean

**Temporal Types:**
- Instant / Timestamp (ISO 8601 UTC)

**Collection Types:**
- List<T>
- Set<T>

**Complex Types:**
- Nested objects (other DTO classes)
- Nested collections (List<DTO>, Set<DTO>)

### 8.2 Unsupported Types

**MUST NOT** support:

```java
private Map<String, Object> metadata;      // Untyped Map - no generics
private String[] tags;                     // Arrays not supported (use List)
private int age;                           // Primitives not supported (use Integer)
private custom.Collection<String> items;   // Custom collections not supported
private Optional<String> value;            // Optionals not supported (use nullable)
```

**CAN Support (with type constraints):**
```java
private Map<String, String> properties;    // Typed Map - only if all values same type
private Map<String, Integer> scores;       // Typed Map - explicit value type
```

**Why?** 
- Untyped `Object` loses type safety and breaks the manifesto's core promise
- Use explicit typed classes instead of ambiguous Maps
- If you need key-value storage, define a class with named fields

---

## Rule 9: Nested Objects

### 9.1 Requirements

**MUST:** Nested objects MUST conform to these rules  
**MUST:** Recursive validation applied to nested structures

```java
// ✓ CORRECT - Nested object follows all rules
class OrderDTO {
    private String orderId;
    private CustomerDTO customer;        // Valid nested DTO
    private List<ProductDTO> items;      // Valid nested collection
}

class CustomerDTO {
    private String name;
    private String email;
    private Address address;             // Can nest further
}
```

### 9.2 Nesting Depth Limit

**MUST:** Maximum 10 levels of nesting  
**ENFORCEMENT:** Compilation error or runtime error if exceeded

```
Level 1 -> Level 2 -> ... -> Level 10: ✓ OK
Level 1 -> Level 2 -> ... -> Level 11: ✗ ERROR
```

This prevents accidental deep structures and performance issues.

### 9.3 Circular Reference Prevention

**MUST NOT:** Circular references allowed  
**ENFORCEMENT:** Compilation error if detected

```java
// ✗ FORBIDDEN - Circular reference
class UserDTO {
    private GroupDTO group;
}

class GroupDTO {
    private UserDTO owner;  // ERROR: Circular reference detected
}
```

---

## Rule 10: Error Handling

### 10.1 Fail Fast Philosophy

**MUST:** Invalid data causes immediate exception  
**MUST NOT:** Silent failures or data corruption

Error messages MUST include:
- Field name
- Line and column (if available)
- Expected vs actual value
- Helpful suggestion

```
✓ EXAMPLE ERROR
Field 'age' at line 3, column 10: 
  Expected Integer but got String "abc"
  Suggestion: Remove quotes from numeric values
```

### 10.2 Error Categories

**Invalid Type:**
```
"age": "abc"  →  ERROR: Expected Integer, got String
```

**Missing Required Field (strict mode):**
```
{} (missing "age")  →  ERROR: Required field 'age' is missing
```

**Invalid Format:**
```
"date": "2024/12/25"  →  ERROR: Invalid date format. Use ISO 8601 UTC: 2024-12-25T14:30:00Z
```

**Depth Exceeded:**
```
Nesting Level 11  →  ERROR: Maximum nesting depth (10) exceeded
```

**Circular Reference:**
```
UserDTO -> GroupDTO -> UserDTO  →  COMPILATION ERROR: Circular reference detected
```

### 10.3 Error Recovery

**MUST NOT:** Partial object creation on error  
**MUST:** Entire deserialization fails or succeeds

```java
// ✗ WRONG - Don't return partially initialized object
// Object with null fields, missing required data, undefined state

// ✓ CORRECT - All or nothing
// Either complete valid object or exception thrown
```

---

## Rule 11: Nesting Depth Limits

### 11.1 Maximum Depth

**LIMIT:** 10 levels maximum

```
Level 1: Root object
Level 2: First nested object
...
Level 10: Deepest allowed
Level 11+: ERROR
```

### 11.2 Rationale

- Prevents accidental deep structures
- Encourages flatter, more maintainable schemas
- Protects against DoS attacks (stack overflow)
- Improves readability

### 11.3 Enforcement

**MUST:** Check at:
- Compile time (if possible)
- Runtime deserialization (if compile-time not possible)

---

## Rule 12: Circular References

### 12.1 Why Circular References Are Forbidden

**Critical Issues:**

1. **Infinite Recursion** — Deserialization never terminates
   ```
   Parse A → Parse B (part of A) → Parse A (part of B) → Parse B → ...
   ```

2. **Stack Overflow** — Each recursion level consumes stack memory
   ```
   Level 1: A references B (stack: 1 KB)
   Level 2: B references A (stack: 2 KB)
   Level 3: A references B (stack: 3 KB)
   ... until stack exhausted → Exception
   ```

3. **Memory Explosion** — Objects never garbage collected
   ```
   Object A holds B, B holds A
   Both always referenced = memory leak
   ```

4. **JSON Representation Problem** — Impossible to represent in JSON
   ```json
   {"a": {"b": {"a": {"b": {...}}}}}  // Infinite nesting
   ```

5. **DoS Attack Vector** — Malicious JSON could crash system
   ```json
   {"user": {"group": {"owner": {"group": {...}}}}}
   Attacker sends infinite circular structure
   ```

### 12.2 Detection

**MUST:** Detect before processing

```java
// ✗ FORBIDDEN
class A { B b; }
class B { A a; }

// Error during compilation or initialization:
// "Circular reference detected: A -> B -> A"
```

### 12.3 Prevention

**MUST NOT:** Allow any circular structure

**Common mistakes:**
- Bidirectional relationships (user ↔ group)
- Self-references (node → parent node → ...)
- Indirect cycles (A → B → C → A)

### 12.4 Alternative

If bidirectional data needed, use separate structures:

```java
// ✓ CORRECT - No cycles
class UserDTO {
    private String id;
    private String name;
    private String groupId;    // ID reference, not object
}

class GroupDTO {
    private String id;
    private String name;
    // Don't include List<UserDTO> members
}
```

---

## Rule 13: Security

### 13.1 No Polymorphic Typing

**MUST NOT:** Support `Object` or polymorphic type loading  
**MUST NOT:** Allow arbitrary class instantiation

```java
// ✗ FORBIDDEN
mapper.readValue(json, Object.class);      // NO
mapper.readValue(json, Unknown.class);     // NO
mapper.enableDefaultTyping();              // NO
```

### 13.2 No Reflection for Field Access

**MUST:** Use generated code only  
**MUST NOT:** Runtime reflection on fields

```java
// ✓ CORRECT
generated_code.setField(dto, value);

// ✗ WRONG
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);
field.set(dto, value);
```

### 13.3 Why Input Size Limits Are Required

**Critical Security & Performance Reasons:**

**1. DoS (Denial of Service) Attack Prevention**
   ```
   Attacker sends:
   - Unbounded JSON file → Memory exhaustion → System crash
   - Array with unlimited elements → Processing hangs → Service unavailable
   - Deeply nested structure (no limit) → Stack overflow
   
   Result: Legitimate users can't access service
   ```

**2. Memory Exhaustion**
   ```
   Without limit:
   {"data": [... unlimited items ...]}
   
   System tries to allocate arbitrary memory:
   - Kills other processes
   - GC pressure increases
   - OOM errors cascade
   ```

**3. Stack Overflow Prevention**
   ```
   Unlimited nesting:
   {"a": {"b": {"c": ... {"z": {"value": 1}}}}}
   
   Each level adds to call stack
   Stack exhausted = JVM crash
   ```

**4. CPU Exhaustion**
   ```
   Large payloads:
   String parsing, validation, copying
   CPU spikes to 100%
   Legitimate requests starved
   ```

**5. Malicious Payload Protection**
   ```
   Attacker intentionally sends oversized data
   Without limits: System easily exploitable
   With limits: Attack fails fast with clear error
   ```

### 13.4 Recommended Default Limits

**MUST Enforce These:**

| Limit | Value | Rationale |
|-------|-------|-----------|
| Max JSON size | **10 MB** | Prevents memory exhaustion, handles realistic bulk operations |
| Max array size | **10,000 elements** | Encourages pagination, prevents spike |
| Max string length | **1 MB** | Most text fields stay under this, catches bombs |
| Max nesting depth | **10 levels** | Prevents stack overflow, enforces flat design |

### 13.5 When These Limits Are Exceeded: Design Problem

If you regularly hit these limits, there's an architectural issue:

**String exceeds 1 MB:**
```
❌ Wrong: Storing 50MB text in JSON field
{"document": "very long content..."}

✓ Correct: Use file upload/storage
POST /documents (multipart/form-data)
Store in S3, Google Cloud Storage, or file system
Return URL reference in JSON: {"documentUrl": "..."}
```

**Array exceeds 10,000 elements:**
```
❌ Wrong: Return all results at once
GET /users → Returns 1 million users

✓ Correct: Implement pagination
GET /users?page=1&limit=100&offset=0

✓ Correct: Add filtering
GET /users?status=active&created_after=2024-01-01

✓ Correct: Use cursors
GET /users?cursor=abc123&limit=100
```

**JSON exceeds 10 MB:**
```
❌ Wrong: Send entire bulk export as one payload
POST /sync {"data": [... all 500MB records ...]}

✓ Correct: Use batching
POST /sync/batch?number=1 (10MB chunk)
POST /sync/batch?number=2 (next 10MB)

✓ Correct: Use streaming protocol
gRPC with streaming
Server-Sent Events (SSE)
WebSocket with message chunks
```

**The Principle:** If legitimate business requires exceeding these limits, it's a signal that JSON might not be the right protocol. Consider file uploads, streaming, or alternative designs.

### 13.6 Configuration Per Environment

Implementations SHOULD allow configuration:

```
Development:
  max_json_size: 100 MB  (loose for testing)
  max_array_size: 1,000,000 (flexible)

Staging:
  max_json_size: 50 MB
  max_array_size: 100,000

Production:
  max_json_size: 10 MB
  max_array_size: 10,000
  max_string_length: 1 MB
  max_nesting_depth: 10
```

### 13.7 Error Messages for Size Violations

When limit exceeded, fail fast with actionable error:

```
✓ GOOD ERROR
"Request rejected: JSON payload 45MB exceeds 10MB limit (400 Bad Request)

Reasons this happened:
- Sending too many records at once
- Including large file as base64 in JSON
- Missing pagination in API design

Solutions:
1. Paginate results: GET /data?page=1&limit=100
2. Filter results: GET /data?status=active&date_range=7d
3. Use file upload: POST /files (multipart/form-data)
4. Split into multiple requests with cursor pagination

Documentation: https://api.example.com/pagination-guide"

✗ BAD ERROR
"413 Payload Too Large"
"Error processing request"
```

### 13.8 No Dynamic Code Generation

**MUST NOT:** Runtime code generation  
**MUST NOT:** eval() or similar  
**MUST:** All code paths compile-time determined

---

## Rule 14: Performance

### 14.1 Zero Reflection

**MUST:** Generated code only  
**MUST NOT:** Reflection at deserialization time

```java
// ✓ CORRECT - Direct field assignment
dto.setName(parser.getString());
dto.setAge(parser.getInt());

// ✗ WRONG - Reflection
Field field = clazz.getDeclaredField("name");
field.set(dto, value);
```

### 14.2 Minimal Allocations

**MUST:** Minimize object creation  
**SHOULD:** Cache string constants, reuse where possible

```java
// ✓ PREFERRED
private static final String FIELD_NAME = "name";
int age = parser.getInt();  // Primitive, not boxed

// ✗ AVOID
Integer age = Integer.valueOf(parser.getInt());  // Unnecessary boxing
```

### 14.3 Switch-Case Optimization

**SHOULD:** Use switch-case for field mapping when possible

```java
// ✓ FAST - Switch statement for O(1) lookup
switch (fieldName) {
    case "name" -> dto.setName(parser.getString());
    case "age" -> dto.setAge(parser.getInt());
    case "email" -> dto.setEmail(parser.getString());
    default -> parser.skipValue();
}

// ✗ SLOWER - Linear search
if ("name".equals(fieldName)) { ... }
else if ("age".equals(fieldName)) { ... }
// or
fieldHandlers.get(fieldName).handle(value);  // Map lookup
```

---

## Rule 15: JSON Compliance

### 15.1 RFC 8259 Compliance

**MUST:** Strict JSON standard compliance  
**MUST NOT:** JSON extensions

```json
✓ VALID - RFC 8259 compliant
{
  "name": "John",
  "age": 25,
  "active": true,
  "tags": ["a", "b"]
}

✗ INVALID - Non-standard
{
  "name": "John",        // Comments not allowed
  "age": 25,            // Trailing comma not allowed (in object)
  unquoted: "value"     // Unquoted keys not allowed
}
```

### 15.2 Character Encoding

**MUST:** UTF-8 encoding required  
**MUST:** Proper Unicode handling

```json
{
  "direct": "José María",                    // Direct UTF-8 OK
  "escaped": "\u0048\u0065\u006C\u006C\u006F"  // Unicode escapes OK
}
```

### 15.3 Number Representation

**MUST:** Support JSON number format  
**Range:** Full range of language numeric types

```json
{
  "int": 123,
  "large": 9007199254740991,           // JavaScript MAX_SAFE_INTEGER
  "decimal": 123.456,
  "negative": -123
}
```

### 15.4 String Escaping

**MUST:** Handle all JSON escape sequences

```
\" → "
\\ → \
\/ → /
\b → backspace
\f → form feed
\n → newline
\r → carriage return
\t → tab
\uXXXX → Unicode
```

---

## Rule 16: Data Transmission Format

### 16.1 Compact JSON Required

**MUST:** JSON MUST be transmitted in compact format (no whitespace)  
**MUST NOT:** Beautified/formatted JSON in production

```
✓ CORRECT (Compact - Production)
{"name":"John","age":25,"email":"john@example.com","active":true}

✗ WRONG (Beautified - Never use in production)
{
  "name": "John",
  "age": 25,
  "email": "john@example.com",
  "active": true
}
```

### 16.2 Why Compact Format

**Performance Benefits:**
- Reduced payload size (typically 20-30% smaller)
- Faster network transmission
- Lower bandwidth costs
- Reduced latency
- Less memory during parsing

**Example Size Comparison:**
```
Beautified: 157 bytes
Compact:    102 bytes
Savings:    55 bytes (35% reduction)

Over 1 million requests:
Beautified: 157 MB
Compact:    102 MB
Savings:    55 MB bandwidth saved
```

**Security Benefits:**
- Harder for attacker to analyze payload
- Slightly reduces attack surface visibility
- No whitespace parsing edge cases

### 16.3 Guidelines

**MUST:**
- Remove all unnecessary whitespace
- No spaces after `:` or `,`
- No newlines or indentation
- No comments

**MUST NOT:**
- Beautify JSON for transmission
- Add debugging information
- Include format version markers
- Add extra fields for readability

### 16.4 Development vs Production

**During Development:**
- Use beautified JSON for debugging
- Enable pretty-printing in logs
- Use formatters in IDE for readability

**Before Transmission:**
- Minify to compact format
- Remove all whitespace
- Validate format

**Tools:**
```
Java: ObjectMapper with compact formatter
JavaScript: JSON.stringify(obj) — already compact
Go: json.Marshal — already compact
Kotlin: kotlinx.serialization — already compact
```

### 16.5 Documentation & Testing

When documenting API responses, ALWAYS show both:

**In Documentation (for readability):**
```json
{
  "id": 123,
  "name": "John",
  "email": "john@example.com"
}
```

**In Wire Protocol (actual transmission):**
```json
{"id":123,"name":"John","email":"john@example.com"}
```

**In Tests:**
```java
String compact = "{\"id\":123,\"name\":\"John\",\"email\":\"john@example.com\"}";
// NOT beautified for actual transmission tests
```

---

## Verification Checklist

When implementing according to Strict JSON principles, verify:

- [ ] Numeric values unquoted in JSON, wrapper types in code
- [ ] Booleans unquoted true/false only
- [ ] Strings preserved exactly (no trimming/normalization)
- [ ] Arrays explicitly declared
- [ ] Null values forbidden in JSON (omit instead)
- [ ] Dates ISO 8601 UTC only
- [ ] Field mapping exact or explicitly custom
- [ ] Types limited to specified set
- [ ] Nested objects follow same rules
- [ ] Max 10 level nesting depth enforced
- [ ] Circular references prevented
- [ ] Fail fast on errors, clear messages
- [ ] No polymorphic type loading
- [ ] Generated code, no runtime reflection
- [ ] Input size limits enforced
- [ ] RFC 8259 compliant
- [ ] UTF-8 support
- [ ] Performance: no reflection, minimal allocations

---

## Questions?

These rules may be strict, but they're designed to catch errors early and make deserialization predictable and safe.

If you're implementing a tool following these rules, we'd love to hear from you. Open an issue or pull request.

If a rule seems too restrictive for your use case, open an issue and discuss. But the bar for changing rules is high — they need to solve real problems for the 99%, not edge cases for the 1%.
