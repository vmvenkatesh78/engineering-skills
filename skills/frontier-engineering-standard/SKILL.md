# v0.2 Per-Skill Extensions — Session Planning Document

**Purpose:** Lock the 6 decisions for the per-skill extension work so the next focused session executes against this plan rather than re-debating decisions.

**Scope:** 5 extensions to existing skills (down from original 8). Each extension is a new section added to the existing skill, 300-500 words, substantively distinct from existing content.

**Total estimated work:** 3-4 hours across 1-2 sessions. Plus README rework (30-45 min) and CHANGELOG (15-20 min) and final assembly (30 min) = 4.5-6 hours total to v0.2 ship.

---

## Decision 1 — Which 5 skills get extensions

| Skill | v0.1 size | Extension? | Why |
|---|---|---|---|
| `engineering-standards` | 2,436 words | YES | Meta-skill's Section 2 explicitly references this skill; multi-year time-scale rules complement the per-line content |
| `production-ui-standard` | 3,970 words | YES | Meta-skill Sections 3-4 cut; production-ui extension carries the frontier-tier UI content |
| `runtime-behavior-review` | 2,641 words | YES | Meta-skill Sections 3-4 cut; runtime extension carries the frontier-tier runtime content |
| `technical-partner` | 3,086 words | YES | Staff-IC operating mode; directly relevant to program positioning |
| `design-partner` | 3,368 words | YES | Design at system level; complements production-ui-standard extension |
| `generator-reviewer-loop` | 1,828 words | NO | Meta-skill Section 3 (review culture) already covers what an extension would add |
| `triple-pass-review-loop` | 3,014 words | NO | Already substantial; no clear frontier-tier delta |
| `business-consultant-partner` | 3,310 words | NO | Meta-skill principles don't strongly apply; consulting work is its own discipline |

---

## Decision 2 — Substantive distinctness test per extension

Each extension MUST pass this test: content substantively distinct from existing skill content AND substantively distinct from the meta-skill's principles.

If an extension fails the test during writing, it gets cut (same discipline as Sections 3-4 of the meta-skill).

**Per-skill distinctness criteria:**

### engineering-standards extension
- Existing skill covers: per-line TypeScript discipline, React patterns, CSS architecture, accessibility rules, testing, performance budgets, code review
- Extension covers: multi-year time-scale rules. Specifically the durability ladder applied to specific TypeScript patterns (branded types as load-bearing, discriminated unions as security boundaries, exhaustiveness checks as compile-time audit), decision-context preservation patterns specific to code, refactor-resilience tests for module design
- Defensibility check: do you operate against multi-year code aging at Workhall? Yes — 5 years at Workhall, primary maintainer of wh-ui-library, 1300+ code reviews. Strong evidence.

### production-ui-standard extension
- Existing skill covers: color, typography, spacing, shadows, components, states, accessibility at the component level
- Extension covers: design-system-level concerns. Token hierarchy and architecture, governance patterns to prevent design drift over time, performance budgets at the design-token level, accessibility as build-time gate (not runtime check)
- Defensibility check: you operate as primary maintainer of wh-ui-library at Workhall. Direct operating evidence. The extension content should match design-system-at-mid-size-SaaS scale, not multi-product-surface scale.

### runtime-behavior-review extension
- Existing skill covers: self-measurement contamination, shutdown/drain races, probe interference, middleware ordering, metric exposure
- Extension covers: runtime engineering culture. SLI/SLO thinking as a design constraint, error budgets as work-prioritization signal, graceful degradation patterns, the cost of over-reliability targets
- Defensibility check: HARDEST. You operate against runtime concerns at Workhall scale, NOT billion-event scale. Extension content must explicitly stay at Workhall scale. Frame as "the runtime discipline at any scale," not "frontier-tier runtime engineering." If the extension drifts toward claiming scale, it gets cut.

### technical-partner extension
- Existing skill covers: persona definition, full-stack scope, infrastructure depth, the operating mode of architectural decisions
- Extension covers: staff-IC operating mode specifically. Cross-team influence without authority, sponsoring others' work, the "snacking avoidance" discipline applied to architecture decisions, when to write code vs when to write a document
- Defensibility check: you operate against this at staff-IC-track scope. Frame as "the discipline I work against on the path to staff," not "the operating mode I have." Honest framing protects against overclaiming.

### design-partner extension
- Existing skill covers: information architecture, interaction patterns, form design, table design, configuration UI, state design, accessibility, labeling
- Extension covers: design at system level. Design tokens as code, design-as-engineering-discipline, governance patterns, design system documentation as durable artifact, the design-engineering interface
- Defensibility check: you operate the wh-ui-library which IS a design system at mid-size SaaS scale. Direct operating evidence.

