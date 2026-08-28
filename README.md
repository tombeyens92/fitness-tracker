<<<<<<< HEAD
# fitness-tracker
=======
# Strength Rotation — PWA

Standalone bodyweight tracker. No build step, no dependencies, no backend.

## Files

    index.html              the whole app (markup, CSS, JS)
    manifest.webmanifest    PWA metadata
    sw.js                   service worker (offline shell cache)
    icon-192.png            \
    icon-512.png             |  home-screen icons
    icon-maskable-512.png    |
    apple-touch-icon.png    /

## Hosting

Any static host works. It must be served over **https** (or localhost) —
service workers and install prompts are refused on plain http.

    # GitHub Pages
    push these files to a repo, Settings → Pages → deploy from branch

    # Netlify / Cloudflare Pages
    drag the folder onto the dashboard

    # local test
    python3 -m http.server 8000     # then open http://localhost:8000

Serve from the directory root, or from a subpath — every reference is
relative, so `/tracker/` works as well as `/`.

## Installing

- **iOS** — open in Safari, Share → Add to Home Screen. Do this rather than
  bookmarking: Safari evicts localStorage for sites unopened for ~7 days,
  but home-screen apps are exempt.
- **Android / desktop Chrome** — browser menu → Install app.

## Data

Everything lives in `localStorage` under the key `strength:v1`, on the device.
There is no sync. Menu → Backup & restore copies the log out as JSON and
pastes it back in; that is how you move between devices.

The restore path is defensive: unknown exercise ids are dropped, missing
fields fall back to defaults, bad dates are filtered, and a log with no
colour data has it reconstructed from rotation order. Files from any earlier
version load.

## Updating

After editing any shell file, bump `CACHE` in `sw.js` (e.g. `strength-v2`).
Otherwise the old cached copy keeps being served.

## Adding an exercise

`EX` holds the definitions, `DAYS` assigns them, `FIGS` and `HARD_FIGS` hold
the drawings. An array inside a day's `ex` list is a variant slot — the user
picks one, and the first entry is the slot's identity, so don't reorder it.
>>>>>>> 6b663eb (Strength rotation tracker)
