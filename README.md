# The Strict JSON Manifesto

[![License: CC-BY-4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](http://creativecommons.org/licenses/by/4.0/)
[![Manifesto Badge](https://img.shields.io/badge/Manifesto-Strict%20JSON-blue)]()

**JSON is just a data transport format. Nothing more.**

We're tired of pretending it's more. We're tired of libraries that try to be everything to everyone. We're done with configuration hell, runtime surprises, and magic that breaks production.

But most importantly: we're done with backwards compatibility that hides design flaws and flexibility that causes more bugs than it prevents.

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
- [Why Strictness Works: Evidence From Production](#why-strictness-works-evidence-from-production)
- [FAQ](#faq)
- [Get Involved](#get-involved)
- [License](#license)

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

### The Real Cost: Production Incidents

These are not theoretical problems. They happen daily:

#### Banking API: Date Format Explosion
- **2010:** API launches supporting "flexible" date handling
- **2015:** 15+ different date formats now supported (backwards compatibility)
- **2018:** Parser code: 800+ lines just for date parsing
- **2020:** Bug discovered: transaction timestamps incorrect during DST transition
- **Root cause:** One client's 2014 integration used a format that special-cased in the parser. This edge case broke in 2020.
- **Cost:** Full audit, client compensation, reputation damage

#### E-Commerce Platform: Type Confusion Attack
- **2018:** Product price field supports String AND Number (for backwards compatibility)
- **2019:** One integration sends `"price": "19.99"` (quoted)
- **2023:** Type confusion bug in calculation: customers charged 10x their price
- **Result:** Automatic refunds, but customer trust destroyed permanently
- **Root cause:** Flexible parsing allowed multiple types, bug hid in calculation logic

#### SaaS Webhooks: Silent Failure
- **2015:** Webhook payload shape "flexible" to support multiple versions
- **2018:** Consumer code: 50+ conditional branches for different payload shapes
- **2023:** Old client sends deprecated format → handler crashes → system down 2 hours
- **Customers:** Unable to receive notifications (SLA violation)
- **Root cause:** "We supported backwards compatibility"

#### Security Breach: RCE from Flexibility
- **Multiple libraries:** Jackson, Gson, Genson all experienced RCE vulnerabilities
- **Root cause:** Flexibility to handle "any" type through polymorphic deserialization
- **Impact:** Emergency patching, forensics, client notification, regulatory issues
- **Lesson:** Flexibility in type handling = attack surface

**Pattern:** All root causes share one thing: too much flexibility in parsing.

---

## What We're Tired Of

### Backwards Compatibility Trap

Backwards compatibility sounds noble: "Never break existing clients."

In practice, it locks you into mistakes forever:

```
2010: API design: "date" field as string
2024: Still can't change because some client depends on it
Result: Better formats exist, but you're locked in by historical accident
```

Backwards compatibility actually:

1. **Hides design flaws** — Bad design becomes standard, spreads to other systems
2. **Punishes new clients** — Must learn all 50+ edge cases from legacy integrations
3. **Makes evolution impossible** — v1, v2, v3, v4 all supported = exponential complexity
4. **Explodes parser complexity** — 10x more code, N times slower, still buggy

**Real solution:** Use versioning, not flexible parsing. `/api/v1/endpoint` frozen, `/api/v2/endpoint` clean. Old version sunsets in 12 months. Clients migrate on their schedule.

### Configuration Hell
10+ configuration options. 30+ annotations. Incompatible defaults between libraries.

```java
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
```

### Security Nightmares
RCE vulnerabilities, polymorphic typing explosions, untrusted deserialization.

```java
// Jackson enableDefaultTyping() = Remote Code Execution
mapper.enableDefaultTyping();  // DON'T DO THIS
mapper.readValue(untrustedJson, Object.class);
// Attacker can instantiate arbitrary classes
```

### Flexibility That Fails

What starts as "just support both formats":

```java
// 2010 - Support one date format
return parseDate(json.getString("date"));

// 2012 - "Just support epoch too"
if (dateValue instanceof String) return parseDate(...);
else if (dateValue instanceof Number) return parseEpoch(...);

// 2014 - "Some send arrays"
// 2016 - "Some send nested objects"  
// 2018 - "What if it's both string and number in same array?"
// 2020 - 400+ lines for ONE field
// 2024 - Code is unmaintainable, still buggy
```

**Result:** Parsing logic becomes 10x more complex, N times slower, and STILL has untested edge cases.

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

**Result:** Errors caught where they're cheap to fix, not in production at 2 AM

### Type Safety — Compile-Time Guarantees

Catch errors where they're cheap to fix. Not in production. Not in logs. Not as customer bug reports.

**Result:** Invalid states are impossible to construct

### Zero Magic — No Reflection, No Unsafe, No Tricks

Generated code only. Compile-time determined paths. No runtime surprises. No constructor bypass. No lost default values.

**Result:** Predictable performance, predictable behavior, no side effects

### Boundary Enforcement — Parse Once, Trust Always

Validate JSON once at the system boundary. After that, your types are the guarantee. No defensive validation everywhere.

**Result:** Simpler application code, single point of validation

### Strictness by Design

Restrictions are not limitations. They're clarity boundaries.

- **Strict format** = Simple parser = Reliable system = Fewer bugs
- **Flexible format** = Complex parser = Unreliable system = More bugs

This isn't ideology. It's decades of production experience.

---

## The Non-Negotiable Rules

### Numeric Values
- **MUST:** Unquoted in JSON, wrapper objects in code
- **Invalid:** `"age": "25"` ✗
- **Valid:** `"age": 25` ✓

### Booleans
- **MUST:** Unquoted `true`/`false` only
- **Invalid:** `"active": "true"` ✗
- **Valid:** `"active": true` ✓

### Arrays
- **MUST:** Explicit annotation required
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

### No Polymorphic Typing
- **MUST NOT:** Support `Object` or polymorphic type loading
- **FORBIDDEN:** Allow arbitrary class instantiation

### Security: Input Size Limits
- **MUST:** Enforce maximum JSON size (recommend 10 MB)
- **MUST:** Enforce maximum array elements (recommend 10,000)
- **MUST:** Enforce maximum string length (recommend 1 MB)
- **Why:** Prevents DoS attacks, memory exhaustion, stack overflow

**See [RULES.md](./RULES.md) for detailed specifications.**

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
- Based on decades of production experience and lessons learned

### What This Manifesto IS NOT

- Not trying to serialize to 50 different formats
- Not supporting custom types or reflection magic
- Not providing escape hatches for edge cases
- Not replacing proper domain modeling and validation
- Not a JSON schema validator (use JSON Schema for that)
- Not a one-size-fits-all solution (it's intentionally opinionated)
- Not applicable when backwards compatibility with bad formats is required

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
- **Event Systems** — Reliable event deserialization
- **Security-Conscious Projects** — No RCE vectors
- **Performance-Sensitive Systems** — Generated code, zero reflection
- **Greenfield Projects** — No backwards compat debt
- **Teams Tired of Configuration** — One way to do it

### Use with Adapters For

- **Third-party APIs** — Adapter layer at boundary converts messy data to strict format
- **Legacy Systems** — Integration layer normalizes old formats
- **Data Migration** — Scripts convert old data to strict format

### Probably Not For

- Projects needing maximum flexibility in JSON format
- Systems deserializing arbitrary untrusted JSON without schema
- Teams comfortable with 47-annotation solutions
- Existing large projects with thousands of dependencies
- Applications where backwards compatibility with bad formats is critical

---

## Why Strictness Works: Evidence From Production

### Decades of Lessons Learned

This manifesto isn't theoretical. It summarizes decades of production experience:

- **Jackson library:** 15+ years of supporting flexibility → became too complex, developed CVE vulnerabilities
- **Gson library:** Same story → same vulnerabilities
- **Protocol Buffers (Google):** Designed strict for exactly these reasons → proven at scale for 20+ years
- **Rust ecosystem:** Serde library → strict typing, no reflection → highly reliable
- **gRPC:** Strict schema required → industry moving this direction
- **Industry trend:** Ecosystem learning to value strictness, not flexibility

### Why Flexibility Causes Problems

**The "Flexible Parsing" Trap:**

Every time you add support for another format variation:
- Parser code grows
- Test coverage becomes incomplete
- Edge case interactions become impossible to track
- Performance degrades (isinstance checks multiply)
- Bugs hide in the branches

This is not opinion. This is observable in every major parsing library.

### Why Strictness Prevents Bugs

| Issue | Flexible Approach | Strict Approach |
|-------|-------------------|-----------------|
| Type confusion | String `"10"` accepted as number, calculation error | Error at boundary, never reaches calculation |
| Date format variance | 15+ formats supported, 800-line parser, DST bug | One format, simple parser, no bugs |
| Backwards compat debt | v1, v2, v3, v4 all need support, exponential complexity | Clear versioning, v1 sunsets, v2 clean |
| Security | Flexible typing enables RCE | No polymorphic typing, no RCE surface |
| Silent failures | Array `[x]` vs `x` accepted, later bugs | Type error caught immediately |

---

## FAQ

### Q: This seems too strict. What if I need flexibility?

**A:** Then this manifesto isn't for you. Use Jackson or Gson—they're proven and handle flexibility well.

This manifesto is for the 99% case where strictness catches more bugs than flexibility provides value.

---

### Q: What about backwards compatibility with existing APIs?

**A:** Great question. The answer is: use adapters.

**Inside your system:** Strict format
**At system boundary:** Adapter converts messy external format to strict internal format

This way:
- External API can send whatever it wants
- Your internal system always sees clean, predictable format
- Easier to upgrade external dependencies

**Better long-term:** Negotiate with API owners to support versioning instead of flexible formats. Mutual benefit.

---

### Q: Backwards compatibility is how real APIs work. You're wrong.

**A:** Backwards compatibility IS how real APIs work. And that's the problem this manifesto identifies.

APIs evolved "pragmatically" to support more formats. Result: unmaintainable, buggy, vulnerable.

Better approach for APIs: Use versioning. Not flexibility.

```
/api/v1 → frozen format, never changes
/api/v2 → improved format, strict
Clients migrate on schedule
Old version sunsets after 12 months
Clean separation.
```

This is how Google, Amazon, Stripe, and other API leaders actually do it.

---

### Q: What if I need flexibility for legitimate reasons?

**A:** Almost always, what feels like "need flexibility" is actually "design wasn't clear."

Examples:

**"Sometimes it's a string, sometimes a number"**
→ Problem: Design ambiguous
→ Solution: Choose one format, version if you need to change it
→ Not solution: Accept both (complexity explodes)

**"Some clients send null, others omit the field"**
→ Problem: API contract wasn't explicit
→ Solution: Document "field must be omitted if not provided"
→ Not solution: Accept both (semantics become unclear)

**"Legacy integration sends different format"**
→ Problem: Outside system has different design
→ Solution: Use adapter layer
→ Not solution: Accept multiple formats in main parser

---

### Q: How is this different from JSON Schema?

**A:** JSON Schema validates structure. This manifesto validates structure AND enforces type safety at the code level.

They're complementary:
- **JSON Schema:** Says what fields must exist
- **Strict JSON:** Says what types they must be AND generates safe code

Use both.

---

### Q: Won't this limit adoption?

**A:** Yes. Intentionally.

This is not "JSON for everyone." It's "JSON for data transfer in controlled environments where safety and reliability matter."

If you need maximum flexibility: Jackson, Gson, JSON5—all great choices.

This manifesto is for when reliability > flexibility.

---

### Q: Can I contribute if I disagree with a rule?

**A:** Yes. Open an issue. We'll discuss.

But understand: the bar for changing rules is high. They need to solve real problems for the 99%, not edge cases for the 1%.

Every rule emerged from production incidents. Propose changes only with equivalent evidence.

---

## Get Involved

### Sign the Manifesto

You believe JSON should be simple? Data transfer should be predictable? Invalid states should be impossible?

**Add your name and organization:** [SIGNATORIES.json](./SIGNATORIES.json)

### Contribute

1. **Found a problem?** Open an issue with your production incident
2. **Want to translate?** Create a PR in `translations/[language-code]/`
3. **Have a reference implementation?** Link it in `IMPLEMENTATIONS.md`
4. **Using this?** Add your company to the "In the Wild" section

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

### Share

- **GitHub:** Star, fork, contribute
- **Social:** Tag us: "I'm adopting Strict JSON principles"
- **Blog:** Write about your experience
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
- Production incidents from across the industry
- Decades of lessons learned from Jackson, Gson, Protocol Buffers, and other serialization frameworks

---

**JSON is just a data transport format. Let's treat it that way. Strictly.**

*Manifest your support. Sign below. Build implementations. Change your projects. Change the industry.*

---

Questions? Ideas? [Open an issue](../../issues/new).

Found a problem? [Report it](../../issues/new).

Want to help? [Contribute](./CONTRIBUTING.md).
