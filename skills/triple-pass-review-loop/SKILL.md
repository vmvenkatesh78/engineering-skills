---
name: triple-pass-review-loop
description: Three-pass code delivery discipline — Generator, Reviewer, Reviewer-of-Reviewer. Use for EVERY code task — writing, reviewing, debugging, refactoring, implementing features, fixing bugs, producing snippets, creating components, patching, or any code artifact, including prototypes, demos, and quick fixes. Generator (Staff Engineer) writes against the full standards bar. Reviewer (Principal Engineer) tears it apart adversarially at six granularities. Reviewer-of-Reviewer (Founding Architect) audits the Reviewer's work — catches missed findings, severity errors, rationalizations, and process violations. 10/10 confidence gate blocks delivery until zero Critical, zero High, zero Medium findings remain AND all proposed fixes are verified. No exceptions for task size.
trigger: Always active for any code generation, review, debugging, or refactoring task. Mandatory for production code changes, multi-file fixes, architectural decisions, high-blast-radius changes.
enforcement: mandatory
---

# Triple-Pass Review Loop

Two-pass review (Generator + Reviewer) catches obvious problems. It misses the subtle ones — the false positives the Reviewer asserts without verification, the severity errors that slip through, the missing findings the Reviewer's blind spots create, the process violations where fixes get proposed without verification gates.

Three-pass review adds a meta-pass (Reviewer-of-Reviewer / RR) that audits the Reviewer's work specifically. The third pass exists because the Reviewer has its own failure modes that need their own review.

This skill replaces ad-hoc review with structured discipline. Apply to every code task without exception.

---

## Why Three Passes

Each pass has commitment bias appropriate to its role:

