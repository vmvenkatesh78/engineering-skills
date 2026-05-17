# Changelog

All notable changes to this package are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) conventions. Versioning is informal but coherent: v0.x reflects iterations of the discipline; major version increments are reserved for breaking changes to the SKILL.md format or composition model.

## [v0.2] - 2026-05-17

### Added

- **`frontier-engineering-standard` meta-skill.** New umbrella skill that declares the operating bar the package was implicitly working against. Six sections (principles, code that ages well, code review culture, agent-augmented engineering, knowing when not to build, the limits of what the package claims) plus a compositional protocol describing how the meta-skill activates alongside the eight domain skills. Grounded in publicly observable engineering practice (Will Larson on staff engineering, Google's SRE Book and Engineering Practices) and articulated as personal operating standards rather than authoritative discipline.
- **`technical-partner` extension: The staff-IC operating mode.** New section covering cross-team influence without authority, the doc-versus-code decision, sponsoring others (explicitly framed as the part of the role I work toward, not the part I currently operate against), and operating across multiple codebases. Approximately 670 words.
- **`engineering-standards` extension: The durability ladder for type-level constraints.** New section codifying a five-tier discipline (type → runtime invariant → test → named constant → comment) for encoding correctness constraints in TypeScript codebases, with cross-reference to the published article on file validation as a security boundary. Approximately 869 words.
- **`design-partner` extension: Design at the system level.** New section covering design tokens as code, governance as drift prevention, documentation as durable artifact, and the design-engineering interface. Approximately 662 words.
- **README rework.** Front door rewritten to reflect the nine-skill package, the operating-bar framing introduced by the meta-skill, and the updated composition diagram.

### Changed

- **Em-dash discipline applied** to the three skills that received v0.2 extensions (`technical-partner`, `engineering-standards`, `design-partner`). The remaining five skills retain v0.1 prose style and will be brought to parity in a follow-up cleanup pass.
- **YAML frontmatter fix** on `technical-partner`. The v0.1 description contained an unquoted colon-space sequence that caused YAML parsers to fail with "mapping values are not allowed in this context." Description is now wrapped in double quotes.

### Operating principles introduced in v0.2

- **Operating-evidence test.** Claim what you operate against; observe what you do not. Applied across all v0.2 content as the discipline that decides what earns its place in a skill. Made explicit in the `frontier-engineering-standard` meta-skill.
- **Honest framing of aspirational content.** Where a principle is something I work toward rather than currently operate against, the framing is explicit about that gap (visible in `technical-partner` Principle 3 and `design-partner` Principle 3).

## [v0.1] - 2026-05-13

### Initial release

Eight skills covering frontend engineering standards, production UI quality, two-pass and three-pass code review loops, runtime behavior analysis, design system discipline, staff-engineer persona, and enterprise BPM consulting persona.

- `engineering-standards` (frontend code quality, React patterns, TypeScript, CSS, accessibility, testing, performance, security)
- `production-ui-standard` (visual quality bar matching Stripe, Linear, Vercel, GitHub)
- `generator-reviewer-loop` (two-pass code delivery with 10/10 confidence gate)
- `triple-pass-review-loop` (three-pass code delivery with reviewer-of-reviewer audit)
- `runtime-behavior-review` (runtime concerns static review misses)
- `design-partner` (senior product designer + design architect persona)
- `technical-partner` (staff engineer + architect persona)
- `business-consultant-partner` (enterprise BPM consulting persona)

Companion artifact: dev.to article on type-safe file validation as a security boundary, demonstrating the engineering-standards discipline applied to a concrete worked example.

[v0.2]: https://github.com/vmvenkatesh78/engineering-skills/releases/tag/v0.2
[v0.1]: https://github.com/vmvenkatesh78/engineering-skills/releases/tag/v0.1
