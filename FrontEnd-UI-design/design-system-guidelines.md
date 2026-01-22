# Web Application Design System — Guidelines & Best Practices

## Purpose

A design system is a **product-quality UI platform**: shared foundations, components, and patterns that keep teams aligned and help the interface scale.

Primary outcomes:

- Consistent UX and visuals
- Faster delivery through reuse
- Lower design and engineering debt
- Better accessibility and quality by default
- Predictable change management (versioning, deprecation)

---

## 1) Core Principles

### 1.1 Consistency over novelty

- Reuse existing patterns by default.
- Introduce new patterns only when a real gap exists.

### 1.2 Design for scale

Assume:

- More pages and states
- More contributors
- More themes/brands
- More devices, languages, and abilities

### 1.3 Accessibility is a feature

- Target **WCAG 2.2 AA** minimum (2.1 AA acceptable if your org standard is older).
- Keyboard support and visible focus are non-negotiable.
- Provide accessible names, roles, and relationships.

### 1.4 Single source of truth

- **Figma describes intent** (layout, hierarchy, tokens).
- **Code defines behavior** (states, keyboard interaction, a11y).
- Product screens should not re-implement UI primitives.

---

## 2) What a Design System Contains

### 2.1 Foundations

The smallest set of consistent rules used everywhere:

- Color (semantic tokens)
- Typography
- Spacing & sizing scale
- Grid/breakpoints
- Elevation/shadows
- Motion
- Iconography
- Border radius, shapes

### 2.2 Components

Reusable building blocks:

- Inputs, buttons, navigation, feedback, data display
- Each component includes **variants, states, and accessibility**

### 2.3 Patterns

Documented ways of combining components:

- Authentication flows
- Tables & filtering
- Empty states
- Error handling
- Multi-step forms

### 2.4 Guidance

- Usage rules (do/don’t)
- Content guidelines (tone, labels, errors)
- Accessibility guidance
- Contribution workflow

---

## 3) Recommended Project Structure (Design + Code)

### 3.1 Code repository layout

A practical layout for a web app (React/Next.js or similar):

```
design-system/
  packages/
    tokens/
      src/
      dist/
    ui/
      src/
        components/
        foundations/
        hooks/
        styles/
      stories/
      tests/
    icons/
      src/
      dist/
  apps/
    docs/            # Storybook / docs site
  tooling/
  README.md
```

### 3.2 Design tool (Figma) layout

- Foundations (color, type, spacing)
- Components (with variants/properties)
- Patterns (page sections, flows)
- Content examples (error copy, empty states)
- Release notes page (what changed)

---

## 4) Tokens (The Most Important Layer)

### 4.1 Token model: base → semantic → component

- **Base tokens**: raw values (rarely used directly in UI)
  - Example: `blue.600`, `gray.50`, `space.4`, `radius.2`
- **Semantic tokens**: meaning and intent (used in components)
  - Example: `color.text.primary`, `color.bg.surface`, `color.border.subtle`
- **Component tokens (optional)**: per-component tuning without breaking semantics
  - Example: `button.primary.bg`, `input.border.focus`

Rule: product code should use **components** and **semantic tokens**, not base values.

### 4.2 Token naming conventions

Use predictable, hierarchical names:

- `color.text.{primary|secondary|muted|inverse}`
- `color.bg.{canvas|surface|elevated}`
- `color.border.{default|subtle|strong|focus}`
- `color.intent.{info|success|warning|danger}.{bg|text|border}`
- `space.{0..n}` and `size.{0..n}` (avoid ad-hoc values)
- `radius.{0..n}` and `shadow.{0..n}`

### 4.3 Dark mode and theming

Design for themes from day one:

- Tokens should support light/dark by swapping semantic values.
- Avoid “hard” mapping like `danger = red.600` everywhere; map it per theme.

Implementation options:

- CSS variables per theme (`[data-theme="dark"] { ... }`)
- Token build step that outputs `:root` variables + per-theme overrides

### 4.4 Token delivery best practices

- Export tokens in formats needed by the stack: CSS variables, JSON, TypeScript.
- Version tokens like an API: changes can be breaking.
- Provide a migration path and deprecations.

---

## 5) Color System

### 5.1 Rules

- No raw hex in application components.
- Use semantic roles, not brand names (avoid `color.blueButton`).
- Ensure contrast for text and interactive boundaries.

### 5.2 Minimum semantic set

- Background: `canvas`, `surface`, `elevated`
- Text: `primary`, `secondary`, `muted`, `inverse`, `link`
- Border: `default`, `subtle`, `strong`, `focus`
- Intent: `info`, `success`, `warning`, `danger`
- Interactive: `primary`, `primaryHover`, `primaryActive` (or state tokens)

### 5.3 States

Document state colors consistently:

- Default, hover, active, focus, disabled
- Include high-contrast focus ring token

---

## 6) Typography

### 6.1 Rules

- Prefer a small number of families (1–2).
- Use a type scale; avoid one-off font sizes.
- Use `rem` and line-height tokens.

### 6.2 Typographic roles

Define roles, not just sizes:

- `display`, `h1–h6`, `body`, `caption`, `code`

### 6.3 Content guidelines

- Labels: short, noun-first (“Email address”).
- Buttons: verb-first (“Save changes”).
- Errors: explain what happened and how to fix it.

---

## 7) Spacing, Sizing, Layout

### 7.1 Spacing scale

Use a consistent scale (commonly 4px or 8px-based). Example (4px base):

