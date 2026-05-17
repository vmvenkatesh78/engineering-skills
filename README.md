# engineering-skills

Nine skills that encode the engineering discipline I bring to every code, design, and consulting task, written as Claude Agent Skills so the discipline runs whether I am operating or an agent is operating on my behalf.

These are the rules I refuse to ship without. They are opinionated, declarative, and assume you want production-grade output rather than first-pass approximations. They compose with each other and with Anthropic's Agent Skills system, but the rules themselves are platform-agnostic. The discipline is engineering judgment, not vendor lock-in.

---

## Why this exists

The agent-augmented engineering shift is real. The constraint that has shifted is not "can the code be written," because agents write code. The constraint that has tightened is "can the right code be written," which means code that handles all four async states, code with `AbortController` on every async `useEffect`, code that ships against WCAG AA without bolting accessibility on later, code that survives the next refactor by someone who never read the original author's mind.

That discipline is the load-bearing layer. It is what stops AI-augmented engineering from compounding into AI-augmented technical debt. These skills encode that discipline: the standards bar, the review loop, the durability ladder for decision context, the runtime concerns static review misses, the design constraints that separate production UI from AI-generated aesthetics, and the operating bar that judges the combined output.

I use these skills daily in my own work. Publishing them means the discipline is visible, auditable, and applicable in any context where an AI agent operates on engineering work.

---

## The operating bar

