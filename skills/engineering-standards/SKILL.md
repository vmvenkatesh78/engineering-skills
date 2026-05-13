---
name: engineering-standards
description: Complete frontend engineering standards for code quality, React patterns, TypeScript, CSS architecture, accessibility, testing, performance, security, and code review. Apply to every code task — writing, reviewing, debugging, refactoring. These rules are non-negotiable and override convenience or speed.
trigger: Always active for any code generation, review, debugging, or refactoring task.
enforcement: mandatory
---

# Frontend Engineering Standards

These standards apply to every line of code written, reviewed, or refactored. No exceptions for prototypes, demos, or "quick fixes." Code that violates these rules does not ship.

---

## FOUR-STATE RULE — EVERY ASYNC OPERATION

AI generates code that works when everything goes right. Production code handles when things go wrong.

Every fetch, API call, or async operation MUST handle four states:

1. **Loading** — skeleton or spinner visible before data arrives
2. **Error** — meaningful error message, retry mechanism, or fallback
3. **Empty** — explicit empty state with action to resolve it
4. **Success** — the data renders correctly

If code only handles success, it is not production-ready. Reject it on review.

```typescript
// WRONG — happy path only
const { data } = await fetchUsers();
return <UserList users={data} />;

// RIGHT — all four states
function UserList() {
  const { data, error, isLoading } = useUsers();

  if (isLoading) return <Skeleton rows={5} />;
  if (error) return <ErrorState message={error.message} onRetry={refetch} />;
  if (!data?.length) return <EmptyState action={<CreateUserButton />} />;
  return <Table rows={data} />;
}
```

---

## TYPESCRIPT

### Strict Mode — Non-Negotiable

Every project: `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`.

### Forbidden Patterns

- `any` — never. Use `unknown` and narrow with type guards.
- `as` type assertions — only with proof (instanceof check, discriminated union).
- `@ts-ignore` / `@ts-expect-error` without a comment explaining why.
- Non-null assertion (`!`) — use optional chaining or explicit guards.
- `enum` — use `as const` objects instead. Enums have runtime cost and quirky behavior.

```typescript
// WRONG
const value = data as UserResponse;
const name = user!.name;
enum Status { Active, Inactive }

// RIGHT
function isUserResponse(data: unknown): data is UserResponse {
  return typeof data === 'object' && data !== null && 'id' in data;
}
const name = user?.name ?? 'Unknown';
const Status = { Active: 'active', Inactive: 'inactive' } as const;
```

### Required Patterns

- All function parameters: `Readonly<>` when the function should not mutate them.
- All component props: defined as interfaces, not inline types.
- All re-exports: `export type { X }` for type-only exports (isolatedModules compliance).
- Generic components: explicit constraint + default — `<T extends ElementType = 'button'>`.
- Discriminated unions for state machines, not boolean flags.

```typescript
// WRONG — boolean flags
interface State {
  isLoading: boolean;
  isError: boolean;
  data: User[] | null;
  error: Error | null;
}

// RIGHT — discriminated union
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'error'; error: Error }
  | { status: 'success'; data: User[] };
```

---

## REACT PATTERNS

### Component Architecture

- One component per file. Component name matches filename.
- Compound components over prop-heavy monoliths (>8 props = split).
- Hooks extract reusable logic. Components own behavior, styles own presentation.
- `forwardRef` components MUST have `displayName`.
- No inline styles except truly dynamic values (computed positions, API-driven colors).
- No class components. Functional only.

### Hooks Rules

- `useEffect` with async work: ALWAYS include cleanup (AbortController or stale flag).
- `useEffect` dependency arrays: exhaustive or line-level eslint-disable with rationale.
- `useMemo`/`useCallback`: only when profiling shows measurable gain or reference stability matters.
- `useState`: prefer single state object for related state that updates together.
- `useRef`: for DOM refs, abort controllers, values that don't trigger re-renders.
- Never call `setState` during render — derive with `useMemo`.
- Never nest hooks inside conditions or loops.

