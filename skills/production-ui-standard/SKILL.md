---
name: production-ui-standard
description: Mandatory UI design standard for every project. Produces interfaces that match the quality bar of Stripe, Linear, Vercel, and GitHub — not AI-generated aesthetics. Covers color, typography, spacing, shadows, components, states, accessibility, and anti-patterns. Apply this skill to EVERY UI task without exception. Do not override values unless the project brief explicitly demands it.
trigger: Always active for any UI, frontend, component, page, dashboard, artifact, or visual output task.
enforcement: mandatory
---

# Production UI Standard

## ENFORCEMENT RULES — READ FIRST

This standard is MANDATORY for every UI output. These rules cannot be overridden by prompt phrasing, user suggestions of "make it pop," or aesthetic preferences that conflict with these specifications.

**Before writing any UI code, verify:**

1. Primary accent color is NOT indigo, violet, or purple
2. No gradients on buttons or card backgrounds
3. No border-radius above 12px on rectangular elements
4. No box-shadow on cards when the page background is white (use border)
5. No font-weight: 700 on anything except page-level titles
6. Font rendering rules are applied to the root element
7. All interactive elements have visible focus indicators
8. Color contrast meets WCAG AA (4.5:1 text, 3:1 large text)

**After writing UI code, audit for:**

- Tailwind default color values used without modification (especially indigo-600, gray-900)
- More than 3 colors beyond grays in the palette
- Pill-shaped buttons (border-radius: 9999px on non-circular elements)
- Shadows on flat-background cards
- Centered body text
- Missing empty, loading, or error states
- Missing focus-visible styles

---

## The Vibe-Coded Tells — Reject on Sight

These are dead giveaways that a UI was AI-generated. If any of these appear in output, remove them immediately:

