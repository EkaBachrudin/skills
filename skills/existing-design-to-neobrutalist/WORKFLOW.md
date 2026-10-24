# Transformation Workflow

Use this procedure for a component, page, or set of related components. Keep the change scoped to the requested UI; do not redesign unrelated screens or modify global project configuration.

## 1. Inspect the source and project conventions

- Read the complete supplied component and any directly relevant child components. If working in a repository, inspect the nearby styling patterns and Tailwind setup/version when available.
- Identify the requested output scope: a snippet, component, or complete file. Preserve that scope and its existing imports, exports, directives (including `"use client"`), props, and types.
- Note existing design tokens, responsive breakpoints, dark-mode behavior, and component variants. The requested Neo-Brutalist palette takes precedence for the requested surface, but do not silently change unrelated theme behavior or global tokens.
- Do not assume a plugin, custom Tailwind configuration, or utility exists. Prefer standard utilities and arbitrary values supported by the project's Tailwind version; do not edit the config to make a proposed class work.

## 2. Lock behavior and semantic invariants

Before styling, identify what must remain unchanged:

- React hooks, state, refs, effects, memoization, and context use.
- Event handlers and their bindings; props, callback signatures, and conditional rendering.
- Data fetching, mutations, stores, form actions, validation, and loading logic.
- Existing separation of concerns, component boundaries, and ownership of state/data. Do not move business or data logic into a view or create abstractions solely to apply styles.
- Element roles and native semantics; form names, values, labels, IDs, `href`, `type`, `disabled`, `aria-*`, keyboard behavior, and live-region attributes.
- Visible copy, meaningful icons/alternative text, list keys, and existing error or status messages.

Prefer changing `className` values. A purely presentational wrapper is acceptable only when necessary and when it does not alter layout behavior, semantics, event propagation, or accessibility. Never replace a native button/link/control with a generic element just to style it. Do not add, remove, rename, or reinterpret behavior.

## 3. Inventory the visual and interactive states

Map the component tree before editing. Identify structural surfaces, content hierarchy, controls, separators, overlays, and overlap. For each interactive element, check which existing states are present or implied by existing props:

- Rest, hover, active/pressed, keyboard focus, disabled, selected/current, expanded/collapsed, invalid, loading, success, and error.
- Narrow and wide layouts, long text, wrapping, overflow, and any existing theme modes.

Style only states supported by the source. Do not fabricate state or change interaction logic. Avoid putting a border on every anonymous wrapper: make actual cards, controls, panels, and other structural components read as distinct compartments, while preserving breathing room and hierarchy.

## 4. Apply a coherent Neo-Brutalist visual system

### Palette and contrast

- Use pure black (`#000000`) for text, structural borders, and hard shadows. Use white or high-saturation surfaces; the default accent palette is mustard `#FFD700`, hot pink `#FF69B4`, neon green `#39FF14`, and cyan `#00FFFF`.
- Use exact arbitrary hex values when a named Tailwind color would not match the specified palette. Keep accent use intentional—usually one or two accents per surface—and assign repeated roles consistently. Do not use every accent everywhere.
- Do not use pastel/faded colors, gradients, blur, `bg-opacity-*`, or opacity modifiers to soften surfaces. Avoid low-contrast text and do not communicate status through color alone. Keep text black on white or approved saturated surfaces; do not put text on a black decorative backdrop.
- Check contrast rather than assuming a palette passes: target at least 4.5:1 for normal text and 3:1 for large text; ensure important control boundaries and focus indicators are clearly distinguishable from adjacent colors.
- Preserve meaningful brand/status distinctions when the user explicitly requires them, but keep the requested high-contrast treatment and verify text contrast rather than assuming a color is accessible.

### Geometry, borders, and shadows

- Give actual structural elements (cards, buttons, inputs, panels, dialogs) solid black `border-2` or `border-4` outlines. Prefer `rounded-none`; use at most minimal rounding when it is necessary to match the UI.
- Replace soft/default shadows (`shadow-sm`, `shadow-md`, and similar) with solid, zero-blur black offsets, for example `shadow-[4px_4px_0px_0px_#000000]`, `shadow-[6px_6px_0px_0px_#000000]`, or `shadow-[8px_8px_0px_0px_#000000]` according to component scale.
- Use hard separators for actual section boundaries, such as `border-b-4 border-black`, and full-width dividers such as `border-t-4 border-black w-full`. Preserve spacing where a divider would add clutter.
- Keep overlapping borders visually distinct. Check stacking order, focus rings, and clipping/overflow so outlines are not hidden.

### Type, spacing, and component treatment

- Make true page/section headlines assertive: use a responsive `text-4xl`–`text-7xl` scale, `font-black`, and usually `leading-none`. Preserve semantic heading order and keep smaller component labels appropriately sized.
- Make buttons chunky (at least `px-6 py-3`), uppercase, `font-black`, and `tracking-wide`; maintain comfortable hit areas for icon-only controls too.
- Style cards as distinct compartments (commonly `border-4 border-black bg-white` with a hard shadow). Separate real header/body sections with visible hard lines, not whitespace alone.
- Style inputs with a black `border-2`, white background, generous padding (commonly `p-4`), and a sharp, visible focus treatment. Keep labels, help text, validation text, and control associations intact.
- Make badges/tags rectangular or minimally rounded, bordered, bold, uppercase, and high-contrast. Keep badges legible and avoid color-only meaning.
- Keep responsive composition, content order, and readable wrapping. Do not add decorative overlap if it obscures content or focus.

## 5. Style interactions without changing them

- For buttons, use a visible hard shadow at rest; on hover, a small negative translation with a stronger offset shadow; on active, a positive translation and removed shadow to create a pressed effect. Keep the shift proportional to the shadow. Do not rely on hover as the only affordance.
- Provide a strong, sharp keyboard focus indicator. A common Tailwind pattern is `focus-visible:outline-none focus-visible:ring-4 focus-visible:ring-black focus-visible:ring-offset-2`; remove an outline only when an equally visible replacement is present.
- Inputs should snap to a thicker black border or hard black shadow on focus, never a soft glow. Keep invalid/error cues, helper text, and `aria-invalid` behavior intact.
- Keep disabled controls visibly disabled without opacity-based fading; do not alter their existing disabled semantics or behavior.
- Keep motion brief and nonessential. Add reduced-motion variants for motion/transition utilities (for example `motion-reduce:transition-none motion-reduce:transform-none`) and ensure the interface remains clear without animation.

## 6. Verify before returning

- Compare source and result to confirm the behavioral invariants and user-facing content are unchanged.
- Check every existing state, control, focus treatment, and responsive layout in the inventory. Confirm no default soft shadows, low-contrast borders, pastel/transparent surfaces, or unsupported classes remain in the transformed scope.
- If a local preview is available, inspect narrow and wide viewports and keyboard focus. Otherwise perform a static review; do not claim a render or test that was not performed.
- When editing a repository, run the relevant existing lint/type checks if practical. Do not change unrelated files to satisfy them.
- Return only the requested code. If a missing source detail makes safe preservation impossible, ask a concise clarification instead of inventing logic.
