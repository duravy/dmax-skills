<!-- TEMPLATE: This is the blank structure. Copy it to review-criteria.md and fill every
     section in for YOUR project and language. The multipass-review skill loads
     review-criteria.md (NOT this file) as the single source of truth for a review.
     See review-criteria-example-springboot.md for a fully worked example. -->

# Code Review Criteria — <PROJECT NAME>

> **How this file is used:** the `multipass-review` skill embeds this file verbatim into
> every fresh review agent's prompt. Vague criteria → false positives → developers stop
> trusting *all* findings. Be **categorical**, not subjective. Define what to **skip** as
> explicitly as what to **flag**.

## 1. Tech stack & context

- **Language(s) / runtime:** <e.g. Java 21, Node 20>
- **Frameworks:** <e.g. Spring Boot 3.x>
- **Architecture:** <e.g. microservices, layered, event-driven>
- **What a reviewer must know:** <project-specific conventions, key invariants, "we always do X">

## 2. Severity levels (with a concrete code example each)

Fill in a real example from your stack for every level. Low is defined **only so it is never reported.**

| Level | Definition | Concrete example (your language) | Action |
|-------|------------|----------------------------------|--------|
| **Critical** | Data loss or security vulnerability | `<example>` | Report |
| **High** | Functional bug affecting users | `<example>` | Report |
| **Medium** | Quality issue likely to cause a future bug | `<example>` | Report |
| **Low** | Style / convention / formatting | `<example>` | **Never report** (linters handle this) |

## 3. INCLUDE — report these (categorical)

List concrete, checkable categories. Each should be objectively decidable, not a judgment call.
Add a one-line example per category.

- <category> — <one-line example>
- <category> — <one-line example>
- ...

## 4. EXCLUDE — never report these

Be just as explicit here. Anything a linter/formatter/type-checker catches belongs here.

- <category> — <why it's excluded>
- ...

## 5. Per-category examples (anchor the agent with 2–3 each)

For your top INCLUDE categories, give 2–3 short before/after or good/bad snippets so the
review agent classifies consistently. Few-shot anchoring measurably lowers false positives.

### <Category A>
```text
BAD:  <snippet>
WHY:  <one line>
```

### <Category B>
```text
BAD:  <snippet>
WHY:  <one line>
```

## 6. Project-specific rules / known false-positive traps

- <"Do NOT flag X — it's intentional here because ...">
- <"Always flag Y in this repo because ...">
