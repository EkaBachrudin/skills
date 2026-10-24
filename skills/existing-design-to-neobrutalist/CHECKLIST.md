# Review Checklist

## Scope and behavior

- [ ] Change is limited to the requested component/page and its presentation.
- [ ] Hooks, state, refs, effects, stores, fetching, mutations, handlers, props, types, and exports are unchanged.
- [ ] Existing conditional rendering, form submission/validation, IDs, names, values, links, keys, and visible text are preserved.
- [ ] Native elements and semantics remain appropriate; no button/link/input was replaced with a generic element for styling.
- [ ] Existing `aria-*`, labels, descriptions, live regions, keyboard behavior, and status/error messaging are preserved.

## Neo-Brutalist styling

- [ ] Cards, controls, dialogs, and other real structural surfaces have solid black `border-2` or `border-4` outlines.
- [ ] Shadows are hard black offsets with zero blur; no default Tailwind soft shadow remains.
- [ ] Corners are sharp (`rounded-none`) or only minimally rounded where justified.
- [ ] Surfaces use white or intentional high-saturation accents (mustard, hot pink, neon green, cyan); no muted pastels, gradients, blur, or opacity-based fading remain.
- [ ] Text, structural borders, and shadows use pure black; text/background pairs have adequate contrast.
- [ ] Large headings are assertive but responsive; labels and body text remain legible.
- [ ] Buttons are chunky, clearly interactive, and have visible rest/hover/pressed/focus states.
- [ ] Inputs remain easy to identify and use, with generous padding and a hard focus treatment.
- [ ] Actual card sections and content groups use visible dividers where needed; wrappers are not needlessly boxed.
- [ ] Overlapping borders, focus rings, and shadows are not clipped or hidden by stacking/overflow.

## Accessibility and responsive behavior

- [ ] Keyboard focus is easy to see and is not removed without an equivalent replacement.
- [ ] Links, buttons, and icon-only actions retain accessible names and their original semantics.
- [ ] Color is not the only signal for selected, invalid, success, or error states.
- [ ] Controls retain practical target sizes; content wraps and remains usable at narrow widths and zoom.
- [ ] Motion is nonessential and reduced-motion preferences are respected where motion is used.
- [ ] Disabled, selected, loading, and validation states remain understandable without opacity-based styling.

## Tailwind and output

- [ ] Styling uses only `className` Tailwind utilities/arbitrary values supported by the project.
- [ ] No dependencies, external CSS, inline styles, global config, or unrelated files were added or changed.
- [ ] Any available relevant checks were run, and no unrun check is claimed as successful.
- [ ] Final response contains only the requested component/file/snippet code, without fences or explanation.