```typescript
// WRONG — no cleanup, race condition
useEffect(() => {
  fetchData().then(setData);
}, [id]);

// RIGHT — abort controller, cleanup, error handling
useEffect(() => {
  const controller = new AbortController();

  async function load() {
    try {
      const result = await fetchData(id, { signal: controller.signal });
      setData(result);
    } catch (err) {
      if (!controller.signal.aborted) {
        setError(err instanceof Error ? err : new Error('Unknown error'));
      }
    }
  }

  load();
  return () => controller.abort();
}, [id]);
```

### State Management

- Co-locate state where it's used. Don't hoist to global store unless 2+ unrelated components need it.
- No derived state in store — compute with selectors or `useMemo`.
- Context: use for dependency injection (theme, auth), not for frequently changing state.
- Async operations: dispatch started/success/failure actions. Never leave error paths unhandled.

### Props

- Boolean props: `is`/`has`/`allow`/`enable` prefix — `isLoading`, `hasError`.
- Event handlers: `handle` prefix for component handlers, `on` prefix for prop callbacks — `handleClick`, `onClick`.
- No boolean parameters in functions — use options objects instead.

```typescript
// WRONG
function createUser(name: string, isAdmin: boolean, sendEmail: boolean) {}

// RIGHT
function createUser(name: string, options: { isAdmin?: boolean; sendEmail?: boolean }) {}
```

---

## CSS ARCHITECTURE

### Design System CSS

- Token-driven: every color, spacing, font size, radius, and shadow comes from CSS custom properties.
- No hardcoded hex values, pixel values, or font names in component CSS.
- Option B pattern: base selector declares structure with `--_` intermediate variables, variant selectors swap values.
- Data attribute selectors: `[data-variant="primary"]`, not class-based variants.
- Plain CSS files with data attributes — no CSS-in-JS, no CSS Modules, no Tailwind in component library code.

### CSS Rules

- No `!important` unless overriding third-party styles (with comment).
- No element-type selectors deeper than 2 levels.
- `&:focus-visible` not `&:focus` for keyboard-only indicators.
- `flex-shrink: 0` on icons and fixed-width elements in flex containers.
- No `:global()` selectors targeting library internals.
- Transitions: only on `color`, `background-color`, `border-color`, `opacity`, `transform`, `box-shadow`. Never on `padding`, `margin`, `width`, `height`, `border-radius`.
- Duration: 100-150ms for hover/active, 200ms for modals. No bounce/spring/elastic easing.

### Production UI Standard Compliance

