# 1-Click SeedVR 2.5 — Release Notes (v1.9.63b)

Everything new since v1.9.62b.

---

## v1.9.63b — 7B download crash root-caused & fixed, never-die errors, split-line anywhere, model deletion

### Fixed: 7B model downloads crashed the app (critical, root cause found)
- PySide6 marshals ``Signal(int)`` arguments to a **32-bit C++ integer**.
  The 7B model sizes (4.8–16.5 GB in bytes) exceed the 32-bit limit, so
  every progress update raised ``OverflowError`` → "Unhandled Exception"
  dialog → freeze → app closed. This is why small models (≤2 GB) downloaded
  fine while every 7B download died.
- The progress signal now transmits **megabytes** (comfortably inside the
  32-bit range). Verified live against HuggingFace: partial download of
  the exact failing model (7B Q4) runs clean with zero exceptions, and
  cancel/resume still work. All 10 catalog links re-verified (200 OK,
  sizes match).

### New: "never die" error policy (Topaz-style)
- The error dialog is now **non-modal**: an unhandled exception no longer
  enters a nested event loop (that re-entrancy froze and killed the app
  right after showing the dialog). The application keeps running so the
  user can read the report and press Send Logs.
- Only **one** error dialog at a time — cascading exceptions during a
  crash no longer spawn a dialog storm.
- Hard C++ crashes (access violations) are captured by ``faulthandler``
  and appended to **error.log** (no separate crash.log; Send Logs ships
  them automatically).

### New: Split line drags from anywhere (ComfyUI Image-Comparer style)
- **LMB press anywhere on the image grabs the split line** — no precise
  aiming, the line jumps to the cursor and follows the drag at any zoom
  level. Video play/pause stays on the control button and the Space key.
  RMB pans while zoomed; double-click still recenters.

### New: Delete button in the model manager
- Downloaded models can be checked and **deleted** (with the model's
  stale ``.download`` partial cleaned up too). A confirmation dialog
  lists the names and sizes and shows the disk space to be freed before
  anything is removed; the DiT list refreshes afterwards.

### Improved: download safety
- **Disk-space pre-check**: if the selection needs more space than the
  drive has free, downloading refuses to start with a clear
  "Needed X GB / Free Y GB" message.
- Model-manager signal handlers are guarded against the dialog being
  closed mid-download (late signals can no longer crash anything), and
  the window was narrowed.

### Verified (highlights)
- Live network tests: 7B Q4 partial download + cancel + Range-resume.
- Real Qt mouse events: split-line grab-anywhere at 3× zoom.
- Crash simulation: two consecutive slot exceptions → one dialog, app
  alive, both tracebacks in error.log.
- Delete flows: confirm/decline/partial-cleanup/empty-selection.
