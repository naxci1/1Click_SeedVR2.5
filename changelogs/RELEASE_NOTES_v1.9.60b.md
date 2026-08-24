# 1-Click SeedVR 2.5 — Release Notes (v1.9.60b)

Everything new since v1.9.59b. This is a large release: a new Model Manager,
a full Split View rework, GPU-aware attention lists, non-system-drive install
fixes, and a critical error-reporting repair.

---

## v1.9.60b — Model Manager, Split View rework, error reporting

### New: AI Models manager (top-bar "Models" button)
- A Topaz-style model manager window with three groups: **GGUF Models,
  3B Models, 7B Models**.
- Every model shows its **file size** (fixed values captured once from the
  download links — the dialog never queries the network).
- Models already on disk show a green **OK** and cannot be re-selected.
- Select any combination and press **Download**: sequential downloads with
  a progress bar, **Cancel** aborts the current transfer (the partial
  `.download` file is kept, so a later attempt **resumes** where it stopped),
  **Close** finishes.
- The DiT list in settings refreshes automatically after downloads;
  downloaded models are **bold** there, missing ones carry a
  "downloads on first use" tooltip.
- The non-existent "7B Q8" entry was removed (no download source exists).

### Fixed: the error dialog never opened on processing failures (critical)
- The failure dialog was called with an invalid argument list, the
  TypeError was silently swallowed and only a small toast appeared — which
  is why **Send Logs was never visible and no reports arrived by mail**.
- The dialog now opens with the full report (command + last 100 log lines)
  auto-expanded, plus Copy to Clipboard and Send Logs buttons.

### Improved: Send Logs mail
- The mail body now contains the **full error text** (LAST ERROR section)
  and the subject carries the error headline.
- The archive additionally contains the three most recent
  `cli_failure_*.log` reports and a `last_error.txt`.
- The Send Logs button on unhandled-exception dialogs is wired too, so
  every error path can report by mail.

### Attention modes are now filtered by GPU
- Unsupported modes are **removed from the list** (not greyed out): every
  visible option is selectable.
  - RTX 50xx: sdpa, flash_attn_2, sageattn_2, sageattn_3
  - RTX 30/40: sdpa, flash_attn_2, sageattn_2
  - GTX 10/20 / AMD / Intel / CPU: sdpa only
- The saved selection moves to the best supported mode automatically.

### Split View rework (tester feedback)
- **LMB drags the split divider at any zoom level**, **RMB pans**, and
  **double-click recenters** (zoom preserved) — in normal and fullscreen
  split view alike (ComfyUI Image-Comparer-style behaviour).
- **Video outputs** open a synchronized dual-video comparison instead of
  dead-ending.
- The comparison is **remembered per file**: switching to another file and
  back restores that file's preview.
- The **timeline seeks the split view** in video mode; hovering frames no
  longer looks broken.
- The **frame position is remembered** when switching between Split and
  Single view.
- Fullscreen **can be reopened repeatedly** (stale-window bug fixed) and
  **ESC** exits fullscreen.

### Fixed: installs on non-system drives (G:\ etc.)
- Default asset locations (python_embeded / ffmpeg / models) now probe the
  shared root **one level above the project folder** first — matching the
  shipped layout — instead of only looking inside the project folder.
- The bundled **ffmpeg** is found in both layouts (`ffmpeg\ffmpeg.exe` and
  `ffmpeg\bin\ffmpeg.exe`) and is **always preferred**; the codec probe no
  longer silently falls back to a system-wide ffmpeg.
- A new `--ffmpeg_path` argument hands the CLI the exact binary so the
  writer, concat and audio mux never depend on the system PATH.
- Hand-edited config.json damage is handled: invalid JSON escapes
  (`G:\1Click…`) keep a `config.json.bak` instead of silently discarding
  **all** settings; valid escapes that embed control characters
  (`\f`, `\b`, `\t`) are repaired to forward-slash paths.
- A failed config save (read-only folder) now shows a clear warning
  instead of silently losing the settings.

### Fixed: Auto Tune target resolution
- Auto Tune logged and sized its parameters from the raw `--resolution`
  (e.g. "target 2160×1080" for a 1920×960 source at 1:3 + 3×). It now uses
  the same effective-resolution math as the pipeline (→ 1920×960), so tile
  choices match the real workload.

### Other
- New **1:4 pre-downscale ratio** (for 4K–8K sources).
- File dialogs (Import / Browse / folder select) **remember the last used
  directory** across restarts (`last_import_dir` in config.json).
- A transient `CUDA error: device not ready` in the downscaler is retried
  once after a synchronize before failing.
- Down: Stop/abort during model downloads keeps the resumable partial.