---

## Decision 3 — Writing order

Sequence chosen for momentum + risk distribution:

1. **technical-partner** — smallest extension, strong evidence, builds momentum
2. **engineering-standards** — direct meta-skill reference; lays foundation for other extensions
3. **design-partner** — strong evidence, content adjacent to production-ui-standard
4. **runtime-behavior-review** — HIGHEST RISK; tackle after rhythm is established
5. **production-ui-standard** — largest extension, capstone work

Each extension goes through both review loops (insider + outsider) before declaring complete. Same discipline as meta-skill C1-C4.

---

## Decision 4 — Word target per extension

- **300-500 words each.** Smaller than meta-skill sections; each extension is a complement, not standalone.
- **Total extension content across 5 skills: 1,500-2,500 words.** Plus the 3,616 meta-skill words = 5,116-6,116 new words for v0.2.
- **Comparable to v0.1 package growth:** the v0.1 package was 23,653 words across 8 skills. Adding 5,000-6,000 words is ~25% growth, substantial but not bloated.

---

## Decision 5 — Session structure

Two possible session structures:

**Option A — One long session (3-4 hours)**
- Write all 5 extensions in sequence
- Run review loops after each extension
- Higher cognitive load; fatigue risk on later extensions

**Option B — Two sessions (90-120 min each)**
- Session 1: technical-partner + engineering-standards + design-partner (3 extensions)
- Session 2: runtime-behavior-review + production-ui-standard (2 extensions, the at-risk and capstone)
- Lower fatigue risk; better calibration on the harder content

**Recommended: Option B.** The runtime-behavior-review and production-ui-standard extensions are the highest-stakes; saving them for a fresh session protects their quality.

---

## Decision 6 — Meta-skill protocol updates after extensions

**Decision: No updates to the protocol section.**

The protocol references skills by name (e.g., "engineering-standards"). Extensions are part of those skills, not separate artifacts. The protocol stays at skill-level granularity.

---

## After all 5 extensions ship

Remaining v0.2 work:

1. **README rework (30-45 min):** Update v0.2 framing to reference the new meta-skill and extension philosophy. Frame the operating-evidence test as the package's discipline.

2. **CHANGELOG.md (15-20 min):** v0.1 → v0.2 evolution narrative. Names what was added, what was kept the same, what discipline emerged through writing.

3. **Final assembly + repo push (30 min):** All files into the engineering-skills repo, tagged as v0.2.

4. **LinkedIn announcement (separate session):** Once shipped, one LinkedIn post per the playbook discipline. Different content angle than the file validation post.

---

## Operating evidence test — applied across all extensions

Every extension must pass this test, evaluated section by section:

**Question:** "Does this content describe a discipline you operate against, or a discipline you observe?"

- If operate: claim it confidently. State as personal operating standard.
- If observe: frame as "the discipline observable in [source]" or cut.

Sections that drift from operate to observe during writing get cut, not softened. The honesty discipline established in the meta-skill applies to every extension.

---

## Total v0.2 ship checklist (after this session)

- [ ] technical-partner v0.2 extension
- [ ] engineering-standards v0.2 extension
- [ ] design-partner v0.2 extension
- [ ] runtime-behavior-review v0.2 extension
- [ ] production-ui-standard v0.2 extension
- [ ] README rework
- [ ] CHANGELOG.md
- [ ] Final assembly + git tag v0.2
- [ ] LinkedIn announcement (post-ship)

Plus already-complete:
- [x] frontier-engineering-standard meta-skill (3,616 words)

---

## What I committed to vs what's pending

Original v0.2 Heavy plan was 12-18 hours. Current actual work:

- Meta-skill: ~6 hours across C1-C4 (complete)
- Per-skill extensions: estimated 3-4 hours (pending)
- README + CHANGELOG + assembly: ~1.5 hours (pending)
- **Total to v0.2 ship: ~10.5-11.5 hours**

Under the 12-18 hour estimate. The scope reduction (5 extensions instead of 8, Sections 3-4 of meta-skill cut) accounts for the savings without quality compromise.

**Target ship date: still May 24** (9 days from today). Achievable with 4.5-6 hours of work distributed across the remaining window.

---

## What you do next

1. Read this planning document. Verify the 6 decisions and the per-skill distinctness criteria.
2. Flag anything you want to override.
3. When you have a focused 90-120 min window, start Session 1 (technical-partner + engineering-standards + design-partner).
4. Session 2 follows when you have another 90-120 min window.

I do not start writing any extension content until you confirm this plan.
