# 1-Click SeedVR 2.5 — Release Notes (v1.9.55b)

Everything new since v1.9.54b.

---

## v1.9.55b — Pre-downscale crash fix, filter selection, smarter Auto Tune

### Fixed: pre-downscale crash (critical)
- Every run with **Pre-downscale enabled crashed** with
  `ValueError: Anti-alias option is only supported for bilinear and bicubic
  modes` — the old resize call combined `mode="lanczos"` with
  `antialias=True`, a combination PyTorch does not support at all.
- The downscaler now uses **antialiased bicubic with a [0, 1] clamp** —
  PyTorch's highest-quality downscale path (matches PIL's antialiased
  resize), fully float32, no uint8 quantization anywhere.

### New: Downscale filter selection
- A new **"Downscale filter" dropdown** appears in the settings panel
  whenever Pre-downscale is enabled. Eight resize kernels to choose from:
  - **Bicubic + Anti-alias (best)** — default
  - Bicubic
  - Bilinear + Anti-alias / Bilinear
  - Area (average) / Nearest
  - **Lanczos** and **Mitchell cubic** (via OpenCV, float32 CPU — torch
    has no Lanczos kernel)
- New CLI argument `--downscale_filter`; the choice persists in settings
  and presets.
- Robust by design: an unknown or stale value can never crash a run — it
  silently falls back to the safe default (verified by automated tests).
- All kernels were self-tested on GPU/CPU for correct shape, float32 dtype,
  [0, 1] range and NaN-free output at factors 2× and 3×, plus an offscreen
  GUI test (default value, selection, preset restore, visibility toggle).

### Auto Tune is now resolution-aware
- Starting parameters are computed from the **actual target output
  dimensions** (VAE working-set estimate from measured decode/encode peak
  memory), instead of a fixed VRAM-tier table that ignored resolution.
- Example: a 12 GB card processing 1080p output now starts at
  batch=81 / tile=384 / swap=16 (~7.2 GB working set) instead of tile=512,
  which silently spilled into shared memory (WDDM spills raise no OOM
  error, so the retry chain never fired).
- Tiling now engages whenever the tile size is smaller than the output
  width or height (previously only below a fixed 1024 px, which skipped
  tiling for wide frames).
- The log line now reads
  `Auto Tune: free VRAM X GB, target W×H → batch=… tile=… swap=…`.

### Other
- Executable is now built as **`1Click_SeedVR_2.5.exe`** (InternalName and
  OriginalFilename metadata updated to match).
