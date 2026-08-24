# 1-Click SeedVR 2.5 — Release Notes (v1.9.61b)

Everything new since v1.9.60b.

---

## v1.9.61b — First-launch path bootstrap, safer error reporting

### New: one-shot path bootstrap on first launch
- The shipped config.json carries a hidden `"first_run": "true"` flag with
  the build machine's default paths (`C:\1Click_SeedVR2.5\...`).
- On the **first launch** the app derives the real install location from
  the executable's own path (e.g. `G:\seedvr\seedvr2_videoupscaler\dist\…`
  → shared root `G:\seedvr\`) and **rewrites every path** (python, ffmpeg,
  models, seedvr2 folder, temp, log, presets) to that root, saves
  config.json and flips the flag to `"false"` — the rebase runs exactly
  once and is never repeated.
- Paths that still carry the shipped default root, or no longer exist,
  are rebased; intentional custom locations that resolve are untouched.
- A safety net remains for copies made **without** the flag: a config
  whose `seedvr2_folder` names a different install is re-anchored the
  same way.

### config.json now uses forward slashes
- Paths are stored as `C:/1Click_SeedVR2.5/...`: Windows accepts forward
  slashes everywhere, JSON needs no escaping for them, and hand edits in
  Notepad can no longer break the file (a single `\` in JSON is an escape
  sequence — `\\` was always correct, but `/` removes the confusion).

### Fixed: "last used folder" forgotten after restart
- Two independent writers were overwriting each other's value; the key is
  now owned by a single writer (the main window), so the last import
  folder survives restarts.

### Improved: error reporting for thread crashes
- Unhandled exceptions in worker/QThread threads previously bypassed the
  app's exception hook and vanished. A `threading.excepthook` now funnels
  them into error.log with **full tracebacks** (and therefore into Send
  Logs mails) — this also captures the reported `OverflowError` class of
  failures with real context next time it happens.
- The progress-line parser is hardened against `OverflowError` from
  malformed numeric output.

### Fixed: manual batch size was overwritten by Auto Tune
- With Auto Tune enabled, a GUI-configured batch size (e.g. 21) was
  silently replaced by the computed default (81). The user's setting is
  now authoritative: Auto Tune only caps it when the VRAM estimate says
  it cannot fit, and the log states what happened
  (`batch=21 (user setting kept: 21)` /
  `user 999 capped to 81 by VRAM estimate`).
- The last-resort OOM fallback no longer RAISES a smaller user batch to
  45 — the floor only limits automatic reduction.
- Note: "N-frame sequence" log lines use the temporal expansion
  (batch−1)×4+1 — a batch of 21 legitimately processes 81 frames; that
  is not the batch size.

