---
name: ui-development
description: Decide whether to reuse, extend, compose, or create a UI component, and build it consistently with the design system, accessibility, responsiveness, and maintainability standards. Use whenever the user asks to build, add, change, or style a screen, page, form, modal, button, card, layout, or any front-end component, or mentions design system, Tailwind, shadcn, React, or Next.js UI. Trigger before writing any new component, even if the user does not ask about reuse.
---

# UI Development

Before writing any new UI, find out what already exists. Prefer reuse. Create only with a reason. Every component meets accessibility, responsiveness, and maintainability standards.

## Rules

1. **Search before you build.** Always look in the component library, design system, and feature folders before creating anything.
2. **Use design tokens.** Colors, spacing, typography, radii, and shadows come from the design system or theme. No arbitrary hex values or magic pixel numbers unless the system has no equivalent (and then note it).
3. **No one-off copies.** Do not duplicate an existing component with small tweaks. Extend it with a prop or variant instead.
4. **Accessibility is not optional.** It is part of "done".
5. **Do not add a new UI library** without asking the user.
6. **Match the stack.** Use the project's framework, styling approach, and file conventions (check neighboring components first).

## Procedure

### Step 1: Understand the need
Capture: what the UI does, states it must handle (loading, empty, error, success, disabled), data it needs, and where it will be used (one place or many).

### Step 2: Search for existing components
- Check the shared component directory, design-system package, and feature-level components.
- Search by purpose and by name variants (e.g. "Modal", "Dialog", "Drawer"; "Button", "Action").
- Read the matching components' props, variants, and stories/docs.

### Step 3: Make the decision
Follow this order and stop at the first that fits:

| Option | Choose when | Action |
|--------|-------------|--------|
| **1. Reuse** | An existing component covers the need as is | Import and use it |
| **2. Configure** | It fits with existing props or variants | Use props/variants |
| **3. Extend** | It is close and the missing behavior is generally useful | Add a prop/variant to the shared component, keeping backward compatibility and updating docs/tests |
| **4. Compose** | The need is a combination of existing parts | Build a new component from existing primitives |
| **5. Create** | Nothing suitable exists | Create a new component in the correct shared or feature folder, following the project's conventions |

Write a one-paragraph **decision note**: what you searched, what you found, which option you chose, and why.

### Step 4: Build to the standards

**Design-system consistency**
- Use tokens and existing primitives (buttons, inputs, typography, layout).
- Follow naming, file structure, and export patterns of neighboring components.

**Accessibility (aim for WCAG 2.1 AA)**
- Use semantic HTML first (`button`, `nav`, `label`, headings in order); ARIA only when semantics are not enough.
- Everything operable by keyboard with a visible focus state; logical tab order; modals trap and restore focus.
- Form inputs have labels; errors are linked to fields and announced.
- Images have meaningful `alt` (or empty alt if decorative); icon-only buttons have an accessible name.
- Color contrast meets AA; meaning is never conveyed by color alone.
- Respect reduced-motion preferences.

**Responsiveness**
- Mobile-first; verify at small, medium, and large breakpoints and with long text.
- Use flexible layouts (flex/grid, relative units); no fixed widths that overflow.
- Touch targets comfortably sized (about 44px).

**Maintainability**
- Single responsibility; split when a component grows hard to read.
- Typed props with sensible defaults; avoid prop explosion (use composition or variants).
- State lives at the lowest sensible level; no duplicated state.
- Handles loading, empty, and error states explicitly.
- Add or update tests and a story or usage example where the project does so.
- No inline duplication of styles that exist as tokens or utilities.

### Step 5: Self-check and report
```
# UI Change: <component/screen>

**Decision:** Reuse / Configure / Extend / Compose / Create
**Reason:** <what existed, why this option>

## Files changed
- <path>: <what and why>

## Checklist
- [ ] Uses design tokens and existing primitives
- [ ] Keyboard and focus behavior verified
- [ ] Labels, alt text, and contrast checked
- [ ] Responsive at small / medium / large
- [ ] Loading, empty, error states handled
- [ ] Props typed and documented; tests/story added

## Follow-ups
<design-system gaps, tech debt, things to confirm with design>
```

## Limits
An agent cannot fully judge visual quality or real-world accessibility. Recommend a manual keyboard pass, a screen-reader spot check, and design review for new patterns.
