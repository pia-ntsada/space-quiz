# Copilot instructions

A space exploration quiz delivered as a single `index.html` (inline `<style>` and `<script>`). There is no build step, package manager, linter, or test suite. Open the file directly in a browser to run it. Keep it self-contained: no server, no external dependencies, no CDN assets.

## Architecture

Everything lives in `index.html`, in three layers:

- **Markup:** a `#game` section (question view) and a `#results` section, toggled with the `.hidden` class. Elements are looked up once into the `els` object by id.
- **Styles:** colours are CSS custom properties on `:root`, overridden in a `prefers-color-scheme: dark` block. Always use the variables (`--good`, `--bad`, `--accent`, `--card`, ...) rather than hard-coded colours so both themes keep working.
- **Script:** module-level state (`index`, `score`, `answered`, `order`) drives the flow `start()` → `render()` → `choose(i)` → `next()` → `showResults()`. Questions are objects in the `QUESTIONS` array: `{ q, a: [4 options], c: correctIndex, f: fact }`. `order` is a shuffled index list, so always reference questions as `QUESTIONS[order[index]]`.

## Conventions

- **Accessibility is a requirement.** Preserve these when changing UI:
  - Focus moves to the `h2#question` on each new question, to Next after answering, and to `#result-title` on the results screen.
  - Feedback is announced via `role="status"` / `aria-live="polite"`.
  - Answer state is never colour-only (✓/✗ marks are added).
  - Keyboard support: 1–4 / A–D answer, Enter activates Next.
  - Use `:focus-visible` outlines and keep the skip link.
- **Reduced motion:** every animation must be disabled under `prefers-reduced-motion`. The CSS block does this globally, and `countUp()` also checks `matchMedia` in JS and skips the count-up.
- **Score count-up:** `countUp()` sets an `aria-label` with the final score up front so screen readers don't hear each tick. Clear `countTimer` (as `start()` does) whenever the results screen is left.
- **Dynamic text** is inserted with `textContent`; `innerHTML` is only used for fixed strings.
- When adding or removing a question, give each exactly four options (the `LETTERS` array and key handler assume four) and update the hard-coded `<progress max="10">` and the "Ten questions" intro text. The counter text and results score derive from `QUESTIONS.length`.

## Verifying changes

There is no test runner. To exercise the logic headlessly, load `index.html` in `jsdom` (installed temporarily, e.g. in `/tmp`) with `runScripts: "dangerously"` and `pretendToBeVisual: true`. Stub `window.matchMedia` before the script runs, since jsdom doesn't provide it. Drive the quiz by dispatching `keydown` events on `document` and clicking `#next`.