All UI output must comply with the production-ui-standard skill. Key checks:
- Primary accent: blue (#2563EB light, #3B82F6 dark). Not indigo, not purple.
- Grays: Slate family (cool-toned). Not neutral gray.
- Shadows: use Slate-900 rgba, not pure black. Cards on white = border, no shadow.
- Border radius: 6px buttons/inputs, 8px cards, 12px modals. Never above 12px on rectangles.
- No gradients on buttons. No glassmorphism. No colored card backgrounds.

---

## ACCESSIBILITY — NON-NEGOTIABLE

- Every interactive element: keyboard-reachable and operable.
- Every icon-only button: `aria-label`.
- Every image: `alt` text (empty `alt=""` for decorative).
- `tabIndex` > 0: NEVER.
- Color as sole indicator: NEVER. Always pair with text, icon, or pattern.
- `role="button"` on non-button elements: MUST also have `tabIndex={0}` and `onKeyDown`.
- Focus indicators: 2px solid outline, 2px offset, visible on EVERY interactive element.
- Contrast: 4.5:1 normal text, 3:1 large text minimum. Target 7:1 where possible.
- Touch targets: 44x44px minimum on mobile.
- Reduced motion: `@media (prefers-reduced-motion: reduce)` disables all animations.
- Screen reader: all form inputs have associated labels, all regions have landmarks.
- Focus traps: only in modals/dialogs. Must be escapable with Escape key.

---

## ERROR HANDLING

- No empty `catch` blocks — at minimum log with context or re-throw.
- API errors: distinguish 4xx (user error) from 5xx (server error) in UI messaging.
- Form validation: show inline errors, not just toasts.
- Network errors: provide retry mechanism or clear feedback.
- `Promise.all`: handle errors — one rejection must not crash the entire batch. Use `Promise.allSettled` when partial success is acceptable.
- Every `switch` statement: `default` case required.
- Null/undefined from API: never assume shape matches TypeScript interface at runtime. Validate.

```typescript
// WRONG — assumes API shape
const userName = response.data.user.name;

// RIGHT — defensive
const userName = response.data?.user?.name ?? 'Unknown';
```

---

## PERFORMANCE

- No O(n²) operations on arrays that could exceed 100 items.
- No `new RegExp()` inside render — extract to module scope or `useMemo`.
- `key` props: stable identifiers, never array indices (unless static, never reordered).
- Lazy load routes and heavy components: `React.lazy` + `Suspense`.
- No layout thrashing: batch DOM reads before DOM writes.
- Images: use `loading="lazy"` for below-fold images, specify `width`/`height` to prevent CLS.
- Bundle: code-split at route level minimum. Monitor with build:size script.
- Memoization: profile first. Don't `useMemo` everything — measure before optimizing.

---

## SECURITY

- No API keys, tokens, or secrets in source code.
- No PII in console.log statements.
- All user inputs sanitized before rendering.
- `dangerouslySetInnerHTML`: ONLY with DOMPurify sanitization. Prefer avoiding entirely.
- `eval()`, `new Function()`, `innerHTML` with user content: FORBIDDEN.
- File uploads: validate extensions AND file size. Use allowlist, not blocklist.
- CSS custom properties from user input: block `url()`, `@import`, `expression()`.
- Third-party scripts: load with `integrity` attribute (SRI) when available.
- CORS: never use `Access-Control-Allow-Origin: *` in production.

---

## TESTING

### Structure

- Test file: adjacent to source — `Component.test.tsx`.
- Test names: describe behavior, not implementation — `"shows error when API returns 400"` not `"calls setError"`.
- No test depends on another test's state.
- Mock at boundaries (API calls, browser APIs), not internal functions.

### Coverage

- Every component: at least one test for render, one for interaction, one for error state.
- Every hook: test the contract (inputs → outputs), not the implementation.
- Every async operation: test both success and error paths.
- Every form: test validation, submission, and error display.
- Edge cases: empty data, null values, extremely long strings, special characters.

### Patterns

```typescript
// Test the four states
describe('UserList', () => {
  it('shows skeleton while loading', () => { /* ... */ });
  it('shows error message when fetch fails', () => { /* ... */ });
  it('shows empty state when no users exist', () => { /* ... */ });
  it('renders user rows on success', () => { /* ... */ });
});
```

- Accessibility tests: use `jest-axe` or `vitest-axe` for automated a11y checks.
- No `data-testid` as the primary query strategy — prefer `getByRole`, `getByLabelText`, `getByText`.
- Test user behavior, not implementation details. Click buttons, type in inputs, verify visible output.

---

## NAMING CONVENTIONS

### Files

- Components: `PascalCase` — `FormUpload.tsx`
- Hooks: `camelCase` with `use` prefix — `useClickOutside.ts`
- Utils: `camelCase` — `formatDate.utils.ts`
- Tests: `*.test.tsx` / `*.test.ts`
- Styles: `*.css` (design system), `*.module.scss` (app-level)
- Constants: `SCREAMING_SNAKE_CASE` in files, `camelCase` filenames

### Code

- Components: `PascalCase`
- Hooks: `useCamelCase`
- Functions/variables: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Types/Interfaces: `PascalCase` — `interface ButtonProps`
- Generic type params: single uppercase letter or descriptive — `T`, `TElement`
- Boolean: `is`/`has`/`should`/`can` prefix
- Event handlers: `handle` prefix (component), `on` prefix (prop)

---

## GIT CONVENTIONS

- Commit format: `type(scope): description`
- Types: `feat`, `fix`, `refactor`, `style`, `perf`, `docs`, `test`, `chore`, `build`, `ci`, `revert`
- One logical change per commit.
- Remove all `console.log` before committing.
- No commented-out code — git has history.
- PR: one concern per PR. Don't mix features with refactors.

---

## FILE SIZE LIMITS

- Components: <300 lines. Split if larger.
- Functions: <40 lines. Extract helpers if larger.
- Max nesting: 3 levels. Invert conditions or extract.
- Utils files: split when exceeding 500 lines.
- CSS files: one file per component. No monolithic stylesheets.

---

## FORBIDDEN PATTERNS — REJECT ON SIGHT

| Pattern | Why | Fix |
|---------|-----|-----|
| `any` | Defeats type safety entirely | `unknown` + type guard |
| `as` without proof | Lies to the compiler | Runtime check first |
| Empty catch block | Swallows errors silently | Log with context or re-throw |
| `console.log` in committed code | Debug artifact in production | Remove before commit |
| `!important` without comment | Specificity war | Fix selector specificity |
| `eslint-disable` at file level | Blanket suppression | Line-level with reason |
| `// TODO` without ticket | Never gets done | `// TODO(TICKET-123): desc` |
| Magic numbers | Unreadable, unmaintainable | Named constants |
| Hardcoded API URLs | Environment coupling | Config constants |
| `index` as React key | Causes bugs on reorder | Stable unique identifier |
| Derived state in store | Stale data, extra updates | Selector or `useMemo` |
| `setState` during render | Infinite loop risk | `useMemo` for derivations |
| `useEffect` without cleanup | Memory leak, race condition | AbortController or stale flag |
| `tabIndex` > 0 | Breaks natural tab order | Restructure DOM order |
| Color as sole indicator | Accessibility failure | Add text/icon/pattern |
| `dangerouslySetInnerHTML` no sanitize | XSS vulnerability | DOMPurify or avoid |
| Fetch with only success handling | Crashes on error/empty | Four-state pattern |
| Boolean function parameters | Unreadable call sites | Options object |
| Nested ternaries | Unreadable | if/else or early return |
| Prop drilling >3 levels | Maintenance burden | Context or composition |

---

## REVIEW CHECKLIST — RUN ON EVERY REVIEW

### Pre-flight (before reading code)

1. Does the PR have a clear title in commit format?
2. Is it one logical change or a mixed bag?
3. Are there any `console.log` statements?

### Code Quality

4. TypeScript strict — no `any`, no unsafe `as`, no `!`
5. Every async operation handles four states (loading, error, empty, success)
6. Every `useEffect` with async work has cleanup
7. Every error path has user-facing feedback
8. No magic numbers, no hardcoded strings, no hardcoded URLs
9. Functions under 40 lines, components under 300 lines
10. Nesting under 3 levels

### Accessibility

11. Every interactive element keyboard-reachable
12. Every icon-only button has `aria-label`
13. Focus indicators visible on all interactive elements
14. Color contrast meets 4.5:1 minimum
15. No color-only state indicators

### Security

16. No secrets, tokens, or PII in source
17. User input sanitized before rendering
18. No `eval`, `innerHTML`, or unsanitized `dangerouslySetInnerHTML`

### CSS

19. No hardcoded colors, spacing, or font values
20. Token-driven: all values from CSS custom properties
21. No `!important` without justification
22. Focus-visible, not focus
23. Compliant with production UI standard

### Testing

24. Tests exist for success, error, and edge cases
25. Test names describe behavior
26. No tests depend on other tests' state
27. Queries use `getByRole`/`getByLabelText`, not `data-testid`

### Performance

28. No O(n²) on growing arrays
29. No RegExp construction in render
30. Stable keys on list items
31. Lazy loading for routes and heavy components

---

## CONFIDENCE THRESHOLD

When reviewing code, score each finding 0-100. Only surface findings at 85+ confidence. This prevents noise and focuses review on real issues.

Severity levels:
- **Critical** — security vulnerability, data loss, crash. Must fix before merge.
- **High** — logic bug, accessibility failure, missing error handling. Should fix.
- **Medium** — performance issue, architectural concern, maintainability. Consider fixing.
- **Low** — naming, style preference, minor improvement. Optional.

Verdict:
- **Ready to Merge** — no Critical or High issues
- **Needs Attention** — Medium issues found
- **Needs Work** — Critical or High issues found
