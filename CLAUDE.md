# Jarvis Timer

A focus timer for time-boxing. You say how much time you have and how many things
you want to do; it splits the total across the tasks (even distribution down to the
second, or per-task overrides) and counts each segment down on a big circular dial,
with an optional real-time wall clock in the top-right. WebAudio chimes mark each
segment end and the finish. Part of the Jarvis constellation (category `standalone`).

- **Prod:** https://jarvis-timer.dabrewer.dev/
- **Dev:** https://jarvis-timer.dabrewer.dev/dev/

## Tech

- Single-file `index.html` — vanilla JS (no framework, no CDN, no dependencies),
  inline `<style>` and `<script>`. No build step.
- Web Audio API for chimes (synthesized, no audio assets).
- `localStorage` for the saved plan and the clock toggle.

## Code organization (all inside `index.html`)

HTML is three `.view` sections toggled by `show(view)`: `setupView`, `runView`,
`doneView`. All logic is one IIFE at the bottom (`<script>`, ~line 460+).

- **Helpers** — `$`, `clamp`, `pad`, `LS` (localStorage get/set), `fmt(ms)`
  (→ `M:SS`/`MM:SS`/`H:MM:SS`), `humanMins`.
- **DEV badge** — shown when the path ends in `/dev` (line ~489).
- **Wall clock** — `tickWallClock()` on a 1s `setInterval`; `wallClockOn` persisted
  as `jt.wallclock`; `applyWallToggle()` shows/hides.
- **Sound** — lazy `AudioContext` (`audio()`, unlocked on the Start gesture);
  `beep()` builds tones; `chimeSegment()` / `chimeFinish()`.
- **Setup / segment math** — `tasks` array of `{name, mins, secs}`. `distribute()`
  splits `poolMins()*60` seconds evenly, giving the remainder to the first rows
  (so 50 min / 3 → 16:40, 16:40, 16:40). `setCount(n)` grows/shrinks the list;
  `renderTasks()` rebuilds rows and wires per-field inputs; `refreshAlloc()` shows
  planned total. Last plan restored from `jt.config`.
- **Run engine** — `RUN` state object (`segs`, `i`, `remaining`, `running`, `endAt`,
  `ticker`). **Drift-free:** a 200ms `setInterval` (`onTick`) computes `remaining`
  from `endAt - performance.now()`, not by decrementing. `advance()` moves to the
  next segment (or `finish()`); `pauseToggle`, `skip`, `restartSeg`, `addMinute`
  (+1 min), `stopRun`, `jumpTo` (click a strip segment). `startRun()` filters to
  tasks with time > 0 and persists `jt.config`.
- **Run rendering** — `renderRun()` updates the dial (SVG `stroke-dashoffset` off
  `CIRC = 2πr`, r=135), stage/name, total remaining + projected finish ETA, the
  segment strip, pause button, and the tab title.
- **Keyboard** — `Space` pause/resume, `N` next, `R` restart task, `C` toggle clock
  (ignored while typing in an input).

## Run locally

Open `index.html` directly, or serve it: `python3 -m http.server 3000`. No build.

## Deploy (GitHub Pages)

One Pages site serves both branches via `.github/workflows/pages.yml`, which lives
on `dev` and checks out both branches itself:

| Branch | Path | URL |
| --- | --- | --- |
| `main` | `/` | https://jarvis-timer.dabrewer.dev/ |
| `dev` | `/dev/` | https://jarvis-timer.dabrewer.dev/dev/ |

The workflow triggers on **push to `dev`** (or manual *Run workflow*). Pushing to
`main` does **not** auto-redeploy — merging a PR to `main` still requires a `dev`
push or manual dispatch to publish. Workflow sets the `CNAME` and strips `.git`/
`.github` from both copies.

## Git workflow

- Work on `dev`. Push to `dev` freely.
- Promote to `main` ONLY via a user-approved PR. Never fast-forward or reset-push
  to `main`.
- Keep `README.md` updated after any meaningful change.

## Gotchas

- Editing `main` alone won't redeploy prod — the workflow only runs off `dev`.
- Per-task seconds are clamped 0–59; minutes 0–1440; task count 1–20; total minutes
  clamped to 1440 (24h).
- The DEV badge and `/dev/` behavior key off the URL path ending in `/dev`, so it
  only appears on the deployed dev build, not local file:// or a bare static serve.
- `jt.config` (saved plan) and `jt.wallclock` (clock toggle) are the only
  `localStorage` keys.
