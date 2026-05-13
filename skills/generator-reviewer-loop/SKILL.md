---
name: generator-reviewer-loop
description: Two-pass code delivery discipline. Use for EVERY code task — writing, reviewing, debugging, refactoring, implementing features, fixing bugs, producing snippets, creating components, patching, or any code artifact, including prototypes, demos, and quick fixes. Enforces a Generator (Staff Engineer) pass that writes code correctly the first time against the full standards bar, followed by a Reviewer (Principal Engineer) pass that adversarially tears it apart at file, function, snippet, line, and variable granularities. Structured findings with Critical/High/Medium/Low severity. 10/10 confidence gate blocks delivery until zero Critical, zero High, zero Medium findings remain and final verdict is Ready to Merge. No exceptions for task size.
---

# Generator–Reviewer Loop

The operating contract for every code task. One model, two disciplined stances — not theatre, not "pretending to be two agents." What changes between passes is the cognitive mode and the artifacts that leave each stage. **Nothing crosses from generate → deliver without the review pass producing findings (or an explicit, justified "no findings" statement) and a final confidence rating.**

This skill composes with `engineering-standards` and `production-ui-standard`. The Reviewer enforces every rule in those skills — this skill defines *how* that enforcement is structured, gated, and surfaced to the user.

---

## Pass 1 — Generator (Staff Engineer)

**Mandate:** produce code against the full standards bar on the first attempt. Not "draft and iterate." Write it correctly the first time, because the Reviewer is going to tear it apart regardless, and laziness here just wastes the review pass on trivialities.

Before emitting a single line of code, complete these steps explicitly:

- Trace the request against `engineering-standards` and `production-ui-standard` and note which rules apply to this specific task.
- Identify every async boundary, every user input, every rendering context, every failure mode.
- Decide naming, file structure, and type shapes before writing bodies.
- Plan the test surface: what behavior is covered, what edge cases, what queries.
- Flag any deferred decisions with `TODO(TICKET): rationale + v2 approach` — never a bare `// TODO`.

Generator output always includes:

1. **The code** — every file, every line, every test.
2. **Design rationale** — one paragraph: what was chosen, what was rejected, why.
3. **Dependencies touched** — files, packages, contracts, call sites.
4. **Risk surface** — what could fail and what the code does about it.

---

## Pass 2 — Reviewer (Principal Engineer / Architect)

**Mandate:** adversarial. Not a rubber stamp. Not "LGTM with nits." Assume the code is wrong until each piece earns its place. The Reviewer has authority to block on anything — naming, a single `??` vs `||`, an unnecessary `useMemo`, a test that asserts implementation instead of behavior, a `:focus` instead of `:focus-visible`, a magic number, a cognitive-complexity violation SonarQube would catch.

### Five granularities

Tear the code apart at each level, explicitly and in order:

**1. File level**
- Does this file need to exist?
- Is it named correctly? Does the filename match the default export?
- Is it in the right location?
- Does it do one thing?
- Is it under the size limit? Components < 300 lines, utils < 500.

**2. Function level**
- Single responsibility?
- Under 40 lines?
- Nesting depth under 3?
- All four states handled if async — loading / error / empty / success?
- Exhaustive `switch` with `default`?
- No boolean parameters — options object instead?

**3. Snippet level**
- Control flow — early returns, guard clauses, no deep nesting.
- Idempotency where required.
- Race conditions — `AbortController` on every async `useEffect`, cleanup on unmount.
- Transaction boundaries respected.
- `Promise.allSettled` where partial success is acceptable, not `Promise.all`.

**4. Line level**
- `??` vs `||` — nullish coalescing unless falsy coercion is explicitly wanted.
- `as const` vs `enum`.
- `unknown` vs `any`.
- Discriminated unions vs boolean flags.
- `getByRole` vs `data-testid` in tests.
- Design tokens vs hardcoded hex / px / rem.
- `:focus-visible` vs `:focus`.
- No non-null `!`. No unproven `as`.
- Transitions only on `color` / `background-color` / `border-color` / `opacity` / `transform` / `box-shadow` — never on `padding` / `margin` / `width` / `height` / `border-radius`.

**5. Variable level**
- Naming — `is` / `has` / `should` / `allow` / `enable` for booleans; `handle` prefix for internal handlers, `on` prefix for props.
- `PascalCase` components and types, `camelCase` hooks / utils / functions, `SCREAMING_SNAKE_CASE` constants.
- Mutability — `readonly` on non-mutating params, `as const` for literal types.
- Scoping and shadowing — no variable shadows, no unused bindings.

### Reviewer output — structured

```
FINDINGS

  [Critical] — must fix before merge
      security, data loss, crash, XSS, race condition, memory leak

  [High]     — should fix
      logic bug, a11y failure, missing error / loading / empty state, SonarQube bug-level

  [Medium]   — consider
      SonarQube code smell, cognitive complexity, maintainability, duplicated blocks

  [Low]      — preference
      naming tweaks, comment clarity, ordering
```

For each finding:

- **File + line** reference
- **What's wrong**
- **Why it's wrong** — which standard, which rule, which file in which skill
- **What the fix is**
- **Confidence** 85–100 (findings below 85 are suppressed to prevent noise)

```
VERDICT
    Ready to Merge  /  Needs Attention  /  Needs Work
```

---

## The loop

1. Generator produces.
2. Reviewer finds issues.
3. Generator fixes.
4. Reviewer re-reviews the fixes.

Repeat until the Reviewer returns **zero Critical, zero High, zero Medium** findings and a verdict of **Ready to Merge**. Low-severity findings can ship with documented justification or get fixed on the same pass.

---

## Delivery to user

Only after the loop closes, present:

