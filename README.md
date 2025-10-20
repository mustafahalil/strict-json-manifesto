# The Strict JSON Manifesto

[![License: CC-BY-4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](http://creativecommons.org/licenses/by/4.0/)
[![Manifesto Badge](https://img.shields.io/badge/Manifesto-Strict%20JSON-blue)]()

**JSON is just a data transport format. Nothing more.**

We're tired of pretending it's more. We're tired of libraries that try to be everything to everyone. We're done with configuration hell, runtime surprises, and magic that breaks production.

---

## Table of Contents

- [The Problem](#the-problem)
- [What We're Tired Of](#what-were-tired-of)
- [What We Believe](#what-we-believe)
- [The Non-Negotiable Rules](#the-non-negotiable-rules)
- [Our Goal](#our-goal)
- [Scope](#scope)
- [The Principles in Practice](#the-principles-in-practice)
- [Who Should Use This](#who-should-use-this)
- [Get Involved](#get-involved)
- [Translations](#translations)

---

## The Problem

### JSON's Purpose Has Been Lost

JSON's job is simple: move data from Point A to Point B. That's it.

But the ecosystem built around it is anything but simple. We have libraries that:

- Require **47 annotations** to deserialize a simple object
- Throw `NullPointerException` on line 847 because JSON had `"age": "abc"`
- Hide **security vulnerabilities** in fine print
  - [CVE-2017-9424](https://nvd.nist.gov/vuln/detail/CVE-2017-9424) — Jackson RCE
  - [CVE-2017-9785](https://nvd.nist.gov/vuln/detail/CVE-2017-9785) — Genson RCE
  - [CVE-2022-25647](https://nvd.nist.gov/vuln/detail/CVE-2022-25647) — Gson DoS
- Force developers to write **100+ lines** of custom deserializer code
- Use `Unsafe` allocators that bypass constructors and lose default values
- Support 500 edge cases, making the happy path confusing

**We've optimized for the 0.1% of use cases while making the 99% miserable.**

---

## What We're Tired Of

### Configuration Hell
10+ configuration options. 30+ annotations. Incompatible defaults between libraries.

```java
// What this looks like today
@JsonProperty("userName")
@JsonFormat(pattern = "yyyy-MM-dd")
@JsonIgnore
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)
@JsonAnySetter
@JsonInclude(JsonInclude.Include.NON_NULL)
private String name;
```

### Runtime Surprises
Type errors at 2 AM in production because JSON wasn't what we expected.

```json
// Your JSON
{"age": "abc"}

// Your exception
Exception in thread "main" java.lang.NumberFormatException: 
  For input string: "abc"
  at java.lang.NumberFormatException.forInputString
  (... 47 stack frames later ...)
```

### Security Nightmares
RCE vulnerabilities, polymorphic typing explosions, untrusted deserialization.

```java
// Jackson enableDefaultTyping() = Remote Code Execution
mapper.enableDefaultTyping();  // DON'T DO THIS
mapper.readValue(untrustedJson, Object.class);
// Attacker can instantiate arbitrary classes on your system
```

### Wasted Time
Custom deserializers, type adapters, manual mapping, boilerplate.

```java
// 100+ lines to safely deserialize a simple object
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule();
module.addDeserializer(User.class, new UserDeserializer());
mapper.registerModule(module);
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, true);
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, false);
// ... 50+ more lines of configuration
```

### Magic & Reflection
Constructor bypass, default value loss, field introspection at runtime.

```java
// Gson uses sun.misc.Unsafe
private boolean isRequired = true;
// After Gson deserialization: isRequired == false
// Constructor was never called. Default never applied.
```

### Silent Failures
Data corruption masquerading as success. Null fields nobody noticed until production.

```java
// Silently fails, returns null
List<User> users = gson.fromJson(json, List.class);
// Runtime: users is actually List<LinkedHashMap> 
// Type information is lost. NullPointerException on next line.
```

---

## What We Believe

### YAGNI — You Aren't Gonna Need It

Don't support the edge cases used by 0.1% of developers. Don't add configuration for "future flexibility." Build for the real use case, not the imaginary one.

**Result:** Fewer options = easier to learn = fewer bugs

### KISS — Keep It Simple, Stupid

One way to do things. Not fifty. Same behavior, same errors, same results. Predictability > flexibility.

**Result:** No surprises = no debugging = confidence

### Fail Fast — Loud Errors, Early

Invalid data causes immediate, clear errors. Not silent corruption. Not "close enough." Not "maybe it'll work later."

**Result:** Errors are caught where they're cheap to fix, not in production

### Type Safety — Compile-Time Guarantees

Catch errors where they're cheap to fix. Not in production. Not in logs. Not as customer bug reports.

**Result:** Invalid states are impossible to construct

### Zero Magic — No Reflection, No Unsafe, No Tricks

Generated code only. Compile-time determined paths. No runtime surprises. No constructor bypass. No lost default values.

**Result:** Predictable performance, predictable behavior, no side effects

### Boundary Enforcement — Parse Once, Trust Always

Validate JSON once at the system boundary. After that, your types are the guarantee. No defensive validation everywhere.

**Result:** Simpler application code, single point of validation

---

## The Non-Negotiable Rules

### Numeric Values
- **MUST:** Unquoted in JSON, wrapper objects in Java
- **Invalid:** `"age": "25"` ✗
- **Valid:** `"age": 25` ✓

### Booleans
- **MUST:** Unquoted `true`/`false` only
- **Invalid:** `"active": "true"` ✗
- **Valid:** `"active": true` ✓

### Arrays
- **MUST:** Explicit annotation required (`@SmartArray` or equivalent)
- **Without annotation:** Array processing is forbidden

### Null Values
- **FORBIDDEN:** `null` values in JSON
- **Invalid:** `"age": null` ✗
- **Valid:** Omit the field entirely ✓

### Dates
- **MUST:** ISO 8601 UTC format only
- **Invalid:** `"2024/12/25"` ✗
- **Valid:** `"2024-12-25T14:30:00Z"` ✓

### Nesting Depth
- **LIMIT:** Maximum 10 levels
- **Deeper:** Compilation error

### Type Safety
- **MUST:** Generate code, never use reflection
- **Forbidden:** Runtime reflection for field access
- **Required:** Compile-time code generation

### Circular References
- **FORBIDDEN:** Circular references between types
- **Detection:** Compilation error

**See [RULES.md](./RULES.md) for detailed rule specifications.**

---

## Our Goal

Create clean, predictable Data Transfer Objects that:

1. **Transport JSON** from Point A to Point B — nothing more
2. **Validate once** at system boundaries — then trust the types
3. **Make invalid states impossible** — via type constraints, not runtime checks
4. **Require zero configuration** — sensible defaults, one clear way
5. **Work the same way every time** — no surprises, no magic
6. **Fail loudly** — clear error messages, line numbers, what went wrong
7. **Compile to fast code** — no reflection, no allocations, no surprises at runtime

---

## Scope

### What This Manifesto IS

- A set of strict, opinionated rules for deserialization
- A commitment to developer predictability and safety
- An answer to "what's the simplest way to safely move JSON data?"
- A refusal to over-engineer for hypothetical use cases
- Security-first, by omission (no RCE vectors if you can't do it)
- Performance-first, by design (generated code, no reflection)

### What This Manifesto IS NOT

- Not trying to serialize to 50 different formats
- Not supporting custom types or reflection magic
- Not providing escape hatches for edge cases
- Not replacing proper domain modeling and validation
- Not a JSON schema validator (use JSON Schema for that)
- Not a one-size-fits-all solution (it's intentionally opinionated)

---

## The Principles in Practice

### Before: The Old Way

```java
@JsonIgnore
@JsonProperty("user_age")
@JsonFormat(pattern = "yyyy-MM-dd")
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)
@JsonAnySetter
private Integer age;

// + Custom serializer (50+ lines)
// + Custom deserializer (50+ lines)
// + StackOverflowError from circular refs
// + Runtime type error on production
// + No compile-time verification
```

### After: The Strict Way

```java
private Integer age;

// Compile-time verified ✓
// Zero configuration ✓
// Clear error if invalid ✓
// Type-safe at runtime ✓
// No reflection ✓
// Invalid states impossible ✓
```

---

## Who Should Use This

### Perfect For

- **REST APIs** — Clear boundaries, predictable contracts
- **Microservices** — Type-safe service-to-service communication
- **Event Systems** — Reliable event deserialization, impossible invalid states
- **Security-Conscious Projects** — No polymorphic type injection, no RCE vectors
- **Performance-Sensitive Systems** — Generated code, zero reflection
- **Teams Tired of Configuration** — Sensible defaults, one way to do it

### Probably Not For

- Projects needing maximum flexibility in JSON format
- Systems deserializing arbitrary untrusted JSON structures without schema validation
- Teams comfortable with 47-annotation solutions
- Organizations that value configuration options over conventions
- Anyone wanting Gson/Jackson's full feature set

---

## Get Involved

### Add Your Signature

You believe JSON should be simple? Data transfer should be predictable? Invalid states should be impossible?

**Sign the manifesto.** [Add your name and organization here](./SIGNATORIES.json) and help us prove this philosophy has community backing.

### Contribute

1. **Found a problem?** Open an issue describing your pain point
2. **Want to translate?** Create a PR in `translations/[language-code]/`
3. **Have a reference implementation?** Link it in `IMPLEMENTATIONS.md`
4. **Using this?** Add your company to the "In the Wild" section

See [CONTRIBUTING.md](./CONTRIBUTING.md) for detailed guidelines.

### Share

- **GitHub:** Star, fork, contribute
- **Social:** Tag us in your posts: "I'm adopting Strict JSON principles"
- **Blog:** Write about your experience (we'll link it)
- **Conference Talks:** Speak about type-safe data transfer

---

## The Contract

**You give us:**
- Strict RFC 8259 JSON
- Unquoted numbers, booleans, strings (as specified)
- Clear structure, max 10 levels deep

**We give you:**
- Compile-time verification (errors before runtime)
- Type-safe DTOs (impossible invalid states)
- Fast deserialization (generated code, no reflection)
- Clear error messages (line numbers, what went wrong)
- Predictable behavior (same every single time)

---

## FAQ

### Q: This seems too strict. What if I need flexibility?

**A:** Then this manifesto isn't for you, and that's okay. Use Jackson with full configuration. This is for the 99% use case. We're being intentionally opinionated.

### Q: What about other JSON features I might need?

**A:** If you need them, you probably don't have a pure data transfer problem. Use a full-featured library. We're not trying to be everything.

### Q: How is this different from JSON Schema?

**A:** JSON Schema validates structure. This manifesto validates structure AND enforces type safety at the code level. Complementary, not competing.

### Q: Can I contribute if I disagree with a rule?

**A:** Yes. Open an issue. We'll discuss. But the bar for changing rules is high — they need to solve real problems for the 99%, not edge cases for the 1%.

---

## License

This manifesto is released under [CC-BY-4.0](http://creativecommons.org/licenses/by/4.0/).

- Share it, translate it, adapt it
- Build implementations following these principles
- Build tooling and automation
- Build community

**Just keep it strict.**

---

## Related Resources

- [RULES.md](./RULES.md) — Detailed rule specifications
- [IMPLEMENTATIONS.md](./IMPLEMENTATIONS.md) — Reference implementations
- [CONTRIBUTING.md](./CONTRIBUTING.md) — How to contribute
- [SIGNATORIES.json](./SIGNATORIES.json) — Organizations using these principles

---

## Credits

This manifesto synthesizes:
- Martin Fowler's DTO patterns
- Alexis King's "Parse, Don't Validate"
- Scott Wlaschin's type-driven design
- Yaron Minsky's "Make Illegal States Unrepresentable"
- Real pain points from thousands of frustrated developers
- Security research on deserialization vulnerabilities

---

**JSON is just a data transport format. Let's treat it that way.**

*Manifest your support. Sign below. Build implementations. Change your projects. Change the industry.*

---

Questions? Ideas? [Open an issue](../../issues/new).

Found a problem? [Report it](../../issues/new).

Want to help? [Contribute](./CONTRIBUTING.md).
