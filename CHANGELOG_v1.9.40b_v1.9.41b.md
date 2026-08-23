## v1.9.40b — FFmpeg 9 GPU-Aware Export System

- **GPU-aware FFmpeg 9 encoding flags** — Export encoder now uses optimal
  flags per GPU vendor, detected automatically at startup via the device probe
  (`self._gpu_backend`) and passed directly to `build_ffmpeg_command()`:
  - **NVIDIA NVENC:** `-preset p7 -tune hq -rc vbr -cq N -spatial-aq 1
    -temporal-aq 1 -aq-strength 15 -rc-lookahead 32 -b_adapt 1` (was `-preset p5`).
    10-bit: `-pix_fmt p010le` (was `yuv420p10le`).
  - **Intel QSV:** `-preset veryslow -global_quality N -look_ahead 1
    -look_ahead_depth 40` (was no preset, minimal flags).
  - **AMD AMF:** `-quality quality -rc cqp -qp_i N -preanalysis 1 -vbaq 1`
    (was `-quality balanced -qp_i N`).
  - **Software (libx264/libx265):** `-preset slow -crf N` (was `-preset medium`).
- **AV1 hardware encoding** — `av1_nvenc` (NVIDIA), `av1_qsv` (Intel), and
  `av1_amf` (AMD) auto-detected and selected when available, falling back to
  `libsvtav1`. AV1 NVENC uses `-preset p7 -tune hq -rc vbr -cq N -spatial-aq 1
  -rc-lookahead 32`.
- **Quality values tightened for AI-upscaled content:**
  - H264/H265: CQ/QP 24/20/16 (was 28/23/18).
  - Software CRF: 22/18/14 (was 28/23/18).
  - AV1: CRF 30/24/20 (was 35/25/18).
  - VP9: CRF 33/27/21 + `-row-mt 1 -deadline good -cpu-used 3`.
- **H265 QuickTime compatibility** — `-tag:v hvc1` automatically added when
  exporting H265 to MOV container.
- **GPU info passed to export** — `app.py:_build_export_ffmpeg_args()` now
  passes `self._gpu_backend` and `self._codec_cache` to the encoder builder,
  eliminating the disconnected dual-probe system (device probe vs ffmpeg
  `-encoders` probe were separate and never communicated).

---

## v1.9.41b — System Tray + Interrupted Job Resume

- **System tray icon** — Application now minimizes to the Windows system tray
  instead of terminating when the window close button (X) is pressed during
  active processing:
  - **Close button during processing:** Window hides, processing continues in
    background. Tray notification: *"İşlem devam ediyor. Tam kapatmak için
    tepsi ikonuna sağ tıklayın."*
  - **Close button when idle:** Normal exit.
  - **Tray icon left-click:** Toggle window show/hide.
  - **Tray icon right-click menu:** "Göster" (Show) and "Tam Kapat" (Full Quit).
  - **Tam Kapat:** Aborts the active worker, releases all resources (threads,
    previews, tray icon), and quits the application.
- **Interrupted job recovery on startup** — When the application is relaunched
  after a crash or force-quit:
  - Jobs stuck in `"running"` status are automatically flipped back to
    `"pending"` via `recover_stale_running()` in `job_queue.py`.
  - If pending jobs exist, a startup dialog asks: *"Yarım kalmış iş bulundu.
    Devam etmek istiyor musunuz?"*
    - **Yes:** Re-enqueues and starts the pending job(s). The CLI automatically
      resumes from completed chunks via `progress.json` + `completed_chunks`
      skip logic — partially rendered chunks are re-rendered, completed chunks
      are skipped.
    - **No:** All pending jobs are cleared (`clear_pending()`).
- **Chunk resume (existing, now surfaced)** — The CLI's chunk progress tracking
  (`_load_chunk_progress` / `_write_chunk_progress` in `inference_cli.py`)
  was already functional but invisible to the user. Now the GUI drives the
  resume by re-invoking the same job, and the CLI transparently skips
  completed chunks.
