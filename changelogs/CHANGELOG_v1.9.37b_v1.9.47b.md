# Changelog — v1.9.37b → v1.9.47b

All changes between v1.9.37b and v1.9.47b, grouped by release.

---

## v1.9.38b — Model RAM Caching (WITHDRAWN)

- **DiT/VAE RAM caching between runs** — when `cache_dit` / `cache_vae` was
  enabled with offload set to GPU, models were moved to CPU RAM after each run
  instead of being freed, so subsequent runs skipped re-loading from disk.
- **Withdrawn:** the RAM-resident models slowed down DiT inference on the next
  run (speed regression). The user reverted to the v1.9.37b backup. None of
  these changes are present in v1.9.39b and later — development continued from
  the v1.9.37b codebase.

---

## v1.9.39b — GUI-Controlled Pipeline Behavior

- **All offload settings are now fully GUI-driven** — no hardcoded pipeline
  behavior. The GUI dropdowns for DiT / VAE / tensor offload control the
  pipeline end to end, and their values are forwarded to the CLI correctly:
  - GUI labels changed from `None` / `CPU` to **GPU / CPU** (clearer wording).
  - `_build_cli_args()` maps the GUI selection to `--dit_offload_device`,
    `--vae_offload_device`, and `--tensor_offload_device` for every run.
  - `_parse_offload_device("none")` returns `None` (GPU-resident) in the CLI.
- **Frame sync fix** — `prepend_frames` conditioning frames are now correctly
  trimmed from the exported output (postprocess receives
  `prepend_frames=args.prepend_frames`), so output frame count matches the
  input instead of being offset.
- **Single-file runner caching fix** — `runner_cache` is now used for
  single-file processing as well, not only for batch/queue processing.

---

## v1.9.40b — FFmpeg 9 GPU-Aware Export System

- **GPU-aware FFmpeg 9 encoding flags** — export encoder selects optimal flags
  per GPU vendor, detected automatically at startup via the device probe and
  passed to `build_ffmpeg_command()`:
  - **NVIDIA NVENC:** `-preset p7 -tune hq -rc vbr -cq N -spatial-aq 1
    -temporal-aq 1 -aq-strength 15 -rc-lookahead 32 -b_adapt 1`.
    10-bit output uses `-pix_fmt p010le`.
  - **Intel QSV:** `-preset veryslow -global_quality N -look_ahead 1
    -look_ahead_depth 40`.
  - **AMD AMF:** `-quality quality -rc cqp -qp_i N -preanalysis 1 -vbaq 1`.
  - **Software (libx264/libx265):** `-preset slow -crf N`.
- **AV1 hardware encoding** — `av1_nvenc`, `av1_qsv`, `av1_amf` auto-detected;
  falls back to `libsvtav1`.
- **Quality values tightened for AI-upscaled content** — H264/H265 CQ
  24/20/16 (was 28/23/18); AV1 CRF 30/24/20; VP9 CRF 33/27/21 with
  `-row-mt 1 -deadline good -cpu-used 3`.
- **H265 QuickTime compatibility** — `-tag:v hvc1` added automatically for H265
  in MOV containers.
- **Single GPU probe path** — the device probe result and the ffmpeg encoder
  cache are both passed into the export builder; the previously disconnected
  dual-probe system (device probe vs `ffmpeg -encoders` probe) is unified.

---

## v1.9.41b — System Tray + Interrupted Job Resume

- **Minimize to system tray** — pressing the window close button (X) hides the
  application to the Windows system tray; processing continues in the
  background. Full quit is available only from the tray icon's right-click
  menu ("Full Quit"), which aborts the active worker, releases threads,
  previews and the tray icon, and exits.
  - Tray icon left-click toggles the main window.
  - A notification is shown when hiding during active processing.
- **Interrupted job recovery on startup** — jobs stuck in `running` status
  (after a crash or force-quit) are automatically flipped back to `pending`
  via `recover_stale_running()` in `job_queue.py`. If pending jobs exist, a
  startup dialog offers to resume them:
  - **Yes** — the jobs are re-enqueued and started; the CLI resumes from
    completed chunks via `progress.json` + `completed_chunks` skip logic.
  - **No** — pending jobs are cleared.
