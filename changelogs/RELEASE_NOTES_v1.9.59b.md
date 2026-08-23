# 1-Click SeedVR 2.5 — Release Notes (v1.9.59b)

Everything new since v1.9.58b.

---

## v1.9.59b — GPU-aware attention, BlockSwap safety, Split View fix

### Fixed: sageattn_3 crash on non-RTX-50 GPUs (critical)
- Selecting **sageattn_3** on anything other than a Blackwell GPU
  (RTX 50xx) crashed at runtime — the SageAttention package imports fine
  on RTX 30/40 cards, so the old import-only check passed while the kernel
  could not run.
- The attention mode is now validated against the **actual GPU
  architecture**: RTX 40 → automatic fallback to sageattn_2, AMD / Intel /
  CPU-only machines → SDPA. A second kernel-level safety net guarantees
  the fallback even if a mode slips through.

### New: attention options filtered by detected GPU
- At startup the hardware probe result now drives the settings panel:
  attention modes the detected GPU cannot run are **greyed out** (with a
  tooltip explaining why) and an invalid saved selection is moved to the
  best mode the GPU supports.
  - RTX 50xx: sdpa, flash_attn_2, sageattn_2, sageattn_3
  - RTX 30/40: sdpa, flash_attn_2, sageattn_2
  - GTX 10/20, AMD, Intel, CPU: sdpa only
  - flash_attn_3 stays disabled everywhere (the Hopper FA3 package is not
    part of the shipped Python bundle).

### Fixed: BlockSwap without an offload device
- Runs with `blocks_to_swap > 0` no longer abort with
  "dit_offload_device is invalid": the CLI now auto-sets the DiT offload
  to CPU at startup, in Auto Tune, in every OOM retry step, and rebuilds
  a cached context when swap is enabled mid-run.
- In the GUI, enabling **blocks_to_swap** or **swap_io_components**
  automatically switches the DiT / VAE / tensor offloads to CPU.

### Fixed: Split View "does nothing"
- Split View now opens a **synchronized dual-video comparison** for video
  outputs (previously it dead-ended with a tiny corner toast).
- When there is nothing to compare yet, a persistent status-bar message
  and a clear warning are shown instead of silence.

### Improved: failure headlines
- The error dialog headline no longer shows the generic
  "CUDA kernel errors might be asynchronously reported…" boilerplate —
  it now surfaces the real error line.

### Other
- **Tile debug** row is labelled "(TEST)" with a warning note underneath
  so it is not enabled accidentally.
- UI language is fully English again — leftover localized tooltips were
  translated and the whole codebase scanned clean.
