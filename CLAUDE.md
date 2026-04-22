# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

No build step. Open `index.html` directly in a browser:

```bash
start "" "C:\Users\galo5\habit-tracker\index.html"   # Windows
```

There are no dependencies, no package manager, no test suite, and no dev server.

## Git workflow

Branch: `master`. Remote: `https://github.com/galo571-Git/habit-tracker`

After every change: commit with a conventional prefix, then push immediately.

```bash
git add index.html
git commit -m "feat: ..."   # or fix: / refactor: / style:
git push
```

Never batch unrelated changes into one commit.

## Architecture

Everything lives in a single `index.html` file with three sections: `<style>`, HTML body, and `<script>`.

**Data model** — stored in `localStorage` under key `ht-v1`:
```js
{ habits: [{ id: string, name: string, completions: string[] }] }
// completions: ISO date strings e.g. ["2026-04-22"]
```

**Rendering** — `render()` is the only render path. It calls `load()`, rebuilds the entire `#list-root` innerHTML from scratch, and updates the progress bar. There is no diffing or partial update.

**Actions** — all exposed as `window.*` functions and wired via inline `onclick`/`ondrag*` attributes in the HTML string built inside `render()`:
- `toggle(id)` — checks/unchecks today's date in `completions`
- `del(id)` — removes a habit from the array
- `startEdit(id)` — swaps `.habit-name` with an `<input>` in-place; commits on blur/Enter, cancels on Escape
- `dragStart / dragOver / drop / dragEnd` — reorder `habits[]` using the HTML5 Drag and Drop API; module-level `let dragging` holds the id being dragged

**Streak logic** (`streak(completions)`) — counts consecutive days backwards from today (or yesterday if today isn't checked yet); returns 0 if neither today nor yesterday is in the set.

**Styling** — dark theme via CSS custom properties on `:root` (`--bg`, `--surface`, `--accent`, `--success`, `--danger`, `--streak`, etc.). All colors should use these variables, not hardcoded hex.
