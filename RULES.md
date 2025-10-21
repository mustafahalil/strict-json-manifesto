# Strict JSON Manifesto — Detailed Rules

This document specifies the exact rules that implementations following the Strict JSON Manifesto must enforce.

**Note:** These rules are strict by design, but pragmatism matters. Where rules conflict with legitimate production needs, use adapters or make explicit trade-offs documented.

---

## Table of Contents

- [How to Use This Document](#how-to-use-this-document)
- [Quick Verification Checklist](#quick-verification-checklist)
- [Implementation Approaches](#implementation-approaches)
- [Rule 1: Numeric Values](#rule-1-numeric-values)
- [Rule 2: Boolean Values](#rule-2-boolean-values)
- [Rule 3: String Handling](#rule-3-string-handling)
- [Rule 4: Array Handling](#rule-4-array-handling)
- [Rule 5: Null and Missing Fields](#rule-5-null-and-missing-fields)
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

---

## How to Use This Document

These rules can be implemented in three ways:

### 1. New Strict JSON Implementation
Build a tool from scratch following these rules
- **Best for:** New projects, teams with capacity, maximum control
- **Effort:** High (build parser, code generator, tooling)
- **Result:** Perfect compliance, best UX

### 2. Strict Configuration of Existing Tools
Configure Jackson, Gson, or other libraries to follow these rules
- **Best for:** Existing projects, immediate results
- **Effort:** Low (configuration + validation layer)
- **Result:** Good compliance, leverages familiar tools

### 3. Hybrid Approach
Mix strict internals with adapters for messy external data
- **Best for:** Mature systems, multiple teams
- **Effort:** Medium (architecture + adapters)
- **Result:** Pragmatic safety where it matters

Each rule includes implementation guidance for all three approaches.

---

## Quick Verification Checklist

When implementing according to Strict JSON principles, verify:

- [ ] Numeric values unquoted in JSON, wrapper types in code
- [ ] Booleans unquoted true/false only
- [ ] Strings preserved exactly (no trimming/normalization)
- [ ] Arrays explicitly declared with type information
- [ ] Null values handled explicitly (documented strategy)
- [ ] Dates ISO 8601 UTC only
- [ ] Field mapping exact or explicitly custom
- [ ] Types limited to specified set (no Object/Any)
- [ ] Nested objects follow same rules
- [ ] Max 10 level nesting depth enforced
- [ ] Circular references prevented at compile time
- [ ] Fail fast on errors with clear messages
- [ ] No polymorphic type loading
- [ ] Generated code preferred, no reflection
- [ ] Input size limits enforced
- [ ] RFC 8259 compliant JSON

---

## Implementation Approaches

### Configuration Profiles

#### Strict Profile (Recommended)
```java
// Jackson
ObjectMapper mapper = new ObjectMapper();
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, true);
mapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, false);
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, false);
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, false);
mapper.configure(JsonParser.Feature.ALLOW_TRAILING_COMMA, false);
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
```

#### Lenient Profile (With Adapters)
```java
// Jackson + custom validation layer
ObjectMapper mapper = new ObjectMapper();
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, true);
mapper.configure(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT, false);
// Add validation layer for strict requirements
```

---

## Rule 1: Numeric Values

### Why This Rule Exists

**Production Incident:** E-commerce platform accepted price as both String and Number
- **2018:** Design decision: support both for "flexibility"
- **2023:** Type confusion bug in calculation logic
- **Impact:** Customers charged 10x their intended price
- **Root cause:** Flexible type handling + calculation bug = missed in testing

**Principle:** Type clarity at the boundary prevents downstream bugs.

### The Rule

- **MUST:** Numbers in JSON MUST be unquoted
- **MUST:** Use wrapper/nullable types in code (Integer, not int)
- **FORBIDDEN:** Quoted numbers `"age": "25"` ✗
- **VALID:** Unquoted numbers `"age": 25` ✓

### Why It Matters

- Clear type intent at parsing time
- Errors caught immediately, not hidden in calculation
- No accidental type coercion
- Better performance (no string-to-number conversion needed)

### Implementation

**With Jackson (Strict):**
```java
mapper.configure(JsonParser.Feature.ALLOW_NUMERIC_LEADING_ZEROS, false);
mapper.configure(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS, true);

// Use wrapper types
private Integer age;      // ✓ Correct
private Long id;          // ✓ Correct
private Double price;     // ✓ Correct
// NOT: private int age; (primitive not allowed)
```

**With Gson:**
```java
gson = new GsonBuilder()
    .setObjectToNumberStrategy(ToNumberPolicy.LONG_OR_DOUBLE)
    .create();

private Integer age;      // ✓ Wrapper type
```

**From Scratch:**
```
Parser rules:
- Reject if number is quoted
- Accept: 123, -456, 123.45
- Reject: "123", 1.23e10 (scientific notation)
- Use language's nullable numeric type
```

### Common Mistakes

```java
// ✗ WRONG - Primitive type
private int age;

// ✗ WRONG - Quoted number
{"age": "25"}

// ✓ CORRECT
private Integer age;
{"age": 25}
```

### When to Be Flexible

If third-party API sends quoted numbers, use adapter:
```java
// External API: {"price": "19.99"}
// Adapter converts to strict: {"price": 19.99}
```

---

## Rule 2: Boolean Values

### Why This Rule Exists

**Problem:** Legacy systems sometimes send booleans as strings/numbers
- `"active": "true"` (string)
- `"active": 1` (number)
- `"active": "yes"` (natural language)

**Result:** Parser complexity, type confusion, silent bugs

### The Rule

- **MUST:** Booleans MUST be unquoted `true` or `false` only
- **MUST:** Use nullable wrapper type (Boolean, not boolean)
- **FORBIDDEN:** String representations, numeric values
- **VALID:** `"active": true` ✓
- **INVALID:** `"active": "true"` ✗, `"active": 1` ✗

### Why It Matters

- Unambiguous true/false state
- No type coercion surprises
- Simpler parser logic
- Tri-state logic supported (true/false/null)

### Implementation

**With Jackson:**
```java
private Boolean isActive;    // ✓ Nullable wrapper

mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, false);
// Enforces true/false syntax
```

**With Gson:**
```java
private Boolean isActive;
gson = new GsonBuilder().create();
// Gson strictly enforces true/false by default
```

**From Scratch:**
```
Parser rules:
- Accept: true, false (lowercase only)
- Reject: "true", True, TRUE
- Reject: 1, 0, "yes", "no"
- Use nullable boolean type
```

### Common Mistakes

```java
// ✗ WRONG - Primitive
private boolean active;

// ✗ WRONG - String representation
{"active": "true"}

// ✓ CORRECT
private Boolean active;
{"active": true}
```

---

## Rule 3: String Handling

### Why This Rule Exists

**Principle:** Strings are user data. Preserve as-is.

### The Rule

- **MUST:** Preserve exactly as received (no trimming, normalization)
- **MUST:** Support full UTF-8
- **MUST NOT:** Auto-convert to other types

**Valid Examples:**
```json
{
  "name": "  John Doe  ",        // Whitespace preserved
  "code": "00123",               // Leading zeros preserved
  "empty": "",                   // Empty string valid
  "unicode": "José María 🚀"     // Full UTF-8 supported
}
```

### Why It Matters

- Data integrity
- Respects user intent (leading zeros, whitespace)
- No hidden transformations
- Full internationalization support

### Implementation

**With Jackson:**
```java
private String name;
mapper.configure(JsonParser.Feature.INCLUDE_SOURCE_IN_EXCEPTION, true);
// Preserves exact string content
```

**With Gson:**
```java
gson = new GsonBuilder().create();
// Gson preserves strings by default
```

---

## Rule 4: Array Handling

### Why This Rule Exists

**Problem:** Some APIs send single values for array fields, others send arrays
- `"tags": "new"` vs `"tags": ["new"]`
- Auto-wrapping hides intention
- Makes schema unclear

### The Rule

- **MUST:** Array fields explicitly declared/typed
- **MUST:** Always expect `[]` format
- **MUST NOT:** Auto-wrap single values to arrays
- **VALID:** `"tags": ["new", "sale"]` ✓
- **INVALID:** `"tags": "new"` ✗ (for List field)

### Why It Matters

- Clear contract: field is always array or never array
- No implicit type coercion
- Parser simpler
- Schema unambiguous

### Implementation

**With Jackson:**
```java
private List<String> tags;           // ✓ Declares array
private Set<Integer> scores;         // ✓ Also valid

// NOT: private String[] tags;       // Array type not used
// NOT: private String tags;         // Single value
```

**With Gson:**
```java
private List<String> tags;
gson = new GsonBuilder()
    .create();
// Reject if not array: throw error
```

**Validation Layer:**
```java
if (json.has("tags") && !json.get("tags").isArray()) {
    throw new ValidationException("Field 'tags' must be array");
}
```

### Common Mistakes

```java
// ✗ WRONG - Type coercion
private List<String> tags;  // But accepts: "value"

// ✗ WRONG - Auto-wrapping
"tags": "new" → ["new"]

// ✓ CORRECT
private List<String> tags;
"tags": ["new", "sale"]
```

### When to Be Flexible

If external API sends single values, use adapter:
```java
// External: {"tags": "new"}
// Adapter: {"tags": ["new"]}
// Then parse as strict
```

---

## Rule 5: Null and Missing Fields

### Why This Rule Exists

**Problem:** "Null" and "missing field" have different semantics
- `{"name": "John"}` — age field missing
- `{"name": "John", "age": null}` — age explicitly null

**In strict systems:** These should be explicit, not ambiguous

**However:** Pragmatism matters. Most systems need to handle both.

### The Rule

**For missing fields:**
- **Default:** Missing fields MAY become null (implementation choice)
- **STRICT MODE (optional):** Missing required fields → ERROR
- **LENIENT MODE:** Missing fields → null

**For explicit null:**
- **PREFERRED:** Omit field instead of setting null
- **ALLOWED:** Explicit null IF documented
- **NEVER:** Ambiguous null that could mean "missing"

**Valid Examples:**
```json
// Omit field (preferred)
{"name": "John"}

// Explicit null (allowed if documented)
{"name": "John", "age": null}

// NOT: {"name": "John", "age": null} mixed with omit pattern
// (pick one pattern and stick to it)
```

### Why It Matters

- Explicit intent: Is field missing or null?
- Tri-state logic possible: true/false/null
- Consistent patterns prevent bugs
- Backward compatibility easier with versioning

### Implementation

**With Jackson (Lenient):**
```java
private String name;
private Integer age;  // null if missing

mapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);
// Primitives can't be null, forces wrapper types
```

**With Jackson (Strict):**
```java
// Add validation: required fields must be present
@NotNull
private String name;

// Optional fields may be null
@Nullable
private Integer age;

mapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);
```

**With Gson:**
```java
private String name;
private Integer age;
// Both may be null
```

**From Scratch:**
```
Strategy 1: Lenient (default)
- Missing field → null
- Explicit null → null
- No error

Strategy 2: Strict (optional)
- Missing required field → ERROR
- Explicit null on optional → null
- Requires annotation: @Required, @Optional

Strategy 3: Explicit (recommended)
- Choose one pattern per API
- Document in API spec: "Fields must be omitted, not set to null"
- Or: "Fields may be null to indicate unknown state"
```

### Common Patterns

```java
// Pattern 1: Lenient (permissive)
private String name;      // null if missing OR explicit null
private Integer age;      // null if missing OR explicit null

// Pattern 2: Strict (enforced)
@NotNull
private String name;      // ERROR if missing or null

@Nullable
private Integer age;      // null if missing or explicit null

// Pattern 3: Tri-state (business logic)
private Boolean featureEnabled;  // true/false/null (unknown)
```

### When to Be Flexible

**OK to relax null rules if:**
- Third-party API sends nulls inconsistently
- Use adapter to convert to consistent pattern
- Internal system uses strict pattern

**NOT OK:**
- Accepting both null and missing field as same (pick one)
- Treating null as empty string or false
- Silent null that causes NullPointerException later

---

## Rule 6: Date/Time Handling

### Why This Rule Exists

**Problem:** Date formats vary across systems
- ISO 8601: `2024-12-25T14:30:00Z`
- Unix epoch: `1735126200`
- Custom format: `12/25/2024`

**Result:** Parser complexity, timezone bugs, DST issues

**Production Incident:** Banking API supported 15+ date formats
- One 2014 integration used custom format
- Format special-cased in parser
- 2020: DST transition broke edge case
- Cost: Full audit + compensation

### The Rule

- **MUST:** ISO 8601 UTC format only
- **FORMAT:** `YYYY-MM-DDTHH:mm:ss[.fff]Z`
- **TIMEZONE:** UTC always (Z = Zulu = UTC)
- **INVALID:** Custom formats, timezones, epoch
- **VALID:** `"2024-12-25T14:30:00Z"` ✓

### Why It Matters

- Standard format, no ambiguity
- Timezone explicit (always UTC)
- No conversion confusion
- Works across all systems

### Implementation

**With Jackson:**
```java
private Instant timestamp;  // Jackson handles ISO 8601 natively

mapper.registerModule(new JavaTimeModule());
mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
// Forces ISO 8601 format

// Custom deserializer if needed:
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());
```

**With Gson:**
```java
private LocalDateTime timestamp;

gson = new GsonBuilder()
    .registerTypeAdapter(LocalDateTime.class, 
        new JsonDeserializer<LocalDateTime>() {
            public LocalDateTime deserialize(...) {
                return LocalDateTime.parse(json, ISO_DATE_TIME);
            }
        })
    .create();
```

**From Scratch:**
```
Parser rules:
- Accept: 2024-12-25T14:30:00Z
- Accept: 2024-12-25T14:30:00.123Z (with milliseconds)
- Reject: 2024-12-25T14:30:00+02:00 (timezone offset)
- Reject: 1735126200 (epoch)
- Reject: 12/25/2024 (custom format)
- Always interpret as UTC
```

### Common Mistakes

```java
// ✗ WRONG - Epoch timestamp
{"date": 1735126200}

// ✗ WRONG - Timezone offset
{"date": "2024-12-25T14:30:00+02:00"}

// ✗ WRONG - Custom format
{"date": "12/25/2024"}

// ✓ CORRECT
{"date": "2024-12-25T14:30:00Z"}
```

### When to Be Flexible

If legacy API uses epoch, use adapter:
```java
// External: {"date": 1735126200}
// Adapter: {"date": "2024-12-25T14:30:00Z"}
// Then parse as strict
```

---

## Rule 7: Field Mapping

### Why This Rule Exists

**Problem:** Field name conventions vary
- camelCase vs snake_case vs kebab-case
- `userName` vs `user_name` vs `user-name`

**Too much flexibility:** Parser must try multiple names

### The Rule

- **DEFAULT:** Exact field name matching (case-sensitive)
- **CUSTOM:** Single explicit custom mapping per field
- **NOT:** Multiple aliases (no `userName` OR `user_name` OR `email_address`)

**Valid:**
```json
{"userName": "john"}
// Maps to: private String userName;  (exact match)

// With custom mapping (if supported):
@JsonProperty("first_name")
private String firstName;
```

### Why It Matters

- Clear schema contract
- No ambiguity about field names
- Parser simple and fast
- No guess-work

### Implementation

**With Jackson:**
```java
private String userName;      // Exact match

@JsonProperty("user_name")    // Custom mapping (single)
private String firstName;

// NOT multiple aliases
```

**With Gson:**
```java
private String userName;

@SerializedName("user_name")
private String firstName;
```

**From Scratch:**
```
Field mapping strategy:
1. Exact name match first
2. If custom mapping exists, use it
3. If no match: throw error
4. No guessing, no fallbacks
```

---

## Rule 8: Type System

### Why This Rule Exists

**Principle:** Type safety prevents bugs. Untyped data → bugs.

### The Rule

**Supported Types:**
- String
- Integer (32-bit)
- Long (64-bit)
- Double, Float
- Boolean
- Instant/LocalDateTime (ISO 8601 UTC)
- List<T> (typed collections)
- Set<T> (typed collections)
- Nested typed objects

**FORBIDDEN:**
- `Object` or untyped values
- `Map<String, Object>` (untyped values)
- `Any` type
- Custom collection types
- Reflection-based field access

**Allowed with Caution:**
```java
Map<String, String> properties;      // ✓ Typed map (String→String)
Map<String, Integer> scores;         // ✓ Typed map (String→Integer)
```

### Why It Matters

- Compiler catches type errors
- No runtime type surprises
- No type coercion bugs
- Performance optimizations possible

### Implementation

**With Jackson:**
```java
// ✓ Type-safe
private List<String> tags;
private Map<String, Integer> scores;

// ✗ NOT allowed
private Map<String, Object> metadata;
private Object value;
```

**With Gson:**
```java
private List<String> tags;
private TypeToken<Map<String, String>> mapType;
```

### Common Mistakes

```java
// ✗ WRONG - Untyped
private Map<String, Object> data;
private Object value;
private List items;

// ✓ CORRECT
private Map<String, String> data;
private String value;
private List<String> items;
```

---

## Rule 9: Nested Objects

### Why This Rule Exists

**Principle:** Nested objects follow same strictness rules.

### The Rule

- **MUST:** Nested objects conform to all rules
- **MUST:** Recursive validation applied
- **LIMIT:** Maximum 10 levels nesting depth

**Valid:**
```java
class OrderDTO {
    private String orderId;
    private CustomerDTO customer;        // ✓ Valid DTO
    private List<ProductDTO> items;      // ✓ Valid collection
}

class CustomerDTO {
    private String name;
    private String email;
    private AddressDTO address;          // ✓ Can nest further
}
```

### Why It Matters

- Consistent validation everywhere
- No "escape hatches" for complex data
- Forces clear structure
- Prevents deeply nested mess

### Implementation

**With Jackson:**
```java
@Valid  // Triggers nested validation
private CustomerDTO customer;

private List<@Valid ProductDTO> items;
```

**From Scratch:**
```
Validation rules:
- For each nested object: run all rules
- Track depth, error if > 10
- Recursively validate all levels
```

---

## Rule 10: Error Handling

### Why This Rule Exists

**Principle:** Fail fast. Silent failures hide bugs.

### The Rule

- **MUST:** Invalid data causes immediate exception
- **MUST NOT:** Silent failures or data corruption
- **Error message MUST include:**
  - Field name
  - Line and column (if available)
  - Expected vs actual value
  - Helpful suggestion

**Example Error:**
```
Field 'age' at line 3, column 10:
  Expected Integer but got String "abc"
  Suggestion: Remove quotes from numeric values
  Path: order.customer.age
```

### Why It Matters

- Bugs caught at boundary, not downstream
- Clear error messages enable fast fixes
- No wasted debugging time
- No production surprises

### Implementation

**With Jackson:**
```java
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, true);
mapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);
// Fails loudly on errors
```

**From Scratch:**
```
Error strategy:
1. Detect error immediately
2. Include context: field, line, column
3. Explain what expected vs what received
4. Suggest fix
5. Don't continue parsing
```

---

## Rule 11: Nesting Depth Limits

### Why This Rule Exists

**Security & Performance:**
- Stack overflow protection
- DoS attack prevention
- Encourages flat, maintainable schemas

### The Rule

- **LIMIT:** Maximum 10 levels deep
- **ENFORCEMENT:** Compile-time (if possible) or runtime error
- **Deeper:** Rejected with clear error

**Nesting Example:**
```
Level 1: {
  Level 2: {
    Level 3: {
      ...
      Level 10: {
        value: "here"  // ✓ OK
      }
    }
  }
}

Level 11: { ... }  // ✗ ERROR
```

### Why It Matters

- Encourages flat, understandable schemas
- Prevents accidental deep nesting
- Stack protection
- Forces API design clarity

### Implementation

**With Jackson:**
```java
// Custom deserializer to track depth
class StrictJsonDeserializer extends StdDeserializer {
    private int depth = 0;
    
    public T deserialize(...) {
        if (depth > 10) {
            throw new ValidationException("Max nesting depth exceeded");
        }
        depth++;
        try {
            // deserialize
        } finally {
            depth--;
        }
    }
}
```

**From Scratch:**
```
Tracking:
- Increment depth on object start
- Decrement on object end
- Throw error if depth > 10
```

---

## Rule 12: Circular References & Bidirectional Relationships

### Why This Rule Exists

**The Real Problem:** Infinite recursion, not relationships

**Distinction:**
- **Bidirectional DAG (allowed):** User → Group → Users (no cycle to root)
- **Circular reference (forbidden):** User → Group → Users → User (cycles to root)

**Real incidents:**
- Naive JSON serialization of ORM entities → infinite loop → stack overflow
- Client can't deserialize because entity has cycles
- API returns 500 error on valid request

### The Rule

**What's FORBIDDEN:**
- Circular references: A → B → A (actual cycle)
- Infinite recursion during serialization/deserialization

**What's ALLOWED:**
- Bidirectional relationships: User ← many-to-many → Group (no cycles if DAG)
- One-way references

**Examples:**

```java
// ✗ FORBIDDEN - Cycle exists
class User {
    String id;
    Group group;
}

class Group {
    String id;
    User owner;  // Creates: User → Group → User (cycle)
}

// ✓ ALLOWED - DAG (Directed Acyclic Graph)
class User {
    String id;
    String name;
    Group group;  // User → Group
}

class Group {
    String id;
    String name;
    List<String> memberIds;  // Group → IDs, not back to User
}

// ✓ ALSO ALLOWED - With proper depth limit
class User {
    String id;
    String name;
    Group group;
}

class Group {
    String id;
    String name;
    @JsonIgnore      // Or: @JsonBackReference
    List<User> members;  // Prevents serialization cycle
}
```

### Three Strategies for Relationships

#### Strategy 1: IDs Only (Recommended for JSON APIs)
**Use when:** REST APIs, microservices, external communication

```java
// User → Group reference
class User {
    String id;
    String name;
    String groupId;          // Just ID
}

// Group with member IDs
class Group {
    String id;
    String name;
    List<String> memberIds;  // Just IDs
}

// Benefits:
// - Flat JSON, no cycles
// - Client fetches what it needs
// - Reduces payload size
```

#### Strategy 2: DTOs for Public APIs (Recommended for strict JSON)
**Use when:** Public APIs, external APIs, when you want strict types

```java
// Internal entity (with cycles OK for ORM)
@Entity
class UserEntity {
    @ManyToOne
    Group group;
    // ...
}

@Entity
class GroupEntity {
    @OneToMany
    List<UserEntity> members;
}

// Public DTO (strict, acyclic)
class UserDTO {
    String id;
    String name;
    String groupId;  // Reference, not object
}

class GroupDTO {
    String id;
    String name;
    List<String> memberIds;
}

// Conversion layer
UserDTO toDTO(UserEntity entity) {
    return new UserDTO(
        entity.getId(),
        entity.getName(),
        entity.getGroup().getId()
    );
}
```

#### Strategy 3: Bidirectional with Annotations (For ORM+JSON)
**Use when:** JPA/Hibernate + JSON serialization needed

```java
@Entity
class User {
    String id;
    
    @ManyToOne
    Group group;  // User → Group
}

@Entity
class Group {
    String id;
    
    @OneToMany(mappedBy = "group")
    @JsonIgnore  // or @JsonBackReference
    List<User> members;  // Prevents serialization cycle
}

// Benefits:
// - Works with ORM
// - Prevents infinite serialization
// - Database relationships intact
```

### Why It Matters

- **Infinite recursion** prevented at serialization level
- **Stack overflow** cannot happen
- **Clear semantics:** Which direction can navigate?
- **Performance:** No redundant data in JSON

### Implementation

**With Jackson:**
```java
// Strategy 1: IDs only
private String groupId;  // Not: private Group group;

// Strategy 2: Use DTOs
private UserDTO {
    String groupId;
}

// Strategy 3: Annotations
@ManyToOne
private Group group;

@OneToMany
@JsonIgnore  // Prevent cycle
private List<User> members;

// or use @JsonBackReference / @JsonManagedReference
@ManyToOne
@JsonBackReference
private Group group;

@OneToMany(mappedBy = "group")
@JsonManagedReference
private List<User> members;
```

**With Gson:**
```java
// Strategy 1: IDs only (recommended)
private String groupId;

// Strategy 3: Custom serializer to break cycle
class GroupSerializer implements JsonSerializer<Group> {
    public JsonElement serialize(Group group, ...) {
        JsonObject obj = new JsonObject();
        obj.addProperty("id", group.getId());
        obj.addProperty("name", group.getName());
        // Don't serialize members (prevents cycle)
        return obj;
    }
}
```

### When to Use Each Strategy

| Strategy | Use Case | Pros | Cons |
|----------|----------|------|------|
| IDs Only | REST APIs, microservices | Flat, strict, no cycles | Extra requests needed |
| DTOs | Public APIs, external | Type-safe, strict | Conversion layer |
| Annotations | Internal, ORM+JSON | Works with Hibernate | Need Jackson config |

### Common Mistakes

```java
// ✗ WRONG - Infinite recursion
@Entity
class User {
    @ManyToOne
    Group group;
}

@Entity  
class Group {
    @OneToMany
    List<User> members;  // BOTH sides serialized = cycle
}

UserDTO user = new UserDTO(entity);  // Stack overflow

// ✓ CORRECT - Use one of the three strategies above
```

### For Strict JSON Deserialization

If receiving circular data from external source (shouldn't happen with strict JSON):
```java
// Adapter layer detects and converts
{
    "id": "user1",
    "name": "John",
    "group": {
        "id": "group1",
        "name": "Admins"
        // No "members" field = no cycle
    }
}
```

---

## Rule 13: Security

### Why This Rule Exists

**Production Incidents:**
- CVE-2017-9424 (Jackson): RCE via enableDefaultTyping()
- CVE-2017-9785 (Genson): RCE
- CVE-2022-25647 (Gson): DoS via unbounded parsing

### The Rules

#### 13.1 No Polymorphic Typing
- **MUST NOT:** Support `Object` type
- **MUST NOT:** Allow arbitrary class instantiation
- **RESULT:** No RCE vectors

```java
// ✗ FORBIDDEN
mapper.readValue(json, Object.class);
mapper.enableDefaultTyping();

// ✓ CORRECT
mapper.readValue(json, UserDTO.class);  // Specific type
```

#### 13.2 Reflection Usage: Generated Code Preferred
- **SHOULD NOT:** Use reflection for field access (prefer generated code)
- **ACCEPTABLE:** Jackson/Gson reflection usage (they optimize well)
- **ACCEPTABLE:** JPA/Hibernate reflection (necessary for ORM)
- **GOAL:** Minimize reflection in deserialization path where possible
- **RESULT:** Better performance, GraalVM compatibility, predictability

**Rationale:**
Reflection isn't inherently bad, but generated code is better when possible. Jackson and Gson are well-optimized and use reflection efficiently. JPA/Hibernate require reflection by design. The principle is: prefer generated code in your deserialization layer, accept reflection where frameworks require it.

```java
// ✗ AVOID in custom code
Field field = clazz.getDeclaredField("name");
field.set(object, value);

// ✓ ACCEPTABLE - Jackson handles reflection internally
ObjectMapper mapper = new ObjectMapper();  // Uses reflection optimally

// ✓ ACCEPTABLE - JPA/Hibernate requires reflection
@Entity
class User {
    private String name;
    // Hibernate uses reflection to set fields
}

// ✓ BEST - Generate code for your own deserialization
// (if building custom tool from scratch)
public class UserDTODeserializer {
    public UserDTO deserialize(String json) {
        // No reflection: direct field assignment
        UserDTO dto = new UserDTO();
        dto.setName(jsonObject.getString("name"));
        return dto;
    }
}
```

#### 13.3 Input Size Limits
- **MUST:** Enforce maximum JSON size (recommend 10 MB)
- **MUST:** Enforce maximum array elements (recommend 10,000)
- **MUST:** Enforce maximum string length (recommend 1 MB)

**Why:** Prevents DoS attacks, memory exhaustion

### Implementation

**With Jackson:**
```java
mapper.getFactory().setStreamReadConstraints(
    StreamReadConstraints.builder()
        .maxStringLength(1_000_000)
        .maxNestingDepth(100)
        .build()
);

// For input validation:
if (json.length() > 10_000_000) {
    throw new PayloadTooLargeException();
}
```

**From Scratch:**
```
Security checklist:
1. Type: Enforce specific types only
2. Reflection: Use generated code, no reflection
3. Size: Validate JSON size < 10MB
4. Array: Validate array elements < 10,000
5. String: Validate string length < 1MB
6. Nesting: Validate depth < 10 levels
```

---

## Rule 14: Performance

### Why This Rule Exists

**Principle:** Strict rules enable performance optimizations.

### The Rules

#### 14.1 Zero Reflection
- **MUST:** Generated code only
- **RESULT:** No runtime overhead
- **Benefit:** Predictable performance, GraalVM compatible

#### 14.2 Minimal Allocations
- **MINIMIZE:** Object creation
- **CACHE:** Reuse where possible
- **BENEFIT:** Lower GC pressure

#### 14.3 Switch-Case Optimization
- **PREFER:** Switch statements for field mapping (O(1) lookup)
- **AVOID:** Linear search or HashMap lookup

**Example:**
```java
// ✓ FAST - Switch statement
switch (fieldName) {
    case "name" -> dto.setName(...);
    case "age" -> dto.setAge(...);
    case "email" -> dto.setEmail(...);
    default -> skip();
}

// ✗ SLOWER - HashMap
fieldHandlers.get(fieldName).handle(value);
```

### Implementation

**With Jackson:**
- Generated code via annotation processors
- Reflection minimized via generated code paths

**From Scratch:**
```
Performance rules:
1. Generate code at compile time
2. Use switch statements for field lookup
3. Minimize object allocation
4. Cache string constants
5. Profile and benchmark
```

---

## Rule 15: JSON Compliance

### Why This Rule Exists

**Principle:** Strict RFC 8259 compliance ensures interoperability.

### The Rules

#### 15.1 RFC 8259 Compliance
- **MUST:** Strict JSON only (no JSON5, JSONC, etc.)
- **NO:** Comments
- **NO:** Trailing commas
- **NO:** Unquoted keys
- **NO:** Single quotes

#### 15.2 Character Encoding
- **MUST:** UTF-8 encoding required
- **MUST:** Proper Unicode handling
- **VALID:** Direct UTF-8 or Unicode escapes

```json
{
  "direct": "José María",
  "escaped": "\u0048\u0065\u006C\u006C\u006F"
}
```

#### 15.3 Number Representation
- **MUST:** Support language numeric types
- **NO:** Scientific notation

#### 15.4 String Escaping
- **MUST:** Handle all JSON escape sequences
  - `\"` → quote
  - `\\` → backslash
  - `\n` → newline
  - `\t` → tab
  - `\uXXXX` → Unicode

### Implementation

**With Jackson:**
```java
ObjectMapper mapper = new ObjectMapper();
// Natively compliant with RFC 8259
```

**From Scratch:**
```
Compliance checklist:
1. Parse only valid RFC 8259
2. UTF-8 required
3. Handle escape sequences
4. No non-standard extensions
5. Validate all inputs
```

---

## Summary: Implementation Checklist

When building a tool following Strict JSON principles:

- [ ] Rule 1: Numeric unquoted, wrapper types
- [ ] Rule 2: Boolean unquoted true/false only
- [ ] Rule 3: Strings preserved exactly
- [ ] Rule 4: Arrays explicitly typed
- [ ] Rule 5: Null/missing explicit, consistent pattern
- [ ] Rule 6: Dates ISO 8601 UTC only
- [ ] Rule 7: Field mapping exact or explicitly custom
- [ ] Rule 8: Types from approved set only
- [ ] Rule 9: Nested objects follow same rules
- [ ] Rule 10: Fail fast, clear errors
- [ ] Rule 11: Nesting depth max 10 levels
- [ ] Rule 12: Circular references prevented
- [ ] Rule 13: Security (no polymorphism, no reflection, size limits)
- [ ] Rule 14: Performance (generated code, minimal allocations)
- [ ] Rule 15: RFC 8259 compliant

---

## Questions?

These rules are strict by design, but pragmatism matters. If a rule seems too restrictive:

1. **For new projects:** Follow strictly
2. **For existing projects:** Apply via configuration + adapter layer
3. **For hybrid needs:** Mix strict internals with flexible boundaries

If you have production incidents that would be prevented by these rules, open an issue. We'll discuss and learn together.

The goal isn't perfection. It's reliability.
