# Contributing to Strict JSON Manifesto

This manifesto lives by community contribution. We welcome all forms of help — whether you're fixing typos, proposing new rules, translating, or building implementations.

## Table of Contents

- [Ways to Contribute](#ways-to-contribute)
- [Getting Started](#getting-started)
- [Pull Request Process](#pull-request-process)
- [Translation Guidelines](#translation-guidelines)
- [Proposing Rule Changes](#proposing-rule-changes)
- [Code of Conduct](#code-of-conduct)

---

## Ways to Contribute

### 1. Translate

Help us reach a global audience. Create a new language directory:

```
translations/[language-code]/
├── README.md
├── RULES.md
├── CONTRIBUTING.md
```

**Supported languages (in progress):**
- 🇹🇷 Turkish (Türkçe)
- 🇬🇧 English (Original)
- 🇩🇪 German (Deutsch) — Help wanted
- 🇫🇷 French (Français) — Help wanted
- 🇮🇹 Italian (Italiano) — Help wanted
- 🇪🇸 Spanish (Español) — Help wanted
- 🇯🇵 Japanese (日本語) — Help wanted

### 2. Share Your Experience

Help us learn how these principles work in the real world:

- **Success story**: "We adopted Strict JSON and reduced NullPointerExceptions by 80%"
- **Challenge**: "Rule X is hard to implement in our language"
- **Question**: "How should Rule Y apply to our use case?"

Open an issue with:
- Your organization (optional)
- Principle number (if applicable)
- Your experience (2-3 paragraphs)

### 3. Report Issues

Found problems? Help us improve:

- **Contradiction**: "Rule 1 and Rule 15 conflict because..."
- **Unclear rule**: "Rule X doesn't explain..."
- **Missing example**: "Rule Y needs examples in TypeScript"
- **Typo/Grammar**: "Found typo on line X"

### 4. Add Your Signature

Using these principles in your project or organization?

- **Individual**: Fork → add your entry to `SIGNATORIES.json` → PR
- **Organization**: Same process, add organization info
- **Tool/Implementation**: Link your tool and document which rules you follow

### 5. Build an Implementation

Create a tool/library following these rules in your language:

- See [IMPLEMENTATIONS.md](./IMPLEMENTATIONS.md) for guidelines
- Document which rules you enforce
- Add your tool to the "In the Wild" section
- Open a PR linking your repository

### 6. Improve Documentation

- Fix unclear wording
- Add better examples
- Improve formatting
- Clarify edge cases

---

## Getting Started

### Prerequisites

- GitHub account
- Git installed locally
- Basic Markdown knowledge

### Setup

1. **Fork the repository**
   ```bash
   git clone https://github.com/YOUR-USERNAME/strict-json-manifesto.git
   cd strict-json-manifesto
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b your-feature-name
   ```

3. **Make your changes** (see guidelines below)

4. **Test locally**
   - Read changes in Markdown viewer
   - Check links work
   - Verify formatting

---

## Pull Request Process

### Before Opening a PR

- [ ] Changes align with manifesto spirit (strict, opinionated, universal)
- [ ] No breaking changes to existing rules without discussion
- [ ] Spelling and grammar checked
- [ ] Links validated
- [ ] Examples tested/verified

### Opening a PR

1. **Descriptive title**
   ```
   ✓ Add TypeScript examples to Rule 1
   ✗ Fix typo
   ✗ Update
   ```

2. **Clear description**
   ```markdown
   ## What
   Adds TypeScript examples to Rule 1 (Numeric Values)

   ## Why
   JavaScript/TypeScript developers need language-specific guidance

   ## Changes
   - Added Table: "Numeric Types Across Languages"
   - Added TypeScript code examples
   - Validated examples work in TS 5.0+

   ## Related
   Fixes: #123
   ```

3. **Reference issues**
   - `Closes #123` (auto-closes issue)
   - `Relates to #456`

### Review Process

- Maintainers review within 1 week
- Feedback provided constructively
- Minor tweaks may be requested
- Merged once approved

---

## Translation Guidelines

### Requirements

- **Accuracy**: Translate meaning, not just words
- **Tone**: Maintain strict, opinionated, direct voice
- **Examples**: Keep JSON/code examples unchanged
- **Structure**: Mirror original document structure exactly
- **Completeness**: Translate ALL files (README, RULES, CONTRIBUTING)

### Translation Checklist

- [ ] Tone preserved (strict, no softening)
- [ ] Code examples unchanged
- [ ] Links updated to translation directory
- [ ] All headings translated
- [ ] Cultural terms adapted sensibly
- [ ] Proof-read by native speaker
- [ ] Tables formatted correctly
- [ ] No English words mixed in

### Submission

Create directory:
```
translations/[language-code]/
├── README.md
├── RULES.md
└── CONTRIBUTING.md
```

Add to main `README.md`:
```markdown
## Translations

- 🇹🇷 [Turkish (Türkçe)](./translations/tr/)
- 🇫🇷 [French (Français)](./translations/fr/)
```

---

## Proposing Rule Changes

Rules are strict because they solve real problems for the 99%. Rule changes require:

1. **Open an issue** describing:
   - Current rule and its problem
   - Proposed change
   - Why change helps (with examples)
   - Languages/implementations affected

2. **Discussion phase** (1-2 weeks)
   - Community feedback
   - Maintainers evaluate
   - Consider impact

3. **If accepted**: Create PR with:
   - Rule change
   - Updated examples
   - Updated verification checklist
   - Rationale section

### Bar for Rule Changes

**High bar** — must solve problems for 99% use case, not edge cases:

- ✓ "Rule X breaks this common pattern, here's evidence..."
- ✗ "My specific edge case doesn't work..."
- ✓ "Rule incompatible with language X, alternatives are..."
- ✗ "I prefer different approach..."

---

## Code of Conduct

We're building this manifesto to make JSON deserialization better for everyone.

### Be Respectful

- Assume good intent
- Different perspectives are valuable
- Disagree constructively

### Focus on Ideas

- "This rule might be clearer if..." instead of "Your rule is wrong"
- Back up claims with examples
- Reference real problems

### No Harassment

- No personal attacks, discrimination, harassment
- Keep discussion technical and professional

### Violations

Violations result in:
1. Warning
2. Comment removal
3. Temporary ban (1-7 days)
4. Permanent ban (if severe)

Report violations: Open private issue or contact maintainers

---

## Questions?

- **Not sure how to contribute?** Open an issue: "Help wanted: [what you want to do]"
- **Unsure about rule?** Open discussion: "Question about Rule X..."
- **Need clarification?** Comment on existing issue

We're here to help.

---

## License

By contributing, you agree that your contributions are licensed under the [CC-BY-4.0](./LICENSE) license used by this project.

Your name/organization will be listed in SIGNATORIES.json and credited appropriately.

Thank you for contributing!