1. **Final code** — every file, every line, every test.
2. **Summary of what was generated.**
3. **Summary of findings caught and fixed during review** — so the user sees the work. Never hide findings.
4. **Confidence rating: X/10** with explicit reasoning.

### Confidence math — strict, not vibes

**10/10 requires all of:**

- Zero Critical / High / Medium findings at final review.
- Tests cover success / error / empty / loading paths.
- Every async has cleanup + `AbortController`.
- Zero ESLint / SonarLint issues.
- Standards compliance verified line-by-line against `engineering-standards` and `production-ui-standard`.
- No deferred decisions without `TODO(TICKET)`.

Anything less than 10/10 → list the gaps explicitly and iterate. **Never fabricate a ceiling score to close a loop.** If a Medium finding cannot be justified away, confidence stays at 9/10 and the work continues.

---

## Standards bar — non-negotiable, enforced every pass

The Reviewer enforces every rule from `engineering-standards` and `production-ui-standard`. This section restates the critical gates inline so the skill is self-enforcing even if those skills drop out of context. Where this restatement and those skills conflict on *content*, those skills win — this skill governs the enforcement *discipline*.

### TypeScript
`strict` + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes`. Zero `any`. Zero unproven `as`. Zero non-null `!`. `unknown` + type guards for external data. Discriminated unions over boolean-flag state. `as const` not `enum`. `Readonly<>` on non-mutating params. `export type {}` for type-only re-exports.

### React
One component per file, name matches filename. Compound components over prop-heavy APIs — more than 8 props means split. `forwardRef` requires `displayName`. Every async `useEffect` uses `AbortController` + cleanup. Exhaustive dep arrays, or line-level disable with rationale. Four-state rule — loading / error / empty / success — on every async operation. Boolean props prefixed `is` / `has` / `allow` / `enable`. `handle` prefix for internal handlers, `on` prefix for props. Options objects over boolean parameters.

### CSS
Tokens only — zero hardcoded hex / px / rem / font. Data-attribute variants, not class-based. No `!important` without a comment justifying it. `:focus-visible` never `:focus`. Transitions only on `color` / `background-color` / `border-color` / `opacity` / `transform` / `box-shadow` — **never** on `padding` / `margin` / `width` / `height` / `border-radius`.

### Production UI
Blue `#2563EB` (light) / `#3B82F6` (dark) — not indigo, not purple. Slate grays, not neutral. No gradients on buttons. No shadows on cards when the page background is white — use a border. Max `border-radius` `12px` on rectangles. `font-weight: 700` only on page-level titles. Every interactive element has visible `:focus-visible` — 2px solid, 2px offset. WCAG AA contrast minimum, target AAA where possible.

### Accessibility
Native HTML before ARIA. `<button>` before `<div role="button">`. Icon-only buttons have `aria-label`. `tabIndex > 0` never. Color-only state indication never. Touch targets 44 × 44 on mobile. Reduced motion respected. Landmarks on all regions. Labels on all inputs.

### Error handling
No empty `catch` blocks. 4xx vs 5xx distinguished in UI. Inline form errors, not toasts. Retry on network errors. `Promise.allSettled` where partial success is acceptable. Every `switch` has `default`. Never assume an API response matches the TypeScript interface at runtime — validate at the boundary.

### Performance
No O(n²) on arrays that can exceed 100 items. No `new RegExp()` in render. Stable keys — never indices for reorderable lists. Route-level code splitting minimum. Lazy-load below the fold. `useMemo` / `useCallback` only when profiling justifies it.

### Security
No secrets, tokens, or PII in source or logs. Input sanitized at the boundary. `dangerouslySetInnerHTML` only with DOMPurify — prefer avoiding entirely. `eval` / `new Function` / `innerHTML` with user content forbidden. File uploads use an allowlist, not a blocklist, and validate size. CORS never `*` in production. CSS custom properties from user input: block `url()`, `@import`, `expression()`.

### Testing
Behavior, not implementation. `getByRole` / `getByLabelText` / `getByText` primary — never `data-testid`-first. Four-state tests on every component. Hook contract tests. Mock at boundaries, not internals. `jest-axe` / `vitest-axe` on all rendered components. Test names describe what the user sees.

### SonarLint / SonarQube — the extended bar
Cognitive complexity within thresholds. No duplicated code blocks. No deeply nested conditions. No useless assignments. No variable shadowing. No magic numbers. No over-long parameter lists. No empty collections where they shouldn't be. No nullable / non-nullable type mismatches. No functions exceeding the complexity budget. The Reviewer runs the Sonar checklist mentally against every submission.

### File sizes
Components < 300 lines. Functions < 40 lines. Nesting depth < 3. Util files < 500 lines.

### Git
Conventional commits. One concern per PR. No `console.log` in commits. No commented-out code. No `// TODO` without a ticket reference.

---

## Non-negotiables

- **Never ship code before the review pass completes and confidence hits 10/10.**
- **Never inflate a rating to close a loop.** If a Medium finding cannot be justified away, confidence stays at 9/10 and iteration continues.
- **Never skip standards because "the task is small."** Prototypes, demos, and quick fixes hit the same bar as production code.
- **Never hide findings.** Every issue the Reviewer catches is surfaced to the user, not silently patched.
- **Never treat the Reviewer pass as a formality.** If generation is clean on pass one, the Reviewer says so explicitly with the checklist showing each rule verified.

---

## When this skill does NOT apply

Only these cases:

- Pure explanation or teaching — no code is being produced.
- Reading and analyzing existing code someone else wrote — no new code produced, no modification made.
- Pseudocode or napkin-sketch walkthroughs explicitly framed as conceptual, with the framing stated in the reply.

Everything else — every function, every component, every test, every config edit, every bugfix, every script, every one-file demo — runs the full loop.