- **Pass 1 (Generator)** wants the solution to work. Optimistic by nature.
- **Pass 2 (Reviewer)** wants to find problems. Adversarial — but can be inconsistent (too harsh on some dimensions, too lenient on others, systematic in its blind spots).
- **Pass 3 (Reviewer-of-Reviewer)** examines Pass 2's behavior. Catches things like:
  - Findings asserted without verification
  - Severity rankings that don't survive scrutiny
  - Rationalizations dressed up as analysis
  - Granularities Pass 2 skipped
  - Missed findings (the things you'd kick yourself for later)
  - Process gaps (no verification gate between proposed fix and delivery)

Without Pass 3, the failure mode is: Pass 2 says "looks good" and you ship. Pass 3 catches "did you actually verify the fix? did you check the other rendering contexts? did you address the architectural smell or just patch the symptom?"

---

## Pass 1 — Generator (Staff Engineer)

### Discipline

- **Diagnose first.** Never pattern-match. State the problem in your own words. Identify the root cause before designing the solution.
- **Design against the full standards bar.** ESLint, SonarLint, SonarQube refactor tier, accessibility, security, performance, type safety, test coverage. No carve-outs for "quick fixes."
- **Implement with judgment.** No magic numbers. Named constants. Errors never swallowed silently. Clear naming. JSDoc where types alone don't tell the story.
- **State assumptions explicitly.** What did you assume about the runtime, data shape, error conditions, concurrency? Surface every assumption — Pass 2 will test them.

### Output format

```
## Pass 1 — Generator

### Diagnosis
[The problem in your own words. Root cause, not symptom.]

### Approach
[Design choices and the alternatives rejected.]

### Implementation
[The actual code, or pointer to delivered files.]

### Assumptions
- [Each explicit assumption that Pass 2 should challenge.]
```

### Anti-patterns to avoid

- **Pattern matching without diagnosis.** "This looks like the X bug we saw before, applying the X fix" — without confirming it actually IS the X bug.
- **Skipping the design step.** Going straight from problem to code.
- **Hiding judgment calls.** If you made a choice between two approaches, name it.
- **Burying assumptions in comments instead of stating them.**

---

## Pass 2 — Reviewer (Principal Engineer)

### Discipline

Adversarially review Pass 1 across SIX granularities. At each granularity, ask: "What's wrong with this?" Don't ask "is this OK?" — that biases toward acceptance.

### The Six Granularities

#### Word-level
- Variable names: do they say what the value is, or what type it has?
- Comments: do they explain why, or just restate what?
- Identifiers: any single-letter names, magic strings, untyped enums?

#### Line-level
- Each line: does it do what it appears to do?
- Off-by-one: are loop bounds correct?
- Null/undefined handling: optional chaining where it should be, defaults where they should be?
- Falsy traps: `||` vs `??`, truthy short-circuits that strip empty strings/zero/false?

#### Snippet-level
- Logic blocks: does the control flow handle every case?
- State transitions: are intermediate states valid?
- Async sequencing: are awaits in the right places? Is order preserved when it should be?
- Error propagation: do throws bubble up correctly? Are catches doing the right thing?

#### Function-level
- Single responsibility: does this function do one thing?
- Side effects: are they declared in the contract or hidden?
- Parameter design: are required vs optional clear? Defaults sensible?
- Return values: typed consistently? Are error cases reflected in the return type?

#### File-level
- Cohesion: do all exports belong here?
- Imports: are they minimal? Any transitive chains that drag heavy modules in?
- Module boundaries: does this file violate layer boundaries?
- Public API surface: too wide? Could be reduced?

#### Architecture-level
- Does this fix address the root cause or only the symptom?
- Are there codebase patterns this implementation diverges from?
- What changes at 10x scale? 100x?
- Is there a smell that this fix exposes — even if fixing the smell is out of scope?

### Severity tiers

| Tier | Definition | Action |
|---|---|---|
| **Critical** | Code is broken, will fail at runtime, ships a bug | BLOCK delivery |
| **High** | Real concern that will cause future bugs or rework | BLOCK delivery |
| **Medium** | Code works but quality bar not met | BLOCK delivery |
| **Low** | Hygiene, nice-to-have, taste | NON-BLOCKING (note for later) |

Findings get severity at first pass. Pass 3 may upgrade or downgrade with rationale.

### Output format

```
## Pass 2 — Reviewer

### Word-level
**W-1 [Severity]:** [Finding]

### Line-level
**L-1 [Severity]:** [Finding]
**L-2 [Severity]:** [Finding]

### Snippet-level
**S-1 [Severity]:** [Finding]

### Function-level
**F-1 [Severity]:** [Finding]

### File-level
**FL-1 [Severity]:** [Finding]

### Architecture-level
**A-1 [Severity]:** [Finding]
```

If a granularity has no findings, write "None — verified" rather than skipping it. Pass 3 needs to see that the granularity was considered.

### Anti-patterns to avoid

- **Asserting findings without verification.** "L-3: this regex is wrong" — but did you actually trace the regex against the input?
- **Skewing severity.** Calling everything Critical, or downplaying real issues to Low.
- **Skipping granularities.** Architecture-level review especially. Easy to skip; usually has the highest-leverage findings.
- **Vague findings.** "This could be cleaner" — say what specifically, with what fix.
- **Proposing fixes without explaining why the original was wrong.**

---

## Pass 3 — Reviewer-of-Reviewer (Founding Architect)

### Discipline

Pass 3 is **not another review of Pass 1's code.** It's a review of **Pass 2's review.** The question is: did Pass 2 do its job correctly?

### The Six Audit Dimensions

#### 1. Were the granularities actually covered?
For each of the six granularities in Pass 2, was it given real attention or skipped?

| Granularity | Findings count | Quality assessment |
|---|---|---|
| Word | n | Skim or substantive? |
| Line | n | Skim or substantive? |
| Snippet | n | Skim or substantive? |
| Function | n | Skim or substantive? |
| File | n | Skim or substantive? |
| Architecture | n | Skim or substantive? |

If any granularity has 0 findings AND wasn't explicitly verified as "no issues found," that's a Pass 2 omission. Flag it.

#### 2. Did the Reviewer rationalize anything away?
Look for findings that Pass 2 raised then dismissed mid-review. Each dismissal must be VERIFIED, not asserted.

Examples:
- "L-3 was Critical, then I checked and downgraded to non-issue" — did Pass 2 actually trace the code path? Or did it just convince itself?
- "S-2 was flagged as Medium for fragility" — should it be High? Fragility is critical-adjacent.

If verification was real, accept the downgrade. If it was rationalization, restore the original severity.

#### 3. Did the Reviewer miss findings?
Pass 2 has blind spots. Pass 3 actively looks for what Pass 2 didn't see:
- New transitive imports introduced by the change
- Existing tests that the change might invalidate (mock contracts, fixture assumptions)
- Other rendering contexts the fix needs to work in (overflow menus, poppers, portals, conditional wrappers)
- DOM structure assumptions made without verifying actual rendered DOM
- Regex changes that tighten existing matching while extending it
- Behavior changes that affect downstream consumers
- Process violations (fixes proposed without verification, prod changes without test changes, etc.)

If Pass 3 finds a missed finding, add it with severity. Don't be precious.

#### 4. Were proposed fixes verified?
For every fix Pass 2 proposed, did Pass 2 verify it actually fixes the problem AND doesn't break something else?

This is the most common gap. Pass 2 says "fix is to add X." Pass 3 asks: "did you run it? did you check the side effects? did you confirm the fix matches the original intent?"

Verification means EITHER:
- Code was run and observable behavior matched expectations, OR
- Trace through the code path was performed and the trace was captured (not just claimed)

If neither happened, the fix is unverified. Block delivery until verified.

#### 5. Were process gates respected?
Did delivery skip verification steps?

Common process violations:
- Production code changes and test changes shipped together with no verification step in between
- Mock factories updated for one test file but not its sibling files (cross-file consistency missed)
- Refactor delivered without running existing test suite first
- Fix delivered without confirming the fix doesn't break other consumers
- File replacement assumed to have happened without verifying disk state

For each process gap: name it, explain the harm, propose the gate.

#### 6. Did the Reviewer propose remediation for findings?
Every finding should have either:
- A concrete fix proposal, OR
- An explicit "out of scope, document and move on" with rationale

Findings without remediation are just complaints. Flag any.

### Output format

```
## Pass 3 — Reviewer-of-Reviewer

### Granularity coverage
[Table — were all six dimensions covered substantively?]

### Rationalizations check
**RR-N:** [Did Pass 2 dismiss anything that shouldn't be dismissed? Was a downgrade verified or rationalized?]

### Missed findings
**M-N [Severity]:** [Finding Pass 2 missed, with severity]

### Verification audit
**Fix [Pass 2 finding ID]:** [Did Pass 2 verify the fix? Yes/No. If no, what's the verification step?]

### Process violations
**P-N:** [Process gap, harm, required gate]

### Remediation completeness
[Any findings without remediation? If so, list them.]

### Final verdict
- Critical: N (must be 0 to ship)
- High: N (must be 0 to ship)
- Medium: N (must be 0 to ship)
- Low: N (acceptable)
- **Status:** Ready to Merge / Blocked — [reason]
```

### Anti-patterns to avoid

- **Performing Pass 2 again instead of auditing Pass 2.** Pass 3 is meta. If you find yourself reviewing the original code's variable names, you're doing Pass 2.
- **Rubber-stamping Pass 2.** "Looks thorough, no notes" without checking the dimensions.
- **Adding new findings indefinitely.** Pass 3 has a fixed scope: audit Pass 2. If Pass 2 missed an entire category of issues, surface it ONCE — not in every Pass 3.
- **Failing to verify Pass 3's own findings.** If RR raises a critical finding that doesn't survive scrutiny, downgrade with rationale. Pass 3 isn't infallible.

---

## Verification Gates

The triple-pass loop is gated by actual verification, not confidence.

### Gate 1: Pass 2 doesn't trust Pass 1
Pass 2 runs against Pass 1's output regardless of Pass 1's confidence. Pass 1 cannot self-clear.

### Gate 2: Pass 3 doesn't trust Pass 2
Pass 3 audits Pass 2's findings AND non-findings. Pass 2 cannot self-clear.

### Gate 3: Delivery doesn't trust any of them without actual verification
Even if Pass 3 says "Ready to Merge":
- If the fix involves running tests, run them. Don't ship on assertion alone.
- If the fix involves multiple files, verify all are saved correctly to disk (the macOS case-insensitive fs trap, the IDE-not-saving trap).
- If the fix touches production, confirm it's the version on disk, not a stale buffer.

### Gate 4: Pass 3 findings are subject to verification
Pass 3 isn't a higher authority — it's another set of eyes. If a Pass 3 finding can't be reproduced or doesn't hold up to scrutiny, downgrade or dismiss with rationale.

---

## The 10/10 Confidence Gate

Delivery is blocked until ALL of the following:

- ✅ Pass 1 complete with diagnosis, approach, implementation, assumptions stated
- ✅ Pass 2 complete with all six granularities considered, severities assigned
- ✅ Pass 3 complete with all six audit dimensions executed
- ✅ Zero Critical findings remain
- ✅ Zero High findings remain
- ✅ Zero Medium findings remain
- ✅ Every proposed fix verified (run tests, traced code path, or equivalent)
- ✅ No process gaps unaddressed
- ✅ Pass 3 verdict is "Ready to Merge"

Low findings can ship; everything else blocks.

If the user wants to ship with open Critical/High/Medium findings, they MUST explicitly override with rationale ("shipping with M-2 unresolved because [reason]"). Don't assume override.

---

## Loop Termination

The loop is **fixed-depth at three passes.** There is no Pass 4 by default.

If Pass 3 finds Critical or High issues, Pass 1 starts over (fix issues, then full triple pass on the new version). This is a NEW loop, not Pass 4.

The terminator is: Pass 3 produces "Ready to Merge" verdict OR explicit user override of remaining findings.

---

## Examples

### Example A: Real-world Pass 2 finding

```
## Pass 2 — Reviewer

### Line-level
**L-1 [Critical]:** Mock factory at line 49 lists three exports but the production
hook also imports `normalizeSignedUrlResponse`. After Pass 1's production fix
added the call site, this mock returns undefined for that export. Calling
`undefined(response)` throws, hook's try/catch eats it, all upload-path tests
in this file fail silently. Verified by tracing imports and the failure pattern
in the M-3 combined run.

**L-2 [Medium]:** Optional chaining on line 26 (`name?.split('.').pop()`) doesn't
default to empty string. For undefined input, `.pop()` is called on undefined.
Add `?? ''` to match the production behavior.
```

The finding has: severity, what's wrong, why it matters, evidence (verified by tracing), and a concrete fix. That's a substantive Pass 2.

### Example B: Real-world Pass 3 audit

```
## Pass 3 — Reviewer-of-Reviewer

### Granularity coverage
| Granularity | Pass 2 covered | Quality |
|---|---|---|
| Word | W-1, W-2 | Surface only |
| Line | L-1, L-2, L-3 | L-3 false positive (downgraded after verification) |
| Snippet | S-1, S-2 | S-2 most important — fragility upgrade candidate |
| Function | F-1 | Real but minor |
| File | FL-1, FL-2, FL-3 | FL-1 is the most important PROCESS finding |
| Architecture | A-1, A-2 | A-1 is critical codebase smell |

### Rationalizations check
**RR-1:** L-3 was flagged Critical then downgraded. I verified the downgrade —
both regex paths produce identical output for the no-dot case. Verification was
real, not rationalization. Downgrade stands.

**RR-2:** S-2 was Medium ("fragile, future-proofing"). Severity is too low.
Fragility was the recurring failure mode in this thread — every M-3 retry hit
new chain entries. UPGRADE S-2 to High.

### Missed findings
**M-1 [Low]:** Pass 2 didn't audit deleted console.logs in the dead-code removal.
Worth noting in commit message even if not blocking.

**M-3 [High]:** Pass 2 confirmed individual suites green but didn't run the
combined M-3 verification. That's the actual final gate. Combined run could
expose interaction issues. BLOCK delivery until M-3 runs green.

### Verification audit
**Fix L-1:** Pass 2 added `normalizeSignedUrlResponse` to mock factory. Verified
by re-running the edge file standalone — 29/29 passed. Verified.

**Fix L-2:** Pass 2 changed `?.split` to `?.split ?? ''`. NOT verified — Pass 2
asserted the fix without running tests. Block until verified.

### Process violations
**P-1:** Pass 2 proposed BOTH a production code change AND a test change in one
delivery. No verification step between them. If the production change broke
something, the test change would mask it. Required gate: production change → run
existing tests → confirm green → THEN apply test changes.

### Final verdict
- Critical: 0
- High: 2 (S-2 upgraded; M-3 added)
- Medium: 4
- Low: 6
- **Status:** Blocked — M-3 combined run required, L-2 fix needs verification.
```

That's a substantive Pass 3 — it caught a severity error, flagged a missed finding, identified a process violation, and blocked delivery for the right reasons.

---

## Anti-Patterns Across All Three Passes

- **Skipping passes for "small" tasks.** Every code task. No exceptions. The five-minute fix is exactly where the worst bugs hide.
- **Treating passes as theater.** "I did Pass 2" without actually finding anything substantive isn't Pass 2. Same for Pass 3.
- **Bundling fixes without verification gates.** Production change + test change + refactor all in one delivery. Pass 3 will catch this. Don't make it work harder.
- **Letting Pass 3 become Pass 2-redux.** If Pass 3 is just rereviewing the code, it's not doing its job.
- **Using "I'm confident" as a substitute for verification.** Confidence is not a gate. Test runs are.
- **Allowing severity drift toward Low.** Pass 2 + Pass 3 calibration: Critical/High/Medium are the meaningful tiers. Low is for taste. If everything ends up Low, the review didn't bite.
- **Single-author triple-pass theater.** Same person doing all three passes can blur them. Treat each pass as a different role with different commitments. Out loud. With explicit framing ("now I'm Pass 2" / "now I'm Pass 3").

---

## Operating Principle

The discipline of three passes works because each pass has a different job:

- Pass 1: build it right
- Pass 2: tear it apart at every granularity
- Pass 3: audit Pass 2's work for what it missed

When you skip a pass, you ship the bugs that pass would have caught. The three-pass discipline is a constraint that produces better code by design, not by accident.

Apply to every code task. No exceptions.
