# 1-Click SeedVR 2.5 — Release Notes (v1.9.62b)

Everything new since v1.9.61b.

---

## v1.9.62b — Crash-proof downloads, self-diagnosing config

### Fixed: model download could crash the whole app
- The Model Manager download worker is now fully crash-guarded: any
  unexpected exception (including the reported traceback-less
  `OverflowError` from inside Qt/C++ bindings) is converted into a clear
  status message on the dialog instead of taking the application down.
- The progress percentage is clamped to 0–100 with divide-by-zero
  protection.
- Unhandled exceptions without a Python traceback are now logged with
  their type/repr and a note pointing at the operation context, so the
  real cause shows up in error.log / Send Logs mails.

### New: every log line identifies the build
- Startup writes `MainWindow ready — 1-Click SeedVR 2.5 v1.9.62b` into
  app.log — sent-in logs can always be matched to the exact version.

### New: config write self-test at startup
- Right after load, the app verifies config.json is actually writable.
  On failure (read-only folder, antivirus/lock) it logs
  `config.json WRITE TEST FAILED` to error.log and shows an on-screen
  warning — the "settings silently don't stick" class of problems is now
  immediately visible on the affected machine.

### Improved: timeline behaviour in Split view (tester request)
- Clicking/scrubbing the timeline while in Split view now switches back
  to Single view and seeks to the chosen frame — frame-accurate
  scrubbing without manual mode switching.

### Improved: import dialog start folder
- When no last-used folder is known (fresh install or unwritable config),
  dialogs open in the user's home folder instead of the EXE/dist folder.

### Verified
- The frozen EXE was tested end-to-end in a simulated fresh install on a
  foreign path with a shipped (C:\-pointing, `first_run: true`) config:
  all paths were rebased to the real install root, the flag flipped to
  `false` and config.json was persisted on first launch.