[**frontier-engineering-standard**](skills/frontier-engineering-standard/SKILL.md) is the umbrella declaration of the operating bar these skills implement. It is invoked alongside the domain skills for any production-grade engineering task. The bar is articulated through six sections: principles, code that ages well, code review culture, agent-augmented engineering, knowing when not to build, and the limits of what the package claims. The bar is grounded in publicly observable engineering practice (Will Larson on staff engineering, Google's SRE Book and Eng Practices), articulated as personal operating standards rather than authoritative discipline.

The companion artifact is the [dev.to article on type-safe file validation as a security boundary](https://dev.to/vmvenkatesh78/type-your-file-validation-library-as-a-security-boundary-19bo), which walks through one of the skills' principles applied to a concrete worked example.

---

## The eight domain skills

| Skill | What it enforces |
|-------|------------------|
| [engineering-standards](skills/engineering-standards/SKILL.md) | Complete frontend engineering standards: TypeScript strict mode, React patterns, CSS architecture, accessibility, testing, performance, security, code review. v0.2 adds the durability ladder applied to type patterns. |
| [production-ui-standard](skills/production-ui-standard/SKILL.md) | UI quality bar matching Stripe, Linear, Vercel, GitHub: color, typography, spacing, components, anti-patterns. Rejects AI-generated aesthetics. |
| [generator-reviewer-loop](skills/generator-reviewer-loop/SKILL.md) | Two-pass code delivery: Generator writes against the standards bar, Reviewer tears apart at file, function, snippet, line, and variable granularity. 10/10 confidence gate. |
| [triple-pass-review-loop](skills/triple-pass-review-loop/SKILL.md) | Three-pass code delivery: adds Reviewer-of-Reviewer pass that audits the Reviewer's work. Catches missed findings, severity errors, rationalizations. |
| [runtime-behavior-review](skills/runtime-behavior-review/SKILL.md) | Runtime concerns static review misses: self-measurement contamination, shutdown/drain races, probe interference, config flags that are semantic no-ops. |
| [design-partner](skills/design-partner/SKILL.md) | Senior Product Designer + Design Architect: information architecture, interaction patterns, form and table design, configuration UIs, five-state rule. v0.2 adds design at the system level. |
| [technical-partner](skills/technical-partner/SKILL.md) | Staff Engineer + Architect persona: system design, performance under load, concurrency, file upload pipelines, API contracts surviving years. v0.2 adds the staff-IC operating mode. |
| [business-consultant-partner](skills/business-consultant-partner/SKILL.md) | Enterprise BPM consulting: requirements that trace to pain statements, process design with exception handling, scope by exclusion, domain vocabulary. |

---

## How they compose

The skills form a discipline graph. The meta-skill declares the operating bar. The standards skills articulate the rules. The review-loop skills enforce them. The persona skills inform who operates against the standards. The runtime-behavior-review skill adds the runtime layer that static review cannot catch.

```
                          ┌──────────────────────────────────┐
                          │  frontier-engineering-standard   │
                          │       (the operating bar)        │
                          └────────────────┬─────────────────┘
                                           │
                                    DECLARES BAR FOR
                                           │
                          ┌────────────────┴─────────────────┐
                          │                                  │
                          ▼                                  ▼
                ┌────────────────────────┐   ┌────────────────────────┐
                │  engineering-standards │   │ production-ui-standard │
                │  (the code rules)      │   │ (the UI rules)         │
                └───────────┬────────────┘   └───────────┬────────────┘
                            │                            │
                            └──────────┬─────────────────┘
                                       │
                                  ENFORCED BY
                                       │
       ┌───────────────────┬───────────┴────────────┬─────────────────────┐
       │                   │                        │                     │
       ▼                   ▼                        ▼                     ▼
┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────┐
│  generator-  │  │   triple-pass-   │  │  runtime-        │  │  (the persona      │
│  reviewer-   │  │   review-loop    │  │  behavior-       │  │  skills below      │
│  loop        │  │                  │  │  review          │  │  operate against   │
└──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘  │  these rules)      │
       │                   │                     │            └────────────────────┘
       └─────── OPERATED BY (see below) ─────────┘
                           │
       ┌───────────────────┼─────────────────────────────────────┐
       │                   │                                     │
       ▼                   ▼                                     ▼
┌────────────────┐  ┌────────────────┐               ┌────────────────────────────┐
│  technical-    │  │  design-       │               │  business-consultant-      │
│  partner       │  │  partner       │               │  partner                   │
└────────────────┘  └────────────────┘               └────────────────────────────┘
```

### Composition patterns

**Pattern 1. Code task at production bar**

```
frontier-engineering-standard  (the operating bar)
  + technical-partner          (who operates)
  + engineering-standards      (what the rules are)
  + production-ui-standard     (UI quality bar if UI is involved)
  + generator-reviewer-loop    (two-pass delivery)
  + triple-pass-review-loop    (third audit pass for high-stakes work)
```

Use for any production code work: features, bug fixes, refactors, architectural changes. The Generator writes against the standards bar, the Reviewer tears it apart at five granularities, the Reviewer-of-Reviewer audits the Reviewer. The meta-skill declares the multi-year time horizon the code is judged against.

**Pattern 2. Design task at production bar**

```
frontier-engineering-standard  (the operating bar)
  + design-partner             (who operates)
  + production-ui-standard     (visual quality bar)
  + generator-reviewer-loop    (two-pass delivery)
```

Use for any UI design, wireframe, prototype, or configuration interface. The Generator produces designs covering all five states (loading, empty, populated, error, partial). The Reviewer tears apart at six granularities (label, component, layout, flow, state, system). The meta-skill applies the design at system level discipline added in v0.2.

**Pattern 3. Infrastructure or observability or middleware task**

```
frontier-engineering-standard  (the operating bar)
  + technical-partner          (who operates)
  + engineering-standards      (base rules)
  + generator-reviewer-loop    (two-pass delivery)
  + triple-pass-review-loop    (mandatory for infra changes)
  + runtime-behavior-review    (runtime concerns layer)
```

Use for any code where correctness depends on runtime conditions rather than static logic: middleware ordering, rate limiting, health checks, readiness probes, metrics collection, graceful shutdown, circuit breakers, retry logic.

---

## What's new in v0.2

- Added the `frontier-engineering-standard` meta-skill as the umbrella operating bar (six sections plus compositional protocol)
- Added per-skill extensions to `technical-partner` (the staff-IC operating mode), `engineering-standards` (the durability ladder for type-level constraints), and `design-partner` (design at the system level)
- Em-dash discipline applied across the three extended skills (the remaining five skills retain v0.1 style and will be brought to parity in a follow-up cleanup pass)
- Operating-evidence test established as the discipline that decides what content earns its place: claim what is operated against, observe what is not

---

## Installation

Place each skill's directory into your Claude project's `.claude/skills/` directory. The structure is one directory per skill, with the `SKILL.md` file directly inside:

```
.claude/skills/
├── frontier-engineering-standard/SKILL.md
├── engineering-standards/SKILL.md
├── production-ui-standard/SKILL.md
├── generator-reviewer-loop/SKILL.md
├── triple-pass-review-loop/SKILL.md
├── runtime-behavior-review/SKILL.md
├── design-partner/SKILL.md
├── technical-partner/SKILL.md
└── business-consultant-partner/SKILL.md
```

Each skill is a single `SKILL.md` file with YAML frontmatter declaring its trigger and enforcement level. Claude's Agent Skills system handles activation automatically based on the task. Skills declared as `trigger: Always active for ...` are loaded for matching task categories.

Activate individual skills with slash commands (`/engineering-standards`, `/generator-reviewer-loop`, etc.) or activate the full set for any production code task.

---

## What this is not

- A tutorial. These are the rules, not the path to building intuition for them.
- A framework. There is no runtime, no build step, no dependency. Skills are markdown.
- A drop-in replacement for engineering judgment. The skills make judgment legible, but the judgment is what holds the rules together.
- Vendor-locked. Written for the Claude Agent Skills format, but the rules themselves are platform-agnostic and apply equally well as personal team standards, in-house style guides, or AI agent instructions on any platform.

---

## A note on the durability ladder

Several of these skills implement what could be called a **decision-context discipline**: the recognition that the *how* of code can be regenerated but the *why* cannot, unless the codebase preserves it. The rules around naming, types, invariants, and tests are not stylistic preferences. They are mechanisms for encoding context at the highest tier the language and toolchain support, so that a future reader (human or agent) can reconstruct the reasoning without asking the original author.

In a world where agents are increasingly the author *and* the reader, this matters more, not less. The v0.2 extension to `engineering-standards` makes the durability ladder explicit as a five-tier discipline applied to specific TypeScript patterns.

---

## License

MIT. See [LICENSE](LICENSE).

---

## Author

[Venkatesh Mukundan](https://github.com/vmvenkatesh78), Senior Frontend Engineer with focus on design systems and frontend infrastructure. Find me on [LinkedIn](https://www.linkedin.com/in/venkatesh-mukundan) and at [vmvenkatesh78](https://github.com/vmvenkatesh78) on GitHub.

These skills emerged from five years of frontend infrastructure work: design system maintainership, organization-wide engineering standards, end-to-end module ownership, and code review at scale. They are the discipline I have refused to ship without, written down so that the discipline outlasts any one project, role, or company.