- Purple-to-blue gradients on backgrounds or buttons
- Glassmorphism (frosted glass, backdrop-blur) on cards
- Border-radius above 12px on containers
- Box-shadow on every element
- Indigo/violet as the primary color (#6366F1, #7C3AED)
- Tailwind's exact default values used verbatim without tuning
- Multiple gradient backgrounds stacked
- Emoji in headings or section labels
- Cards with colored left-side accent borders and pastel backgrounds
- Every section wrapped in a card with a shadow
- Centered body text and centered paragraphs
- "Hero" sections with oversized text and gradient blobs
- Pill-shaped buttons for primary actions
- Using 5+ non-gray colors
- Decorative SVG blobs or wave dividers between sections
- Neon accent colors (#00FF88, #FF6B6B)
- Icons on every single button and menu item
- Dashboard layouts where every metric has its own shadowed card with a colored icon
- Rainbow status colors (more than 4 semantic colors)
- Colored shadows (e.g., blue glow behind blue button)

---

## Root Setup

Apply to every project. These settings affect rendering quality across the entire app.

```css
:root {
  color-scheme: light dark;
}

html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
  font-feature-settings: "cv02", "cv03", "cv04", "cv11";
}

::selection {
  background: #DBEAFE;
  color: #1E3A5F;
}

[data-theme="dark"] ::selection {
  background: rgba(59, 130, 246, 0.3);
  color: #F1F5F9;
}

/* Scrollbar — Webkit browsers */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: #CBD5E1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #94A3B8;
}

[data-theme="dark"] ::-webkit-scrollbar-thumb {
  background: #475569;
}

/* Scrollbar — Firefox */
html {
  scrollbar-width: thin;
  scrollbar-color: #CBD5E1 transparent;
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Color System

Grays use the Slate family (cool-toned with a subtle blue undertone). This is what makes text crisper and surfaces feel more refined than perfectly neutral gray. The accent blue is shifted off common Tailwind defaults.

### Light Theme (Default)

```css
:root {
  /* Backgrounds — Slate undertone, not pure gray */
  --bg-primary: #FFFFFF;
  --bg-secondary: #F8FAFC;          /* Slate-50 */
  --bg-tertiary: #F1F5F9;           /* Slate-100 */
  --bg-elevated: #FFFFFF;
  --bg-inset: #E2E8F0;              /* Slate-200 — progress bars, dividers */

  /* Text */
  --text-primary: #0F172A;          /* Slate-900 */
  --text-secondary: #475569;        /* Slate-600 */
  --text-tertiary: #94A3B8;         /* Slate-400 */
  --text-on-accent: #FFFFFF;

  /* Interactive — tuned blue, not Tailwind's exact blue-600 */
  --accent-primary: #2563EB;
  --accent-primary-hover: #1D4FD7;
  --accent-primary-active: #1E3FA0;
  --accent-subtle: #EFF6FF;
  --accent-subtle-hover: #DBEAFE;

  /* Borders */
  --border-primary: #E2E8F0;        /* Slate-200 */
  --border-secondary: #CBD5E1;      /* Slate-300 */
  --border-focus: #2563EB;

  /* Status — muted, not saturated */
  --status-success: #059669;         /* Emerald-600 */
  --status-success-bg: #ECFDF5;
  --status-warning: #D97706;         /* Amber-600 */
  --status-warning-bg: #FFFBEB;
  --status-error: #DC2626;           /* Red-600 */
  --status-error-bg: #FEF2F2;
  --status-info: #2563EB;
  --status-info-bg: #EFF6FF;
}
```

### Dark Theme

```css
[data-theme="dark"] {
  --bg-primary: #0F172A;            /* Slate-900 */
  --bg-secondary: #1E293B;          /* Slate-800 */
  --bg-tertiary: #334155;           /* Slate-700 */
  --bg-elevated: #1E293B;
  --bg-inset: #0F172A;

  --text-primary: #F1F5F9;          /* Slate-100 */
  --text-secondary: #CBD5E1;        /* Slate-300 */
  --text-tertiary: #64748B;         /* Slate-500 */
  --text-on-accent: #FFFFFF;

  --accent-primary: #3B82F6;
  --accent-primary-hover: #2563EB;
  --accent-primary-active: #1D4FD7;
  --accent-subtle: rgba(59, 130, 246, 0.1);
  --accent-subtle-hover: rgba(59, 130, 246, 0.15);

  --border-primary: #334155;        /* Slate-700 */
  --border-secondary: #475569;      /* Slate-600 */
  --border-focus: #3B82F6;

  --status-success: #10B981;
  --status-success-bg: rgba(16, 185, 129, 0.1);
  --status-warning: #F59E0B;
  --status-warning-bg: rgba(245, 158, 11, 0.1);
  --status-error: #EF4444;
  --status-error-bg: rgba(239, 68, 68, 0.1);
  --status-info: #3B82F6;
  --status-info-bg: rgba(59, 130, 246, 0.1);
}
```

### Color Rules

- ONE accent color. Blue for light (#2563EB), lighter blue for dark (#3B82F6). That is it.
- Status colors are semantic only — success, error, warning, info. Never decorative.
- 80% of the UI is slate-toned text on white/slate backgrounds. Accent appears only on interactive elements and selected states.
- No gradients on buttons — solid fill only.
- No colored backgrounds on cards — white card, slate border.
- Dark mode background is Slate-900 (#0F172A), never pure black (#000000).

---

## Typography

### Font Stack

```css
:root {
  --font-sans: "Inter var", "Inter", -apple-system, BlinkMacSystemFont,
    "Segoe UI", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", "Fira Code", "SF Mono", "Cascadia Code",
    "Consolas", monospace;
}
```

Use "Inter var" (variable font) when available — smaller file, smoother weight transitions. Fall back to "Inter" for static font installations.

If the project needs brand personality (not default), swap Inter for ONE of: Geist Sans, DM Sans, or Satoshi. Never decorative display fonts for UI text.

### Inter OpenType Features

```css
/* Tabular numbers — critical for tables, metrics, prices */
.tabular-nums {
  font-variant-numeric: tabular-nums;
}

/* Alternative characters — more distinctive l, a, g */
html {
  font-feature-settings: "cv02", "cv03", "cv04", "cv11";
}
```

- `cv02`: Alternate a (single-storey → double-storey)
- `cv03`: Alternate g (single-storey → double-storey)
- `cv04`: Alternate i (serif → sans-serif dot)
- `cv11`: Single-storey l disambiguation
- Apply `tabular-nums` on every number column, price, metric, and counter.

### Type Scale

8 sizes. No more.

```css
:root {
  --text-xs: 0.75rem;       /* 12px — badges, timestamps, meta */
  --text-sm: 0.875rem;      /* 14px — secondary text, table cells, nav */
  --text-base: 1rem;        /* 16px — body text, form inputs */
  --text-lg: 1.125rem;      /* 18px — section subheadings */
  --text-xl: 1.25rem;       /* 20px — card titles, panel headers */
  --text-2xl: 1.5rem;       /* 24px — page titles */
  --text-3xl: 1.875rem;     /* 30px — major headings (rare in app UI) */
  --text-4xl: 2.25rem;      /* 36px — hero/landing pages only */
}
```

### Line Heights

```css
:root {
  --leading-tight: 1.2;     /* Headings text-xl and above */
  --leading-snug: 1.35;     /* Subheadings, card titles */
  --leading-normal: 1.5;    /* Body text, descriptions */
  --leading-relaxed: 1.625; /* Long-form content, articles */
}
```

### Font Weights

```css
:root {
  --font-normal: 400;       /* Body, descriptions, table cells */
  --font-medium: 500;       /* Labels, nav items, button text, subtle emphasis */
  --font-semibold: 600;     /* Section headings, important numbers */
  --font-bold: 700;         /* Page-level titles ONLY — use sparingly */
}
```

### Typography Rules

- App UI base font: 14px on body. Content/reading pages: 16px.
- Left-align everything. Never center body text. Center only hero headings on landing pages.
- Default heading weight: 600 (semibold). Use 700 only for the single page-level title (h1).
- One heading size per section. Use weight for sub-hierarchy, not stacked sizes.
- Secondary text uses --text-tertiary (slate-400), never a lighter tint of the accent blue.
- Letter-spacing: -0.011em on text-xl and above (tightens large text). 0 on body text. +0.025em on uppercase labels only.

---

## Spacing

### Scale — 4px base unit

```css
:root {
  --space-0: 0;
  --space-0.5: 0.125rem;    /* 2px — hairline gaps, icon optical adjust */
  --space-1: 0.25rem;       /* 4px — icon-to-text gap, inline padding */
  --space-1.5: 0.375rem;    /* 6px — label-to-input gap */
  --space-2: 0.5rem;        /* 8px — tight spacing, button group gap */
  --space-3: 0.75rem;       /* 12px — list item padding, input padding-x */
  --space-4: 1rem;          /* 16px — card grid gap, section inner gap */
  --space-5: 1.25rem;       /* 20px — form field vertical gap */
  --space-6: 1.5rem;        /* 24px — card content padding */
  --space-8: 2rem;          /* 32px — section spacing */
  --space-10: 2.5rem;       /* 40px — major section gap */
  --space-12: 3rem;         /* 48px — page section padding */
  --space-16: 4rem;         /* 64px — landing page section padding only */
}
```

### Spacing Application

| Element | Value | Token |
|---------|-------|-------|
| Card inner padding | 24px | --space-6 |
| Card grid gap | 16px | --space-4 |
| Form field vertical gap | 20px | --space-5 |
| Label to input gap | 6px | --space-1.5 |
| Input to helper/error text | 4px | --space-1 |
| Button group gap | 8px | --space-2 |
| Section vertical spacing | 32px | --space-8 |
| Major section gap | 48px | --space-12 |
| Page horizontal padding (mobile) | 16px | --space-4 |
| Page horizontal padding (desktop) | 32-48px | --space-8 to --space-12 |
| Sidebar item vertical padding | 6px | --space-1.5 |
| Table cell padding | 10px 12px | --space-2.5 / --space-3 |

### Spacing Rules

- Never exceed 64px spacing in app UI. Large spacing is for landing pages only.
- Use optical alignment, not mathematical. If a heading visually appears uncentered in a card despite equal padding, add 1-2px to the top to compensate for descenders.
- Gap between a heading and its first content paragraph: 8px. Not 16px (too much), not 4px (too tight).

---

## Border Radius

```css
:root {
  --radius-sm: 4px;         /* Badges, tags, small chips */
  --radius-md: 6px;         /* Buttons, inputs, dropdowns */
  --radius-lg: 8px;         /* Cards, panels */
  --radius-xl: 12px;        /* Modals, dialogs, popovers */
  --radius-full: 9999px;    /* Avatars, circular indicators ONLY */
}
```

### Radius Rules

- Buttons: 6px. Not pill (9999px), not square (0).
- Inputs: 6px. Matches buttons — they sit together in forms.
- Cards: 8px. Slightly larger than elements inside.
- Modals: 12px. Maximum for rectangular elements.
- NEVER use border-radius above 12px on any rectangular element.
- radius-full (9999px) ONLY for avatars and circular status dots.

---

## Shadows

```css
:root {
  --shadow-xs: 0 1px 2px 0 rgba(15, 23, 42, 0.04);
  --shadow-sm: 0 1px 3px 0 rgba(15, 23, 42, 0.06),
               0 1px 2px -1px rgba(15, 23, 42, 0.06);
  --shadow-md: 0 4px 6px -1px rgba(15, 23, 42, 0.07),
               0 2px 4px -2px rgba(15, 23, 42, 0.05);
  --shadow-lg: 0 10px 15px -3px rgba(15, 23, 42, 0.08),
               0 4px 6px -4px rgba(15, 23, 42, 0.05);
  --shadow-xl: 0 20px 25px -5px rgba(15, 23, 42, 0.1),
               0 8px 10px -6px rgba(15, 23, 42, 0.06);
}
```

Shadow color uses Slate-900 (rgba(15, 23, 42)) instead of pure black. This creates warmer, more natural shadows that match the cool-toned gray system.

### Shadow Rules

- Cards on white background: NO shadow. Use 1px border (--border-primary).
- Cards on gray background: --shadow-xs only.
- Dropdowns and popovers: --shadow-md to --shadow-lg.
- Modals and dialogs: --shadow-xl.
- Buttons: NEVER have shadows.
- Colored shadows (blue glow, etc.): NEVER.

---

## Z-Index System

```css
:root {
  --z-base: 0;
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-overlay: 300;
  --z-modal: 400;
  --z-popover: 500;
  --z-tooltip: 600;
  --z-toast: 700;
}
```

### Z-Index Rules

- NEVER use arbitrary z-index values (z-index: 9999).
- Every floating element uses one of these 8 layers.
- Stacking within a layer: use z-index: calc(var(--z-dropdown) + 1) for sub-layers.
- Toast notifications sit above everything except tooltips.

---

## Buttons

### Variants

```css
/* Primary */
.btn-primary {
  background: var(--accent-primary);
  color: var(--text-on-accent);
  border: 1px solid transparent;
  padding: 7px 15px;
  font-size: var(--text-sm);
  font-weight: var(--font-medium);
  border-radius: var(--radius-md);
  line-height: 20px;
  height: 36px;
  cursor: pointer;
  transition: background var(--transition-normal);
}

.btn-primary:hover {
  background: var(--accent-primary-hover);
}

.btn-primary:active {
  background: var(--accent-primary-active);
}

.btn-primary:focus-visible {
  outline: 2px solid var(--accent-primary);
  outline-offset: 2px;
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

/* Secondary (outlined) */
.btn-secondary {
  background: var(--bg-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-secondary);
}

.btn-secondary:hover {
  background: var(--bg-secondary);
}

/* Ghost (no border) */
.btn-ghost {
  background: transparent;
  color: var(--text-secondary);
  border: 1px solid transparent;
}

.btn-ghost:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

/* Danger */
.btn-danger {
  background: var(--status-error);
  color: var(--text-on-accent);
  border: 1px solid transparent;
}

.btn-danger:hover {
  background: #B91C1C;
}
```

### Button Sizes

| Size | Height | Padding | Font Size | Usage |
|------|--------|---------|-----------|-------|
| Small | 28px | 3px 11px | 12px | Dense UIs, table row actions |
| Medium | 36px | 7px 15px | 14px | DEFAULT — forms, toolbars |
| Large | 44px | 11px 23px | 16px | Landing page CTAs only |

### Button Rules

- Default to Medium (36px). Small for dense UIs. Large for landing pages only.
- Text is sentence case ("Save changes"). Not Title Case. Not ALL CAPS.
- Maximum 2 buttons in a group (primary + secondary). Third action becomes a text link or menu.
- Icon buttons: square (36x36 medium), icon 16-20px centered.
- Loading state: replace text with spinner, maintain button width to prevent layout shift.
- Padding uses odd values (7px, 15px) because the 1px border means total height = 7+7+20+1+1 = 36px exactly.

---

## Inputs & Forms

```css
.input {
  height: 36px;
  padding: 7px 11px;
  font-size: var(--text-sm);
  font-family: inherit;
  color: var(--text-primary);
  background: var(--bg-primary);
  border: 1px solid var(--border-secondary);
  border-radius: var(--radius-md);
  transition: border-color var(--transition-normal),
              box-shadow var(--transition-normal);
}

.input:hover {
  border-color: #94A3B8;            /* Slate-400 — subtle hover feedback */
}

.input:focus {
  border-color: var(--accent-primary);
  box-shadow: 0 0 0 3px var(--accent-subtle);
  outline: none;
}

.input::placeholder {
  color: var(--text-tertiary);
}

.input:disabled {
  background: var(--bg-secondary);
  color: var(--text-tertiary);
  cursor: not-allowed;
}

.input[aria-invalid="true"] {
  border-color: var(--status-error);
}

.input[aria-invalid="true"]:focus {
  box-shadow: 0 0 0 3px var(--status-error-bg);
}
```

### Form Rules

- Labels above inputs. Label: 14px, weight 500, color --text-primary. Gap to input: 6px.
- Helper text below: 12px, --text-tertiary. Gap from input: 4px.
- Error text: 12px, --status-error. Replaces helper text (never stacks).
- Required: red asterisk after label text.
- Input max-width: 400px for single fields, 600px for wide fields, 100% within card containers.
- Textarea: min-height 80px, resize vertical only.
- Select elements: same height/padding as inputs (36px, 7px 11px).
- Checkbox/radio: 16x16px, 2px border, accent color when checked.

---

## Cards

```css
.card {
  background: var(--bg-elevated);
  border: 1px solid var(--border-primary);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
}
```

### Card Rules

- No shadow on white-background pages. Border only.
- No colored backgrounds.
- No left-side colored accent borders.
- Card header: text-lg (18px) or text-xl (20px), weight 600. Not a separate colored bg.
- Card footer (actions): 1px border-top, padding-top 16px.
- Cards in grid: 16px gap. Single column on mobile.

---

## Tables

```css
.table {
  width: 100%;
  border-collapse: collapse;
  font-size: var(--text-sm);
}

.table th {
  text-align: left;
  font-weight: var(--font-medium);
  color: var(--text-tertiary);
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: 0.025em;
  padding: 8px 12px;
  border-bottom: 1px solid var(--border-primary);
  position: sticky;
  top: 0;
  background: var(--bg-primary);
}

.table td {
  padding: 10px 12px;
  color: var(--text-primary);
  border-bottom: 1px solid var(--border-primary);
  font-variant-numeric: tabular-nums;
}

.table tr:hover td {
  background: var(--bg-secondary);
}
```

### Table Rules

- Headers: uppercase, 12px, slate-400, medium weight, 0.025em letter-spacing.
- No alternating row colors. Hover highlight + bottom border on every row.
- Sticky header on scroll.
- Number columns: right-aligned with tabular-nums. Text: left-aligned.
- Status: small colored badge in one cell, not full-width colored row backgrounds.

---

## Navigation

### Sidebar (240px)

```css
.sidebar {
  width: 240px;
  background: var(--bg-primary);
  border-right: 1px solid var(--border-primary);
  padding: 12px 8px;
}

.sidebar-item {
  padding: 6px 12px;
  border-radius: var(--radius-md);
  font-size: var(--text-sm);
  font-weight: var(--font-normal);
  color: var(--text-secondary);
  cursor: pointer;
  transition: background var(--transition-fast);
}

.sidebar-item:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

.sidebar-item.active {
  background: var(--accent-subtle);
  color: var(--accent-primary);
  font-weight: var(--font-medium);
}

.sidebar-section-label {
  font-size: 11px;
  font-weight: var(--font-medium);
  color: var(--text-tertiary);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  padding: 16px 12px 4px;
}
```

### Navigation Rules

- Sidebar: 240px, collapsible to 64px (icons only).
- Active item: --accent-subtle bg, --accent-primary text. Not a bold blue bar.
- Icons: 16px, stroke-width 1.5-2px. Lucide or Heroicons outline style.
- No nested dropdowns. Flat structure with section labels.
- Top nav: 48px height, 14px font, 500 weight. Active: 2px bottom border.

---

## Modals & Dialogs

```css
.dialog-overlay {
  background: rgba(15, 23, 42, 0.5);
  backdrop-filter: blur(2px);        /* Subtle — not glassmorphism */
}

.dialog-content {
  background: var(--bg-elevated);
  border-radius: var(--radius-xl);
  box-shadow: var(--shadow-xl);
  padding: var(--space-6);
  max-width: 480px;
  width: calc(100vw - 32px);
}
```

### Dialog Sizes

| Size | Width | Usage |
|------|-------|-------|
| Small | 400px | Confirmations, simple alerts |
| Medium | 480px | Standard forms (DEFAULT) |
| Large | 640px | Complex forms, multi-step |

### Dialog Rules

- Title: text-lg (18px), semibold. One line maximum.
- Description: text-sm (14px), --text-secondary. Optional.
- Actions: right-aligned. Primary right, cancel left.
- Close: top-right, ghost style, X icon.
- Overlay click dismisses (unless destructive confirmation).
- No nested dialogs. Redesign the flow.

---

## Toasts & Notifications

```css
.toast {
  background: var(--bg-elevated);
  border: 1px solid var(--border-primary);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  padding: 12px 16px;
  font-size: var(--text-sm);
  max-width: 420px;
  z-index: var(--z-toast);
}
```

### Toast Rules

- Position: top-right or bottom-right. Not centered top.
- Duration: 5s for info/success, persistent for errors (requires dismiss).
- Maximum 3 visible toasts stacked. Older ones auto-dismiss.
- Include a small colored icon (16px) for status, not a colored background.
- Dismiss: X button right side. Hover pauses auto-dismiss timer.

---

## Badges & Status

```css
.badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  font-size: var(--text-xs);
  font-weight: var(--font-medium);
  border-radius: var(--radius-sm);
  line-height: 16px;
}

.badge-success { background: var(--status-success-bg); color: var(--status-success); }
.badge-error   { background: var(--status-error-bg);   color: var(--status-error); }
.badge-warning { background: var(--status-warning-bg);  color: var(--status-warning); }
.badge-neutral { background: var(--bg-tertiary);        color: var(--text-secondary); }
```

### Badge Rules

- 4px radius, not pill-shaped.
- Matching hue: green bg + green text, red bg + red text.
- Small: 2px 8px padding, 12px font.
- Maximum 4 variants: success, error, warning, neutral.
- Status dot indicator: 6px circle, margin-right 6px, matching status color.

---

## State Patterns

### Empty States

```
┌──────────────────────────────────┐
│                                  │
│         [illustration]           │  ← Optional, 120-160px max
│                                  │
│      No items yet                │  ← text-lg, semibold, --text-primary
│                                  │
│   Description of what goes       │  ← text-sm, --text-secondary
│   here and how to get started.   │     max-width 320px, centered
│                                  │
│     [ Primary Action ]           │  ← Primary button
│                                  │
└──────────────────────────────────┘
```

- Always provide a primary action to resolve the empty state.
- Description: max 2 lines, action-oriented ("Create your first project" not "There are no projects").
- No sad-face illustrations or apologetic language.

### Loading States

```css
/* Skeleton placeholder */
.skeleton {
  background: linear-gradient(
    90deg,
    var(--bg-tertiary) 25%,
    var(--bg-secondary) 50%,
    var(--bg-tertiary) 75%
  );
  background-size: 200% 100%;
  animation: skeleton-pulse 1.5s ease-in-out infinite;
  border-radius: var(--radius-sm);
}

@keyframes skeleton-pulse {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

- Skeleton loaders for content areas (tables, cards, lists).
- Spinner for actions (button loading, form submit). 16px spinner, 2px stroke.
- Skeleton shapes match the content they replace (line = text, circle = avatar, rectangle = image).
- Never show a full-page spinner. Show the page shell (nav, sidebar) immediately, skeleton the content.

### Error States

- Inline errors: red text below the failed field, 12px.
- Page-level errors: centered card with error icon, message, and retry action.
- Network errors: top-of-page banner, --status-error-bg background, dismissable.
- Never use alerts/modals for inline form validation errors.

---

## Transitions

```css
:root {
  --transition-fast: 100ms ease;
  --transition-normal: 150ms ease;
  --transition-slow: 200ms ease-out;
  --transition-modal-in: 200ms ease-out;
  --transition-modal-out: 150ms ease-in;
}
```

### Transition Rules

- Hover/active: 100-150ms on background and color only.
- Modal enter: 200ms ease-out. Modal exit: 150ms ease-in.
- No bounce, spring, or elastic easing in app UIs.
- Transition only: color, background-color, border-color, opacity, transform, box-shadow.
- NEVER transition padding, margin, border-radius, width, or height.

---

## Responsive

```css
--bp-sm: 640px;
--bp-md: 768px;
--bp-lg: 1024px;
--bp-xl: 1280px;
--bp-2xl: 1536px;
```

### Responsive Rules

- Sidebar collapses below 1024px.
- Card grids: 1 col mobile, 2 col tablet, 3-4 col desktop.
- Tables: horizontal scroll on mobile. Tables remain tables (no card transformation).
- Touch targets: 44x44px minimum on mobile (overrides 36px default).
- No horizontal scrolling at any breakpoint.
- Content max-width: 1200px dashboards, 720px reading/settings pages.

---

## Content Density

### Default Density

The values in this standard are tuned for default density — comfortable spacing for general use.

### Compact Density (Optional)

For power-user UIs (admin dashboards, data-heavy tables):

- Table row padding: 6px 12px (instead of 10px 12px)
- Sidebar item padding: 4px 12px (instead of 6px 12px)
- Card padding: 16px (instead of 24px)
- Button height: 28px (instead of 36px)
- Font base: 13px (instead of 14px)

Compact mode should be opt-in, never the default. Trigger via a user preference toggle, not a global decision.

---

## Layout

```
┌──────────────────────────────────────────────────────┐
│ Top Bar (48px)                                        │
│ logo          breadcrumb              user menu       │
├────────────┬─────────────────────────────────────────│
│ Sidebar    │ Main Content                             │
│ (240px)    │ ┌─────────────────────────────────────┐  │
│            │ │ Page Header                          │  │
│ section    │ │ Title (text-2xl, 600)   [Action btn] │  │
│ labels     │ ├─────────────────────────────────────┤  │
│            │ │ Content                               │  │
│ nav items  │ │ max-width: 1200px                     │  │
│            │ │ padding: 24-48px                       │  │
│            │ └─────────────────────────────────────┘  │
└────────────┴─────────────────────────────────────────┘
```

### Layout Rules

- Sidebar: fixed or sticky, separate scroll from content.
- Page header: auto height, min 48px. Title + primary action.
- Breadcrumbs: text-sm, slate-400, "/" separator, above title.
- No full-width colored header bars. Same bg as content.
- Content padding: 24px mobile, 32px tablet, 48px desktop.

---

## Accessibility Minimums

- Contrast: 4.5:1 normal text, 3:1 large text. Target AAA (7:1) where possible.
- Focus: 2px solid outline, 2px offset, on EVERY interactive element.
- No color-only meaning. Every status has text AND color.
- Touch targets: 44x44px minimum mobile.
- Keyboard: tab order matches visual order. No focus traps except modals.
- Reduced motion: @media (prefers-reduced-motion: reduce) disables all animations.
- Screen reader: all images have alt text, all icon buttons have aria-label, all form inputs have associated labels.

---

## Anti-Pattern Quick Reference

| Do NOT | Do Instead |
|--------|-----------|
| Purple/indigo accent | Blue (#2563EB) |
| Gradient button bg | Solid fill |
| Shadow on flat-bg card | 1px border |
| Pill button (9999px) | 6px radius |
| Colored card bg | White + border |
| Centered body text | Left-align |
| Icon on every button | Text-only, icon-only separate |
| Multiple font families | Inter + monospace only |
| Bold (700) headings | Semibold (600), bold only for h1 |
| 5+ status colors | success/error/warning/neutral |
| Glassmorphism cards | Solid bg |
| Neon accents | Muted functional blue |
| Wave/blob dividers | Clean spacing or borders |
| #000000 dark mode bg | #0F172A (Slate-900) |
| z-index: 9999 | Use z-index system tokens |
| No empty/loading states | Always handle all states |
| Pure neutral gray | Slate (cool-toned) grays |
| Missing font-smoothing | Always apply antialiased |

---

## Quick Reference — Copy to Start

```css
:root {
  color-scheme: light dark;

  --bg-primary: #FFFFFF;
  --bg-secondary: #F8FAFC;
  --bg-tertiary: #F1F5F9;
  --text-primary: #0F172A;
  --text-secondary: #475569;
  --text-tertiary: #94A3B8;
  --accent-primary: #2563EB;
  --accent-primary-hover: #1D4FD7;
  --border-primary: #E2E8F0;
  --border-secondary: #CBD5E1;
  --status-success: #059669;
  --status-error: #DC2626;
  --status-warning: #D97706;

  --font-sans: "Inter var", "Inter", -apple-system, BlinkMacSystemFont, sans-serif;
  --font-mono: "JetBrains Mono", monospace;

  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;

  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;

  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
  --radius-xl: 12px;

  --shadow-sm: 0 1px 3px 0 rgba(15,23,42,0.06), 0 1px 2px -1px rgba(15,23,42,0.06);
  --shadow-lg: 0 10px 15px -3px rgba(15,23,42,0.08), 0 4px 6px -4px rgba(15,23,42,0.05);

  --z-dropdown: 100;
  --z-sticky: 200;
  --z-overlay: 300;
  --z-modal: 400;
  --z-tooltip: 600;
  --z-toast: 700;

  --transition-fast: 100ms ease;
  --transition-normal: 150ms ease;
}

html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-feature-settings: "cv02", "cv03", "cv04", "cv11";
}
```

---

This standard produces UIs indistinguishable from those built by product teams at Stripe, Linear, Vercel, and GitHub. Every value is deliberate. Follow it without exception.
