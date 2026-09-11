# Strength Rotation — PWA

Standalone bodyweight tracker. No build step, no dependencies, no backend.

## Files

    index.html              the whole app (markup, CSS, JS)
    manifest.webmanifest    PWA metadata
    sw.js                   service worker (offline shell cache)
    icon-*.png              home-screen icons
    CLAUDE.md               design invariants — read before changing anything

## Hosting

Static host over **https** (or localhost) — service workers and install
prompts are refused on plain http. Live at
`https://tombeyens92.github.io/fitness-tracker/`.

Local test: `python3 -m http.server 8000`

## Installing

- **iOS** — Safari → Share → Add to Home Screen. Do this rather than
  bookmarking: Safari evicts localStorage for sites unopened for ~7 days,
  but home-screen apps are exempt.
- **Android / desktop Chrome** — browser menu → Install app.

## Data

One JSON blob in `localStorage` under `strength:v1`, per device. No sync.
Menu → Backup & restore moves it between devices.

`sanitize()` is the only way state enters the app. It is deliberately
forgiving: unknown exercise ids dropped, missing fields defaulted, bad dates
filtered, legacy field names migrated, and a log with no colour data has it
reconstructed from rotation order. Any new state field must be handled there.

## Updating

**Bump `CACHE` in `sw.js` on every change to a shell file.** Otherwise the old
cached copy keeps being served to installed devices.

## Adding an exercise

See CLAUDE.md. Short version: an `EX` entry with the right `pat`, plus a
`FIGS` drawing. It appears in the builder automatically.