- **Chunk resume surfaced to the user** — the CLI's chunk progress tracking
  already existed; the GUI now drives resume by re-invoking the same job, and
  the CLI transparently skips completed chunks.

---

## v1.9.42b — Comparison View Modes + Smooth Progress

- **Single / Split / Side-by-Side view buttons fixed** — all three comparison
  view modes now work for both video and static-image previews. Mode switching
  (`_set_preview_mode()`) pauses video playback on switch, and
  `_refresh_comparison_view()` correctly reloads static frames when the
  comparison payload contains images instead of videos.
- **Smooth progress bar** — progress no longer jumps in 25% phase steps. The
  phase update formula `((current_phase - 1) + phase_progress) / total * 100`
  produces continuous within-phase progress, so the bar advances smoothly
  across VAE encode → DiT → VAE decode → postprocess.

---

## v1.9.43b – v1.9.45b — Preview Engine Iterations (mpv experiment)

- **mpv-based preview experiment** — attempted to embed `mpv.exe` (Topaz Video
  AI style) as the playback engine: `--wid` window embedding plus named-pipe
  IPC (`gui/widgets/mpv_widget.py`), to offload decoding from the CPU-bound Qt
  widget.
- **Result: reverted.** `--wid` embedding produced a black screen inside the
  GUI's stacked widget, and `python-mpv` could not load `mpv-2.dll` (missing
  dependencies; a standalone libmpv build is required). The preview was
  reverted to the native Qt `VideoPreviewWidget`.
- **Interim fixes shipped in these builds:**
  - `Colors.TEXT_DIM` → `Colors.TEXT_MUTED` (AttributeError at runtime in
    `video_preview.py`).
  - A preview speed regression introduced during the experiment was reverted
    ("restore the previous speed").

---

## v1.9.46b — Preview Overlay Layout Fix

- **Info overlay / seek bar overlap fixed** — the video info text no longer
  collides with the bottom seek bar. Layout is now: frame counter top-left,
  video info (resolution, fps, frame count, zoom level) top-right, seek slider
  in a reserved 32px strip at the bottom.

---

## v1.9.47b — VideoPreviewWidget Cleanup + Realtime Playback

- **Duplicate play bar removed** — the widget no longer renders its own
  play/pause button and frame label; the application's external
  `PlaybackControls` (play/pause, next/prev) is the single control set. The
  widget keeps only the seek slider.
- **Realtime playback** — `paintEvent` switched from `SmoothTransformation` to
  `FastTransformation` for frame scaling during playback, eliminating preview
  lag/stutter on high-resolution outputs.
- **Video info overlay** — resolution, fps, frame count and zoom level shown
  at the top-right during playback.
- **Frame counter** — current/total frame count at the top-left.
- **Seek slider** — draggable position slider at the bottom with show/hide and
  layout logic (`_show_seek_bar()` / `_hide_seek_bar()` /
  `_layout_seek_bar()`); import list cleaned accordingly.

---

## Unreleased (source only, not yet built)

- **BT.709 color metadata on all FFmpeg outputs** — every FFmpeg command now
  tags the output `-color_primaries bt709 -color_trc bt709 -colorspace bt709`:
  the `FFMPEGVideoWriter` encoder pipe (covers default and GUI-export
  encoding), the chunk concat (`-c:v copy`), and the audio mux step. MediaInfo
  now reports BT.709 primaries / transfer / matrix on all outputs.
- **`PYTORCH_ALLOC_CONF` migration** — the deprecated `PYTORCH_CUDA_ALLOC_CONF`
  environment variable was renamed to the unified `PYTORCH_ALLOC_CONF`
  (`inference_cli.py`, `rocm_compat.py`), removing the PyTorch 2.9+ allocator
  deprecation warning from every run.
- **Startup banner removed from CLI logs** — the stale hardcoded
  `1-Click SeedVR 2.5 v.1.8b (by Naxci1)` + repository URL lines were removed
  from `debug.print_header()`; the dynamic version line remains.
- **`force_upcast` FutureWarning fixed** — the VAE upcast toggle in
  `model_loader.py` now accesses `model.config.force_upcast` instead of the
  deprecated direct attribute, silencing the diffusers FutureWarning during
  VAE materialization.
