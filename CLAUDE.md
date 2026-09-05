# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is here

Two sibling folders, each a single self-contained `index.html`:

- `taha 2026/` — the live project. Git repo, remote `zehtabchiantaha964-droid/Taha-2026`, published via GitHub Pages at  https://tahazehtabchian.github.io/Taha-zehtabchian/iq-game/
- `iq-game/` — an untracked copy of the same file (byte-identical at last check). Not a separate project; if you change one and the copy is meant to stay in sync, copy the file over rather than editing twice.

## Commands

There is no build, no package manager, no test runner, and no dependencies. `.gitignore` mentions `node_modules/` but none exists.

- Run: open `taha 2026/index.html` directly in a browser (`start "taha 2026/index.html"`).
- Deploy: commit and push to `origin/main` — GitHub Pages serves the repo root.

Verification is manual: open the page, check both languages (top-right toggle), both themes, and a full run through the test.

## Architecture — `index.html`

One file, ~1500 lines: `<style>` (lines ~12–438), static markup for three screens (~440–538), then `<script>` (~540–end). The script is organized into ten numbered comment-banner sections; keep new code inside the matching section.

1. **Storage** — every `localStorage` access is wrapped in try/catch and prefixed `sanjeh.`. Keys: `lang`, `theme`, `best`, `session`. The page must stay fully functional when storage throws or returns nothing.
2. **I18N** — `I18N.fa` / `I18N.en` hold *every* user-visible string. `t(key)` reads the active language; `tx(obj)` unwraps a `{fa, en}` pair stored inside data. `num(n)` converts digits to Persian numerals when `LANG === 'fa'`. Adding UI text means adding a key to both language objects and assigning it in `applyLang()` — there is no template system, `applyLang()` sets each element's text by id.
3. **SVG primitives** — no images. Every visual stimulus is drawn from a compact spec object: `fig(spec)` renders one 100×100 figure by `spec.k` (`circ`, `poly`, `dots`, `polydots`, `nest`, `arrow`, `glyph`, `grid2`, `dotbox`, `fold`, `q`), and `matrix()` / `strip()` / `solo()` / `isoStack()` compose those into a stimulus. New figure types are a new `case` in `fig()`.
4. **Question bank** — `BANK` is 30 item objects, six per level, levels 1→5. Item shape: `{id, level, cat, prompt:{fa,en}, ans, why:{fa,en}}` plus exactly one stimulus field (`seq`, `matrix`, `strip`, `iso`, `solo`, or none for pure verbal) and `opts` (each `{n}`, `{t:{fa,en}}`, or `{s: figSpec}`). `ans` is the index into the *authored* `opts` order; options are shuffled per session (`S.order`) and `choose()` maps back through it. `cat` must be one of `numerical | verbal | logical | spatial` — those keys drive `I18N.cat` and the result breakdown.
5. **Scoring** — difficulty-weighted raw score (`WEIGHT` 1.0→2.0 by level) over `MAXRAW`, mapped to a z-score through the piecewise `ANCHORS` curve (deliberately not linear), then `IQ = 100 + 15z`, clamped 65–145. Speed adds at most ±3 and only when ≥80% of items were answered. `phi()` is a normal CDF used for the percentile. Changing `BANK` changes `MAXRAW` automatically; changing the mapping means editing `ANCHORS`.
6. **State** — one plain object `S` (`screen`, `idx`, `answers`, `order`, `elapsed`, `remaining`, `tick`, `result`). No framework, no reactivity: mutate `S`, then call the matching `render*()`. `saveSession()` persists the resumable subset; `finish()` clears it and updates `best`.
7. **Chrome** — `applyLang()` also flips `documentElement.lang`/`dir` (fa = RTL, en = LTR) and re-renders the current screen. `applyTheme(mode)` sets `data-theme` on `<html>`; `null` means "follow the system". CSS defines the light palette on bare `:root`, then overrides tokens under both `@media (prefers-color-scheme: dark) :root:not([data-theme="light"])` and `:root[data-theme="dark"]` — a new color must be defined in all the places the existing ones are, or one mode breaks. The top gauge shows measurement *uncertainty* (narrows as items are answered), not the score.
8–10. **Screens / flow / wiring** — `show(name)` toggles the three `<section>`s via `hidden`. Keyboard: `1`–`4` or `a`–`d` pick an option, `Enter` advances; arrow keys move focus *and* select, and "forward" flips with the language (`ArrowLeft` is forward in Persian). Animations are skipped under `prefers-reduced-motion`.

## Conventions

- Plain ES5-ish JS in `"use strict"`, no modules, no build step — everything must run from `file://`. The only external resource is the Google Fonts stylesheet.
- Bilingual by construction: never hardcode a user-visible string outside `I18N` or a `{fa, en}` pair, and never assume LTR in CSS — the stylesheet already uses logical properties (`margin-inline`, `text-align: start`); the only physical `left`/`right` are direction-neutral corner marks on `.plate`.
- Naming trap: the CSS tokens `--right` / `--right-soft` / `--wrong` are **correctness** colors (green/red for answer review), not direction. Nothing in the palette encodes left or right.
- Accessibility is already wired (`role="radiogroup"`, `aria-live` on the item counter, `aria-label` on figure plates, `role="timer"`); preserve it when editing markup.
- The framing is "an instrument, not a quiz app" — the README calls the result an entertainment-grade estimate, not a clinical measure. Keep that disclaimer intact.
