# CHORD HIIT

A single-file guitar chord flashcard web app. No build step, no dependencies, no backend — everything (HTML, CSS, JS) lives in `index.html`. Open it directly in a browser or deploy it to any static host.

## Deploy

This is a static site consisting of exactly one file: `index.html`. Any of these work with zero configuration:

- **Vercel**: `vercel --prod` from this directory (or connect the repo in the dashboard — Framework Preset: "Other", no build command, output directory `/`). `vercel.json` is included with minimal config (clean URLs).
- **Netlify**: drag-and-drop this folder in the Netlify dashboard, or `netlify deploy --prod --dir .`
- **GitHub Pages**: push this folder to a repo and enable Pages on the branch/root.
- **Anything else**: it's one static HTML file — any host that serves static files works (S3 + CloudFront, Cloudflare Pages, nginx, etc). Just make sure `index.html` is served at the root.

There is no `package.json` and intentionally no build step — don't add a bundler/framework unless the user asks for one; it would be a regression from the current zero-dependency setup.

## What it is

A shuffled/ordered deck of guitar chord flashcards (~619 chord voicings across all 12 keys and common qualities: maj7, m7, 7, m7♭5, dim7, 6, 9, 13, sus2/4, add9, power chords, etc.), each rendered as an inline SVG fretboard diagram generated at runtime — no image assets. Built for quick visual-recognition drilling (hence "HIIT"): autoplay with adjustable speed, optional delayed reveal, shuffle, key/quality filtering, a full searchable chord grid, and a set of named chord progressions (pop/rock/blues/jazz/samba standards) that play the deck in a fixed order instead of shuffled.

## Structure (all inside `index.html`)

- **`<style>`** — everything is CSS custom properties driven. `--ink*` (dynamic, per-chord/tile text color), `--panel-ink*` (fixed, toolbar-only, always light against the toolbar's dark glass), `--g-ink*` (page-level, grid/progressions screens, auto-picked for contrast against the customizable page background).
- **Music/chord data** — `NOTES`, `LIB` (the full chord library, built at load from voicing templates), `PROGS` (the named progression list, each entry tagged with a `level`: Beginner / Intermediate / Advanced).
- **`diagramSVG(c)`** — generates the fretboard diagram SVG for a given chord at render time.
- **Theming/palettes** — `BUILTIN_PALETTES` (the four baked-in defaults: Sunset, Terra, Amber Grove, Deep Verse — Deep Verse loads by default), `PALETTES` (the live, user-editable list, persisted to `localStorage['chordhiit_palettes']`), `applyPaletteObject()` (the single entry point for switching/previewing a palette — recomputes every chord's background + text-ink color). Per-natural-key (A–G) light/dark text overrides live on each palette as `keyInk`; sharps always auto-pick contrast via `pickInk()`.
- **Settings overlay** — full in-app theme editor (page background, dark/light text colors, per-key colors + ink toggle, palette create/delete/duplicate, and a "Copy palette data" button that exports the whole library as JSON to the clipboard for backup/handoff).
- **Splash/loader** — `runSplash()` plays a ~2.5s strobe-through-theme-colors + wordmark fade-in on first load, and is replayed whenever the user hits Home (which also resets every session selection — focus, key, progression, shuffle, speed, reveal, play state — back to defaults).
- **Bottom toolbar / mobile menu** — `#panel` is the desktop toolbar. At ≤760px it collapses: a single `.menu-fab` trigger ("Controls") replaces it, and `#panel` *itself* restyles into a full-viewport sheet toggled by `body.menuopen` (see `setMenu()`). The controls are never duplicated — same DOM, same ids, same handlers — so there is no state to sync between two copies. Mobile-only affordances are `.mlbl` (per-group labels), `.panel-head`, and `.icon-row` (`display:contents` on desktop so buttons stay direct flex children of `.grp`, a real flex row in the sheet). Desktop rendering is unchanged.
- **Progressions page** — each row now shows a colored difficulty badge (Beginner/Intermediate/Advanced) next to its name, tinted via the same `tint()` helper used for chord accent colors so it stays legible against any page background.

## Local preview

No server or build needed — just open `index.html` in a browser. (Some browsers restrict `localStorage`/clipboard APIs on `file://` origins more than on `http://`; if palette save/copy behaves oddly when opened directly from disk, serve it locally instead, e.g. `python3 -m http.server` from this directory and visit `localhost:8000`.)

## Known constraints (by design, not bugs)

- The chord library only covers the qualities listed above — progressions were deliberately written to stay within that vocabulary (no m9, 7♭9, etc.) rather than reference chords that don't exist in `LIB`.
- Palette/theme edits are stored in the browser's `localStorage`, per-browser/per-device — there's no account or sync layer.
