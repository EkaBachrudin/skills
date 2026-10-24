---
name: existing-design-to-neobrutalist
version: 1.0.0
description: >-
  Use when asked to refactor, restyle, or convert an existing React/Next.js
  interface into a Neo-Brutalist UI with Tailwind CSS, such as “make this
  component neo-brutalist” or “convert this existing design.” Preserve behavior
  and accessibility while replacing soft styling with bold black borders, hard
  shadows, sharp geometry, saturated accents, and mechanical interaction
  states. Not for unrelated UI implementation or application-logic changes.
---

# Existing Design to Neo-Brutalist

Restyle the supplied UI as a deliberate, structured Neo-Brutalist interface—not as visual noise. Keep the existing product usable, accessible, and behaviorally unchanged.

## Use this skill when

- The user wants an existing React or Next.js component/page restyled in Neo-Brutalist fashion with Tailwind CSS.
- The request mentions hard shadows, bold outlines, sharp geometry, high-contrast color, or mechanical interaction states.

## Operating rules

1. **Preserve behavior.** Change presentation, not application logic. Keep hooks, handlers, state, data flow, props, types, exports, form behavior, and meaningful semantics intact.
2. **Respect architecture.** Keep styling at the presentation layer and preserve component boundaries and state ownership. Do not move business/data logic or add abstractions solely for visual styling.
3. **Use the requested visual system.** Use black structural borders and shadows, sharp corners, white or saturated surfaces, large purposeful typography, and visible dividers. Avoid soft/default shadows, blur, gradients, muted pastels, and opacity-based styling.
4. **Keep it Tailwind-only.** Use `className` utilities and supported arbitrary values. Do not add dependencies, external CSS, inline style objects, or project-wide config changes.
5. **Keep it accessible and responsive.** Preserve native controls and accessible names, provide visible keyboard focus, maintain contrast and usable targets, and respect reduced-motion preferences.
6. **Style the whole state system.** Account for existing hover, active, focus, disabled, selected, loading, error, and responsive states; do not invent application states or behavior.
7. **Return only code for a code transformation.** Return the complete supplied component/file or snippet at the same scope, without Markdown fences or explanation. If essential source or context is missing, ask only for what is needed instead of guessing.

## Workflow and references

1. Follow [WORKFLOW.md](WORKFLOW.md) to inspect, transform, and verify the UI.
2. Use [CHECKLIST.md](CHECKLIST.md) before returning code.
3. Consult [EXAMPLES.md](EXAMPLES.md) for Tailwind patterns; adapt them to the actual component and project.

## Done when

- Visual styling is distinctly Neo-Brutalist and consistent across component variants.
- Existing behavior, content, semantics, and accessibility affordances are preserved.
- The result uses only project-compatible Tailwind classes and is returned in the requested code-only format.
