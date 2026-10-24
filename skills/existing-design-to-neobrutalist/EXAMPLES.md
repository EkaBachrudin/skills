# Tailwind Patterns

These are styling patterns, not drop-in replacements. Preserve the real component's content, semantics, props, and behavior; change only the presentation needed for the requested result. Adapt the accent and existing states consistently.

## Button: hard edges and mechanical states

The handler and button semantics stay as supplied. Use `type` only as already required by the original form behavior.

```tsx
<button
  onClick={handleSubmit}
  className="min-h-[44px] px-6 py-3 border-2 border-black rounded-none bg-[#FFD700] text-black uppercase font-black tracking-wide shadow-[4px_4px_0px_0px_#000000] transition-transform duration-100 hover:-translate-x-1 hover:-translate-y-1 hover:shadow-[6px_6px_0px_0px_#000000] active:translate-x-1 active:translate-y-1 active:shadow-none focus-visible:outline-none focus-visible:ring-4 focus-visible:ring-black focus-visible:ring-offset-2 motion-reduce:transition-none motion-reduce:transform-none"
>
  Submit Action
</button>
```

The 4px resting shadow, 4px pressed translation, and stronger offset on hover make the movement read as mechanical. Keep keyboard focus visible and ensure reduced-motion users do not depend on movement for feedback.

## Card: visible compartments

```tsx
<article className="border-4 border-black rounded-none bg-white text-black shadow-[8px_8px_0px_0px_#000000]">
  <header className="border-b-4 border-black p-4">
    <h2 className="text-4xl font-black leading-none">Account details</h2>
  </header>
  <div className="p-4">
    <p>Existing card content stays unchanged.</p>
  </div>
</article>
```

Use the card's existing semantic element and content. Add a hard section line only where the design has a real header/body boundary.

## Controlled input: preserve its contract

Keep the original `id`, label association, value, handler, validation, and helper/error text. Only the classes change.

```tsx
<label htmlFor="email" className="mb-2 block font-bold text-black">
  Email address
</label>
<input
  id="email"
  name="email"
  value={email}
  onChange={handleEmailChange}
  aria-invalid={hasEmailError}
  className="w-full border-2 border-black rounded-none bg-white p-4 text-black focus:border-4 focus:border-black focus:ring-0 focus:shadow-[4px_4px_0px_0px_#000000] focus-visible:outline-none"
/>
```

Do not add or remove controlled/uncontrolled behavior, validation, or ARIA attributes as part of a visual restyle.

## Badge and divider

```tsx
<span className="inline-flex border-2 border-black rounded-none bg-[#FFD700] px-3 py-1 text-black font-bold uppercase tracking-wide">
  Featured
</span>

<hr className="w-full border-0 border-t-4 border-black" />
```

If the original divider is not an `<hr>`, preserve its semantics and style its existing element instead.

## Quick mapping from soft UI

| Existing treatment | Neo-Brutalist direction |
| --- | --- |
| `shadow-sm`, `shadow-md`, blur | `shadow-[4px_4px_0px_0px_#000000]` or a larger hard offset |
| Faint gray border | `border-2 border-black` or `border-4 border-black` |
| `rounded-lg`, `rounded-full` on structural surfaces | `rounded-none` (or minimal rounding only when justified) |
| Pastel/gradient/opacity background | White or a consistent saturated accent with black text |
| Whitespace-only card sections | A visible black border at real section boundaries |
| Soft focus glow | Strong `focus-visible` ring or hard black focus shadow |

Do not apply the mapping mechanically to every wrapper. Preserve a clear hierarchy, sufficient spacing, responsive layout, and all original interaction behavior.
