# AGENTS.md


## Project

A single Tampermonkey/Violentmonkey/Greasemonkey userscript (`alza_autopager.user.js`) that turns Alza e-shop product listings into infinite scroll by auto-clicking the "Load more" (`.js-button-more`) button. No build system, no dependencies, no tests — the `.user.js` file *is* the deliverable.

## Architecture & Invariants

The whole script runs in one IIFE. The design deliberately avoids polling for scroll position:

- An `IntersectionObserver` (rootMargin `800px`) watches the "Load more" button and clicks it when it nears the viewport. This is the hot path — it does the actual pagination.
- A cheap `setInterval(checkForButton, 2000)` re-attaches the observer when Alza's AJAX replaces the button node (the DOM element changes across page loads, so the observer target must be re-bound). This interval does *not* check scroll position — that's the observer's job.
- `clickButton()` guards against clicking a button that is hidden (`hdn` class), mid-load (`loading` class), disabled, or detached (`offsetParent === null`).

If you change pagination behavior, keep both mechanisms in sync: the observer handles "scrolled into view", the interval handles "button node swapped out".

## Commands

No automated tests. Verify manually: install the local `.user.js` in a userscript manager (point it at the file, or paste contents), open an Alza listing page (e.g. alza.sk), scroll, and confirm the console logs `[Alza - Infinite Scroll]` messages and pages load without repeated/duplicate clicks. The `.js-button-more` selector is the single point of coupling to Alza's markup — if pagination breaks, check whether that class still exists on the site.

## Conventions specific to this repo

- **Multi-region:** `@match` lines cover alza.sk/cz/de/at/hu. Adding a region = add a `@match` line; the selector is shared across all domains.
- **`@version` bump:** any functional change to the script must bump `@version` in the metadata block, or userscript managers won't offer the update via `@updateURL`.
- **`@downloadURL`/`@updateURL`** point at `github.com/NightMean/Alza_infinite_scroll` (note underscores — differs from the local folder name `Alza-Infinite-Scroll`). Keep them consistent with the actual remote.
- **Translations:** `translation/sk/README.md` mirrors the root `README.md`. Update both when changing user-facing docs.
- Console log prefixes are inconsistent (`[Alza - Infinite Scroll]` vs `[Alza Auto-Pager]`); pick one when touching logging.

## Decisions
- 2026-07-10: Chose `IntersectionObserver` + 2s node-recheck interval over position polling — lighter, avoids per-frame scroll math.

## Repository policy

Public repository (NightMean/Alza-Infinite-Scroll).