- `space.0 = 0`
- `space.1 = 4px`
- `space.2 = 8px`
- `space.3 = 12px`
- `space.4 = 16px`
- `space.6 = 24px`
- `space.8 = 32px`

Rules:

- Components own **internal spacing**.
- Layout controls **external spacing**.
- Avoid arbitrary “magic numbers”.

### 7.2 Breakpoints

Define a small set of breakpoints and use them everywhere:

- `sm`, `md`, `lg`, `xl`, `2xl`

---

## 8) Component Design Rules

### 8.1 Component contract

Every component must define:

- Purpose and non-goals
- Props/inputs and events/outputs
- Variants and sizes
- States: hover/active/focus/disabled/loading/error (as applicable)
- Accessibility behavior (keyboard, ARIA, labeling)
- Do/don’t guidance

### 8.2 Prefer variants over new components

Bad:

- `PrimaryButton`, `BlueButton`, `SubmitButton`

Good:

- `Button` with `variant="primary" | "secondary" | "danger"`

### 8.3 Composition over configuration

- Provide escape hatches via composition (slots/children) rather than 30 props.
- Keep “smart” components rare; default to presentational components.

### 8.4 Controlled vs uncontrolled inputs

- Inputs should support controlled usage.
- Uncontrolled support is optional, but never hide state.
- Expose events and value changes clearly.

### 8.5 Resilience

Design components to handle:

- Long text and truncation
- Empty/undefined values
- Slow networks (loading states)
- Disabled/read-only states
- Validation errors

---

## 9) Accessibility (A11y)

### 9.1 Non-negotiables

- Full keyboard navigation
- Visible focus indicator
- Semantic HTML first
- Accessible names for interactive controls
- Error messaging tied to inputs

### 9.2 Common patterns

- Dialogs: focus trap, close on Escape, restore focus
- Menus: arrow-key navigation, correct roles
- Forms: label + described-by for help/error text
- Tables: consider responsive alternatives, row/column headers

### 9.3 Testing

- Automated: axe-core / eslint-plugin-jsx-a11y
- Manual: keyboard-only walkthroughs and screen reader spot checks

---

## 10) Documentation & Examples

### 10.1 Component docs page checklist

Include:

- Overview and intended usage
- Variants, sizes, and states
- Accessibility notes
- Best practices and anti-patterns
- Examples for common scenarios

### 10.2 Living documentation

Recommended:

- Storybook (or similar) as the canonical component explorer
- Auto-generated prop tables (when feasible)
- “Recipes” section for patterns

---

## 11) Engineering Quality Practices

### 11.1 Testing pyramid

- Unit tests for logic and utilities
- Interaction tests for components (keyboard + pointer)
- Visual regression tests for styling changes

### 11.2 Versioning

Use semantic versioning for the design system packages:

- **Major**: breaking API or token changes
- **Minor**: new components/props, backwards compatible
- **Patch**: fixes

### 11.3 Deprecation policy

- Mark deprecated APIs in docs and types.
- Provide a migration guide.
- Remove only in a major release.

---

## 12) Governance & Contribution

### 12.1 Ownership model

Define who owns:

- Tokens and theme decisions
- Component APIs
- Accessibility acceptance
- Release process

### 12.2 When to add a component

Add a component when it:

- Is needed in 2+ places, and
- Represents a stable pattern, and
- Has clear a11y behavior and states

Don’t add when it’s:

- A one-off for a single screen
- Product-specific business logic
- A temporary experiment

### 12.3 Contribution workflow (recommended)

1. Proposal (brief RFC) with screenshots + use-cases
2. Token impact assessment
3. Design review (variants/states)
4. Engineering review (API + a11y)
5. Implementation + tests + docs
6. Release notes + version bump

---

## 13) Adoption & Rollout Strategy

### 13.1 Start with foundations

- Tokens + typography + spacing + a11y baseline
- Then buttons/inputs/navigation

### 13.2 Measure progress

Track:

- % of screens using design system components
- Accessibility issues over time
- Release cadence and upgrade friction

### 13.3 Migration tactics

- Provide codemods where feasible
- Ship adapter components temporarily
- Prioritize shared layouts and form controls

---

## 14) Practical Checklists

### 14.1 New component checklist

- [ ] Clear purpose and scope
- [ ] Variants + sizes defined
- [ ] States implemented and documented
- [ ] Keyboard interaction defined
- [ ] Accessible naming and labeling supported
- [ ] Visual regression coverage exists
- [ ] Story/examples added
- [ ] Tokens used (no raw values)

### 14.2 Token change checklist

- [ ] Change classified as breaking/non-breaking
- [ ] Both light/dark updated
- [ ] Contrast checked
- [ ] Migration note written

---

## Appendix: Example token output (CSS variables)

```css
:root {
  --color-text-primary: #111827;
  --color-text-muted: #6b7280;
  --color-bg-canvas: #ffffff;
  --color-bg-surface: #f9fafb;
  --color-border-default: #e5e7eb;
  --focus-ring: 0 0 0 3px rgba(59, 130, 246, 0.4);
}

[data-theme="dark"] {
  --color-text-primary: #f9fafb;
  --color-text-muted: #9ca3af;
  --color-bg-canvas: #0b1220;
  --color-bg-surface: #111827;
  --color-border-default: #1f2937;
  --focus-ring: 0 0 0 3px rgba(147, 197, 253, 0.4);
}
```
