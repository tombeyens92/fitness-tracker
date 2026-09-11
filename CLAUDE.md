# Strength Rotation — project context

A bodyweight training tracker. Single static page, no build step, no
dependencies, no backend. Deployed to GitHub Pages.

    index.html              everything — markup, CSS, vanilla JS
    manifest.webmanifest    PWA metadata
    sw.js                   service worker, cache-first shell
    *.png                   icons

## Non-negotiables

- **No dependencies, no build step, no framework.** Everything lives in
  `index.html`. If a change seems to need a bundler, it's the wrong change.
- **No localStorage-free rewrites.** State is one JSON blob under the key
  `strength:v1`. Don't shard it across keys.
- **Bump `CACHE` in `sw.js` on every change to a shell file.** Otherwise the
  service worker keeps serving the old version and the change never reaches
  an installed phone. This is the single most-forgotten step.
- **Mobile first.** The target is an iPhone home-screen PWA. Anything wider
  than ~380px of content must wrap, not overflow.

## The training model — and why it's shaped this way

Three days rotate: Push (A), Core (B), Legs + Pull (C). The rotation is
**position-based, not date-based**: it tracks what's *next*, never what the
calendar says. Missing a day costs nothing and must never be punished — the
plan waits. Don't add catch-up mechanics, debt counters, or consecutive-day
streaks; they push people to train when they shouldn't, which contradicts the
programme's own advice about rest.

A "week" is **7 logged sessions**, not 7 calendar days.

### Slots, not fixed exercise lists

Each day is a list of **movement pattern slots** (`DAYS[].slots`). The user
picks which exercise fills each slot; `poolFor(pattern)` derives the options
from each exercise's `pat` field. This guarantees pattern coverage — you
cannot accidentally build a day of four anti-extension holds and no hinge.

Patterns: `hpush`, `vpush`, `ext`, `antiext`, `antirot`, `antilat`, `squat`,
`hinge`, `lunge`, `pull`. `antiext2` is a second anti-extension slot drawing
from the same pool.

**Adding an exercise** = an `EX` entry with the right `pat` + a `FIGS`
drawing. It then appears in the builder automatically. `DEFAULTS` sets the
fresh-install pick.

### Week 4 and the double-counting trap

Weeks 1–3 add sets then reps. Week 4 keeps volume flat and swaps in harder
variations. Two rules follow:

- If a week-4 variation is genuinely a *different movement* (two-legged glute
  bridge → single-leg), give the exercise `hardId` pointing at that catalogue
  entry. Week 4 then uses that exercise's own reps, `side` flag and drawing.
- **Substituted exercises skip week 4's rep bonus.** The harder movement is
  already the progression; adding reps on top raises load twice. This bug has
  been introduced and fixed twice — don't reintroduce it.

When changing any rep target, sanity-check the *per-limb* load across the week
boundary, not just the printed number. 3×17 two-legged ≈ 8.5 single-leg
equivalents per leg; 3×12 per side is a 40% jump, 3×10 is reasonable.

### Reps and sides

- `side: true` means the printed target is **per side**.
- Per-side exercises are done in blocks (all reps one side, then swap) —
  **except dead bug**, which alternates every rep because anti-rotation is the
  whole point. Its cue says so.
- `halves()` — timed *and* per-side (side plank) — counts in halves: one timer
  run fills half a set dot. Progress for these is stored in half-units, so the
  cap in `sanitize` is 6, not 3.

### Stretches

`STRETCH` entries with `SFIGS` drawings, shown as an optional cooldown.
**They never count toward day completion and never touch the streak.** They're
also explicitly labelled as after-training: static stretching beforehand
reduces force output. Don't gamify them.

## State

`sanitize()` is the **only** way state enters the app — on load and on
restore. It is deliberately forgiving: unknown exercise ids dropped, missing
fields defaulted, bad dates filtered, legacy field names migrated, and a log
with no colour data has it reconstructed from rotation order. Any new state
field must be handled there, with a fallback, or old backups break.

Two sources of truth for the same fact is the bug pattern that has caused most
of this project's issues (session counter vs log dates drifting apart). Prefer
deriving over storing. `cycleCell()` already re-derives sessions and next-up
from the grid; keep it that way.

## Visual language

Colour is **semantic, not decorative**: amber = Push, teal = Core, violet =
Legs + Pull. It appears on the dial, the muscle map, the streak grid, and the
plan panel, always meaning the same thing. `--acc` is scoped per day on `#app`
and re-scoped locally inside `.plan` when previewing another day.

Exercise drawings are chalk-line figures, `viewBox="0 0 100 62"`, with a dashed
floor line, built from the `fig`/`pl`/`ci` helpers. Keep the style consistent —
stroke only, no fills, round caps.

## Testing

There is no test framework. Verify changes headlessly with jsdom:

```bash
npm i --no-save jsdom
node - <<'EOF'
import { JSDOM } from "jsdom"; import fs from "fs";
const d = new JSDOM(fs.readFileSync("index.html","utf8"),
  {runScripts:"dangerously", url:"https://example.com/"});
// click through [data-act="..."] handles; re-query after every click,
// because the app re-renders and earlier node references go stale.
EOF
```

Always assert: uncaught errors is empty, an old backup file still restores,
and week-4 targets are sane per limb. Remove `node_modules` before committing.

## Interaction wiring

All clicks go through one delegated listener keyed on `data-act`. Adding UI
means adding a `data-act` case, not a new listener. The app re-renders wholesale
on most actions, so never hold DOM references across a state change.
