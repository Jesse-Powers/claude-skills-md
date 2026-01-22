# 🎨 Web Application Design System — Guidelines & Best Practices

For a more complete, maintainable version of this guide, see: [design-system-guidelines.md](design-system-guidelines.md)

## Purpose

A design system is a **single source of truth** that aligns design, development, and product teams.

Its goals are to:

- Ensure visual and UX consistency
- Speed up development
- Reduce design debt
- Enable scalability across features and teams

---

# 1. Core Principles

## 1.1 Consistency over Creativity

- Prefer reuse over reinvention
- If a pattern exists, it must be used
- New patterns require justification

## 1.2 Design for Scale

- Every decision should assume:
  - More pages
  - More features
  - More developers
  - More users

## 1.3 Accessibility by Default

- WCAG 2.1 AA minimum
- Color contrast ≥ 4.5:1
- Keyboard navigation required
- Screen reader-friendly components

## 1.4 Single Source of Truth

- One component = one definition
- No visual logic inside product pages
- UI logic lives in the design system

---

# 2. Design System Architecture

```text
design-system/
├── foundations/
│   ├── colors
│   ├── typography
│   ├── spacing
│   ├── grid
│   ├── elevation
│   └── motion
│
├── components/
│   ├── buttons
│   ├── inputs
│   ├── forms
│   ├── navigation
│   ├── feedback
│   └── data-display
│
├── patterns/
│   ├── authentication
│   ├── dashboards
│   ├── tables
│   ├── empty-states
│   └── error-states
│
└── guidelines/
  ├── usage
  ├── accessibility
  └── do-dont
```

---

# 3. Foundations (Non-Negotiable)

## 3.1 Color System

### Rules

- No raw hex colors in components
- All colors must be semantic tokens

### Example

```css
--color-primary
--color-primary-hover
--color-success
--color-danger
--color-warning
--color-background
--color-surface
--color-border
--color-text-primary
--color-text-muted
```

### Best Practices

- Separate brand colors from UI semantic colors.
- Never couple meaning to a brand color.
- Support dark mode from day one.

## 3.2 Typography

### Font Roles

- Heading
- Body
- Mono (code, IDs, logs)

### Guidelines

- Max 2 font families
- Use a type scale (not random sizes)
- Avoid pixel values — use `rem`/`em`

### Example Scale

- xs → 12px
- sm → 14px
- md → 16px
- lg → 18px
- xl → 20px
- 2xl → 24px
- 3xl → 30px

## 3.3 Spacing System

### Spacing Token Scale

Use an 8px-based system:

- 0 → 0
- 1 → 4px
- 2 → 8px
- 3 → 12px
- 4 → 16px
- 5 → 20px
- 6 → 24px
- 8 → 32px
- 10 → 40px
- 12 → 48px

### Rules

- Never use arbitrary spacing
- Padding ≠ margin
- Components define internal spacing

## 3.4 Grid & Layout

### Layout Rules

- Page width tokens (e.g. 1200px / 1440px)
- Consistent gutters
- Predictable breakpoints

### Breakpoints Example

- sm → 640px
- md → 768px
- lg → 1024px
- xl → 1280px
- 2xl → 1536px

---

# 4. Component Design Rules

## 4.1 Component Definition

Each component must include:

- Purpose
- Variants
- States
- Accessibility behavior
- Do / Don’t examples

### Required States

- Default
- Hover
- Active
- Focus
- Disabled
- Loading
- Error (if applicable)

## 4.2 Variants over New Components

Bad:

- PrimaryButton
- BlueButton
- SubmitButton

Good:

```jsx
<Button variant="primary" />
<Button variant="secondary" />
<Button variant="danger" />
```

## 4.3 Controlled vs Uncontrolled

Rules:

- Inputs must support controlled usage
- Internal state must be optional
- No hidden magic

---

# 5. Component Categories

## 5.1 Inputs

- Text input
- Textarea
- Select
- Checkbox
- Radio
- Switch
- Date picker

Rules:

- Always show validation
- Error message below input
- Label always visible (no label-only placeholder)

## 5.2 Buttons

Button Types:

- Primary (1 per screen)
- Secondary
- Tertiary
- Destructive

Rules:

- Never use color alone to convey meaning
- Destructive actions must require confirmation

## 5.3 Feedback Components

- Toast
- Alert
- Modal
- Banner
- Tooltip

Rules:

- Toasts are non-blocking
- Modals block flow
- Tooltips are never required to complete actions

---

# 6. UX Patterns

## 6.1 Forms

- Group related fields
- One column preferred
- Inline validation
- Error shown after interaction

## 6.2 Empty States

Each empty state must include:

- Clear message
- Reason
- Action

Example:

"No projects yet. Create your first project to start tracking data."

## 6.3 Loading States

- Use skeletons over spinners
- Preserve layout
- Never block entire page unless necessary

---

# 7. Accessibility Guidelines

Mandatory Rules:

- Keyboard navigable
- Visible focus ring
- ARIA labels where required
- Semantic HTML first

Forbidden:

- Click-only interactions
- Color-only indicators
- Placeholder as label

---

# 8. Design ↔ Engineering Workflow

Source of Truth:

- Figma → visual intent
- Code → behavioral truth

Figma should never override component behavior.

Naming Convention:
Design and code must match (no creative naming):

- Button
- Input
- Select
- Modal
- Toast
- Dropdown
- Tabs
- Table

---

# 9. Versioning & Governance

When to Add Components:

- Used in 2+ places
- Represents a reusable pattern
- Approved by design + engineering

When NOT to Add:

- One-off UI
- Page-specific logic
- Temporary experiments

Versioning:

- Semantic versioning
- Breaking changes documented
- Changelog mandatory

---

# 10. Documentation Rules

Every component page must include:

- Description
- Variants
- Props
- Examples
- Accessibility notes
- Do / Don’t

Bad documentation = broken design system.

---

# 11. Common Mistakes to Avoid

- Designing pages instead of systems
- Hardcoding colors
- Multiple button styles
- Inconsistent spacing
- Ignoring empty/loading/error states
- Treating design system as static
