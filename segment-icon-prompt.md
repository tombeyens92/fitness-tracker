# Prompt for Claude Code

Build a clickable three-segment donut icon component. Match the framework, styling
approach (CSS modules / Tailwind / plain CSS) and file conventions already used in
this repo — inspect a couple of existing components first and follow them.

## Geometry

SVG with `viewBox="0 0 512 512"`, centre 256,256, outer radius 230, inner radius 141,
13° gaps between segments. Use these three exact `<path>` elements — do not
recalculate the arcs:

- purple `#9783F7` — `M230.0 27.5A230 230 0 0 0 45.1 347.7L126.7 312.2A141 141 0 0 1 240.0 115.9Z`
- teal `#70CCC2` — `M71.1 392.8A230 230 0 0 0 440.9 392.8L369.3 339.9A141 141 0 0 1 142.7 339.9Z`
- amber `#E8B345` — `M466.9 347.7A230 230 0 0 0 282.0 27.5L272.0 115.9A141 141 0 0 1 385.3 312.2Z`

Use filled paths, not stroked circles with `stroke-dasharray` — hit-testing on a
dashed stroke is unreliable across browsers.

## States

Each segment sits in its own `<g>` with `transform-origin: 256px 256px`.

- Inactive: fill opacity `0.34`
- Hover (only under `@media (hover:hover)`): opacity `0.62`, `cursor: pointer`
- Active: opacity `1` plus a translate along the segment's own bisector, so it nudges
  outward and its gaps widen:
  - purple `translate(-13.9px, -8px)`
  - teal `translate(0, 16px)`
  - amber `translate(13.9px, -8px)`
- Transition: `opacity .18s ease, transform .18s cubic-bezier(.2,.8,.3,1)`
- Respect `prefers-reduced-motion: reduce` — drop the transform, keep the opacity change.

## Behaviour and API

The three segments are three states of one control, so model it as a radio group,
not three independent buttons:

- `<svg role="radiogroup">`, each `<g role="radio">` with `aria-checked` and an
  `aria-label`.
- Roving tabindex: the active segment has `tabindex="0"`, the others `-1`.
- Arrow keys move between segments and select; Enter and Space select.
- Visible focus ring via `:focus-visible` — add a transparent duplicate path inside
  each group and give that a white stroke on focus, so the ring traces the wedge
  shape rather than a bounding box.

Component props: `value` (which segment is active), `onChange(value)`, `size` in px
(default 230), and an `interactive` flag. When `interactive` is false, render
`role="img"` with an `aria-label`, no tab stops, no hover, no pointer cursor.

Segment keys and labels should be passed in rather than hardcoded — the colours stay
fixed but the labels are caller-supplied.

## Constraint to enforce

Each wedge is about a third of the icon, so below roughly 88px total the tap targets
fall under the 44px minimum. Below that size the component should render in
non-interactive mode automatically, and log a dev-only warning if `interactive` is
passed at a smaller size.

Add a small demo/story page showing it at 24, 32, 48 and 230px.
