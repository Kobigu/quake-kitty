# quake-kitty

Custom fork of [kitty](https://github.com/kovidgoyal/kitty) (v0.49.1) that
makes kitty's quick-access terminal a **real quake-style drop-down terminal on
the COSMIC desktop** (cosmic-comp).

Branch: **`quake`** (based on tag `v0.49.1`). All changes are marked
`quake fork` in the source.

## Why

kitty's upstream quick-access terminal (`kitten quick-access-terminal`, a
wlr-layer-shell surface) is broken on cosmic-comp in two ways:

1. **It can never be re-shown.** Hiding uses a null-buffer unmap; cosmic-comp
   (like niri) never sends the fresh `configure` event the spec requires for a
   remapped layer surface, so kitty waits forever and the window never
   reappears.
2. **Keyboard focus can't be released while hidden.** cosmic-comp applies
   layer-surface keyboard interactivity only at map time and ignores runtime
   `set_keyboard_interactivity` changes, so a hidden-but-mapped surface keeps
   swallowing every keystroke.

## What the fork does

- **Hide**: slides the surface out past its docked edge, then destroys the
  layer surface (unmaps + releases the keyboard everywhere).
- **Show**: destroys and re-creates the whole `wl_surface` — viewport,
  fractional-scale, EGL window and EGLSurface included (the GL context and all
  GPU state survive) — and creates a fresh layer surface. Fresh surface = fresh
  configure + fresh keyboard interactivity on every compositor.
- **Guard**: `commit_window_surface()` never commits a surface whose
  layer-surface role was destroyed (a protocol error → client disconnect on
  cosmic-comp).
- **`slide_duration`** (default 150ms): slide in/out by animating the
  layer-surface margin (ease-out), direction follows the docked edge.
- **`lines 50%` / `columns 50%`**: sizes as a percentage of the monitor,
  resolved at layer-size calculation time (any scale/resolution).
- **`listen_on`** option for the quick-access kitten (stable remote-control
  socket, no per-PID suffix).
- Fix for an upstream bug in the quick-access Go wrapper (`--detached-log`
  passed as a bare positional).
- Version string says `(quake fork)`.

The integration (F12 binding, autostart, configs) lives in the parent
`quake-terminal` repo.

## Rebasing onto a new kitty release

```bash
git remote add upstream https://github.com/kovidgoyal/kitty.git  # if needed
git fetch upstream --tags
git checkout quake && git rebase v0.4x.y
# resolve conflicts — all fork changes are tagged "quake fork"
```
