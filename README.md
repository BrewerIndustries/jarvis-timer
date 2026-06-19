# Jarvis Timer

A focus timer for time-boxing. Tell it **how much time you have** and **how many
things you want to do**, and it splits the time into segments and counts each one
down on a big on-screen clock — with an optional real-time wall clock in the corner.

Part of the [Jarvis constellation](https://jarvis.dabrewer.dev/). A self-contained
"standalone" app — single-file, no build step.

- **Prod:** https://jarvis-timer.dabrewer.dev/
- **Dev:** https://jarvis-timer.dabrewer.dev/dev/

## Concept

> "I have **50 minutes** and want to do **3 things**."

You name the things (optional) and tweak the per-task minutes and seconds if you don't
want an even split. Hit start and it walks you through each task, chiming and
auto-advancing when a segment ends.

## Features (v0.1)

- **Time-box setup** — "I have *N* minutes and want to do *M* things"; quick presets
  (15/25/30 min, 1h, 90m, 2h) and even-distribution across tasks, computed down to the
  second (e.g. 50 min / 3 → 16:40 each).
- **Per-task editing** — name each thing, override its minutes **and seconds**,
  add/remove rows.
- **Segment countdown** — big circular dial for the current task, plus total time
  remaining and a projected finish time.
- **Segment strip** — see every task's progress at a glance; click to jump.
- **Controls** — pause/resume, +1 min, restart task, skip to next, stop.
- **Chimes** — a tone when each segment ends and a flourish when you finish
  (WebAudio, no assets).
- **Real-time wall clock** — shown top-right, **toggleable** (button or `C`); the
  preference is remembered.
- **Drift-free** — counts down off timestamps, not naive interval decrements.
- Remembers your last plan and the clock toggle in `localStorage`.

### Keyboard

| Key | Action |
| --- | --- |
| `Space` | Pause / resume |
| `N` | Next task |
| `R` | Restart current task |
| `C` | Toggle the wall clock |

## Tech

- Single-file `index.html` — vanilla JS, no dependencies, no build step.
- Open `index.html` directly, or serve it: `python3 -m http.server 3000`.
- Deployed via **GitHub Pages** (see below).

## Deploy (CI/CD)

One Pages site serves both branches, published by
[`.github/workflows/pages.yml`](.github/workflows/pages.yml):

| Branch | Path | URL |
| --- | --- | --- |
| `main` | `/` | https://jarvis-timer.dabrewer.dev/ |
| `dev` | `/dev/` | https://jarvis-timer.dabrewer.dev/dev/ |

The workflow lives on `dev` and checks out both branches itself. **Push to `dev`**
(or use *Run workflow*) to redeploy; pushing to `main` does not auto-deploy. Work on
`dev`; promote to `main` via a reviewed PR.

The `/dev/` build shows a **DEV** badge in the header.

## Registry

This repo declares itself to the Jarvis launcher + dashboard via
[`.jarvis.json`](.jarvis.json) (category `standalone`). The dashboard's
`sync-registry` job reads it per environment (dev branch → dev, main → prod).

## Backlog

- Per-task colors / icons on the dial and strip.
- Optional break segments between tasks.
- Desktop notifications (in addition to the chime) when a segment ends.
- Save/name multiple plans; share a plan via URL.
- Overrun mode — keep counting up past 00:00 instead of auto-advancing.
- Sound on/off toggle and volume.
