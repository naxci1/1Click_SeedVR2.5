# 1‑Click SeedVR2.5 — GUI Reference Guide

> **Version 1.7 Beta** · A high‑performance AI video restoration GUI built on the ByteDance **SeedVR2** architecture.

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![GitHub](https://img.shields.io/badge/GitHub-naxci1%2F1Click__SeedVR2.5-black)](https://github.com/naxci1/1Click_SeedVR2.5)

> [!NOTE]
> This document covers the **graphical interface only**. It is a complete, control‑by‑control reference for the desktop app — every checkbox, field, dropdown and button is documented below. Command‑line usage, installation and environment setup are intentionally out of scope here.

---

## 1. Project Overview & Key Features

**1‑Click SeedVR2.5** is a high‑performance AI video‑restoration GUI optimized for the ByteDance **SeedVR2** diffusion‑transformer (DiT) architecture. It wraps the heavyweight upscaling engine in a fully visual workflow so that restoring and upscaling footage — from a single still frame to multi‑hour 20th‑century film archives — is driven entirely through panels, dropdowns and a live comparison viewer.

### Key Features

| Feature | What it gives you |
|---------|-------------------|
| **Portable `python_embeded` integration** | The GUI launches the upscaling engine through a configurable Python executable (the bundled portable `python_embeded`, or any interpreter you point it at). No global Python install is assumed; paths are resolved through the **Settings** window. |
| **Multi‑stage pipeline** | A staged DiT → VAE‑decode → encode pipeline with automatic fallbacks (tile throttling, batch step‑down, dynamic‑shape clamps) so heavy jobs degrade gracefully instead of crashing. |
| **Native preview workflow** | A one‑click **Preview** captures a single frame, upscales it, and drops you straight into a live split‑screen original‑vs‑result comparison — no full render required. |
| **Simple & Advanced modes** | A clean default surface for quick jobs, with an Advanced toggle that reveals memory, performance, caching and debugging controls. |
| **Container‑aware export** | A two‑step Container → Codec workflow plus a full image‑sequence pipeline (TIFF/PNG/DPX/EXR) for archival masters. |
| **Broad hardware support** | NVIDIA CUDA, AMD ROCm, Intel XPU and CPU fallback, with single‑ and multi‑GPU inference. |

---

## 2. Complete GUI Settings Directory

This section walks through **every** section of the interface and documents **every individual control** — its purpose, its effect on upscale quality, and its impact on performance/VRAM.

> [!NOTE]
> Controls marked **[Advanced]** are hidden in Simple Mode and only appear once you toggle **Advanced Mode** in the header bar.

### 2.0 Header Bar

| Control | Type | Purpose |
|---------|------|---------|
| **About** | Button | Opens the About dialog (version / credits). |
| **GitHub** | Button | Opens the project page in your browser. |
| **Simple / Advanced Mode** | Toggle | Switches between the streamlined surface and the full control set. Advanced reveals BlockSwap memory, Performance, Quality Control, Model Cache and Debug groups. |
| **Settings** | Button | Opens **Paths & Configuration** (Python executable, script folder, FFmpeg, models directory). |

---

### 2.1 File & Path Selection

File and folder paths are managed through the **📁 Folders** dialog and global drag‑and‑drop.

| Control | Type | Behavior |
|---------|------|----------|
| **Input Mode** | Dropdown | `File` → process a single video/image. `Folder` → batch‑process every supported media file in a directory. |
| **Input Path** | Text field + Browse | Path to a video, image, or directory. Supports **drag‑and‑drop anywhere on the window** — a *"Drop supported media anywhere to import"* overlay appears while dragging, and dropping a file immediately loads it into the viewer. |
| **Output Path** | Text field + Browse | **Optional.** Leave blank to auto‑name the result directly beside the input file. |

**Drag‑and‑drop behavior.** Dropping a supported video/image onto the window loads it into the input player and refreshes the metadata label. Supported input video types include `.mp4 .mov .mkv .avi .webm .mpeg .mpg .m4v .wmv .flv .mts .m2ts`; supported image types include `.png .tif .tiff .jpg .jpeg .dpx .exr`.

**Output directory behavior.** When the Output Path is empty, results are written next to the source file using the naming convention `seedvr2_<input_stem>.<ext>`. When a folder is supplied, results are written into it. In batch (Folder) mode, results land in a sibling `<foldername>_upscaled` directory.

> [!TIP]
> Leave the **Output Path** blank to auto‑name the result next to the input file. This is the recommended workflow because it keeps masters and sources together and avoids stray subfolders.

---

### 2.2 Core Model Configuration

Found in the **AI Model** and **Processing Settings** group boxes on the *Adjustments* tab.

#### DiT Model Selection

A single dropdown selects the diffusion‑transformer weights. Selecting a DiT model **auto‑selects the compatible VAE** behind the scenes.

| Model file | Size | Precision / Notes |
|------------|------|-------------------|
| `seedvr2_ema_3b-Q4_K_M.gguf` | 3B | 4‑bit GGUF quant — smallest VRAM footprint, fastest, lowest fidelity. |
| `seedvr2_ema_3b-Q8_0.gguf` | 3B | **8‑bit GGUF quant — default.** Best balance of quality, speed and VRAM for the 3B family. |
| `seedvr2_ema_3b_fp8_e4m3fn.safetensors` | 3B | FP8 — near‑FP16 quality at reduced VRAM on FP8‑capable GPUs. |
| `seedvr2_ema_3b_fp16.safetensors` | 3B | Full FP16 — highest 3B fidelity, highest 3B VRAM cost. |
| `seedvr2_ema_7b-Q4_K_M.gguf` | 7B | 4‑bit GGUF quant of the larger model — more detail than 3B at moderate VRAM. |
| `seedvr2_ema_7b_fp8_e4m3fn_mixed_block35_fp16.safetensors` | 7B | Mixed FP8/FP16 — high 7B quality with reduced VRAM. |
| `seedvr2_ema_7b_fp16.safetensors` | 7B | Full FP16 7B — top‑tier fidelity, heaviest VRAM. |
| `seedvr2_ema_7b_sharp-Q4_K_M.gguf` | 7B "sharp" | Sharpening‑tuned 4‑bit variant. |
| `seedvr2_ema_7b_sharp_fp8_e4m3fn_mixed_block35_fp16.safetensors` | 7B "sharp" | Sharpening‑tuned mixed FP8/FP16. |
| `seedvr2_ema_7b_sharp_fp16.safetensors` | 7B "sharp" | Sharpening‑tuned full FP16. |

> [!NOTE]
> **Precision trade‑offs.** Lower bit‑widths (Q4 → Q8 → FP8 → FP16) progressively raise both quality and VRAM/compute cost. The **3B** family is the practical choice for consumer GPUs; the **7B** family yields the highest detail but demands substantially more VRAM. The "sharp" variants bias toward edge/detail emphasis.

#### VAE Model Selection

The VAE decoder (e.g. `ema_vae_fp16.safetensors`) turns the model's latent output back into pixels. It is **auto‑selected to match the chosen DiT model**, so there is normally no manual VAE picker to manage — the GUI keeps the pairing consistent for you. The VAE decode phase is the single most VRAM‑intensive stage of the pipeline, which is why it has its own tiling and tile‑clamp safeguards (see §2.4).

#### Resolution

Resolution is governed by **Pre‑Downscale**, **Resolution Mode**, **Resolution**, and **Max Resolution**.

| Control | Type | Options / Range | Effect |
|---------|------|-----------------|--------|
| **Pre‑Downscale** | Dropdown | `1:1` (passthrough), `2:1` (halve via Lanczos), `3:1` (reduce to ⅓ via Lanczos) | Shrinks the input *before* upscaling. Lowering the working resolution dramatically cuts VRAM and time, and can clean up noisy sources by giving the model a smaller, denser base to rebuild from. |
| **Resolution Mode** | Dropdown | `Pixel`, `X Times`, `Standard` | Chooses how the target is expressed. |
| **Resolution** | Spin / Dropdown (mode‑dependent) | Pixel: target short‑side in px · X Times: `1x`–`5x` multiplier · Standard: `480`, `720 (HD)`, `1080 (FHD)`, `1440 (2K)`, `2160 (4K)` | The actual target. |
| **Max Resolution** | Spin | Int, `0` = no limit | Hard ceiling on the output short side, regardless of mode. |

**Short‑side scaling mechanics.** Targets are applied to the **short side** of the frame; the long side scales proportionally, so the **aspect ratio is always preserved** (aspect‑ratio protection). In `X Times` mode, the multiplier is applied to the (post‑pre‑downscale) input short side. Higher targets sharply increase VRAM use during VAE decode — this is the primary lever behind out‑of‑memory risk.

#### Seed Configuration

SeedVR2 generation is conditioned on a seed. The GUI runs with a **fixed, deterministic seed (313)**, which means repeated runs of the same source with the same settings produce **identical, reproducible output** — important for archive work where consistency across re‑renders matters. (Deterministic generation eliminates the run‑to‑run variation you'd get from a random seed.)

---

### 2.3 Export & Encoding Settings

Found on the **Codec Settings** tab. An **Output Type** selector at the top switches the whole tab between **Video Export** and **Image Sequence Export**.

#### Container Formats & Output Formats

| Control | Type | Options |
|---------|------|---------|
| **Output Type** | Dropdown | `Video` or `Image Seq.` — toggles the entire export pipeline. |
| **Container** | Dropdown | `MP4`, `MOV`, `MKV`, `WEBM`. Filters the Video Codec list to compatible codecs only. |
| **Image Sequence Format** | Dropdown | `.png`, `.tif`, `.tiff`, `.jpg`, `.jpeg`, `.dpx`, `.exr` (when Output Type = Image Seq.). |
| **Image Bit Depth** | Dropdown | `8‑bit`, `10‑bit`, `12‑bit`, `16‑bit`, `32‑bit`. |

> [!NOTE]
> **TIFF / PNG for masters.** For archival masters, an image sequence in 16‑bit **TIFF** (uncompressed) or **PNG**/**DPX**/**EXR** avoids inter‑frame compression artifacts entirely — at the cost of large files. **MP4** with a modern codec is the right choice for distribution.

#### Video Codecs

The Container selection filters which codecs are available:

| Container | Available Codecs |
|-----------|------------------|
| **MOV** | ProRes 422 HQ, ProRes 4444 XQ |
| **MP4** | H.264 High, H.265 Main, H.265 Main10, AV1 |
| **MKV** | VP9 |

- **AV1 (libaom‑av1)** — modern, high‑efficiency codec; excellent quality‑per‑bit for distribution. Uses `-row-mt 1` row‑based multithreading and a `-cpu-used` speed knob (0 = slow/best → 8 = fast).
- **NVENC (hardware acceleration)** — where available, hardware H.264/H.265 encoders offload encoding to the GPU's dedicated media engine (e.g. `-preset p4`), trading a little quality for very fast, low‑CPU encodes.
- **ProRes (422 HQ / 4444 XQ)** — intra‑frame, archival‑grade quality with 10‑/12‑bit color (`yuv422p10le` / `yuva444p12le`). Large files, but ideal as an editing/restoration master.

#### Encoder Arguments

| Control | Type | Options / Effect |
|---------|------|------------------|
| **Video Backend** | Dropdown | `ffmpeg` (recommended, high‑quality, full codec/10‑bit support) or `opencv` (fallback VideoWriter; MP4/AVI only, no 10‑bit). |
| **Bitrate Mode** | Dropdown | `VBR/CRF` (quality‑targeted) or `CBR` (constant bitrate). |
| **Quality Level** (CRF mode) | Dropdown | `Max` (CRF 18, largest/best), `High` (CRF 23), `Medium` (CRF 28), `Low` (CRF 32, smallest). |
| **Target Bitrate** (CBR mode) | Dropdown | `1`–`180` Mbps presets. |
| **10‑bit Output** | Checkbox | Forces a 10‑bit pixel format for codecs that support it (smoother gradients, less banding). |

**Custom FFmpeg flags.** Per‑codec profiles inject the appropriate FFmpeg arguments automatically — for example **AV1** adds `-row-mt 1` (row multithreading) and a `-cpu-used` CPU‑utilization/speed flag, **NVENC** sets a `-preset`, and **ProRes** sets `-profile:v` and a 10/12‑bit `-pix_fmt`. **CRF vs Bitrate**: CRF (via *Quality Level*) targets a constant *perceptual quality* and lets the file size float; CBR (via *Target Bitrate*) pins the bitrate and lets quality float — use CRF for masters, CBR when you need a predictable file size/stream rate.

#### Audio Modes

A single **Audio** dropdown controls the audio track of the output:

| Mode | FFmpeg behavior |
|------|-----------------|
| **Copy Audio** | `-c:a copy` — passes the source audio through untouched (no re‑encode, lossless, fastest). |
| **AAC** | `-c:a aac -b:a 192k` — re‑encode to AAC. |
| **PCM** | `-c:a pcm_s24le` — uncompressed 24‑bit PCM (archival). |
| **AC3** | `-c:a ac3 -b:a 448k` — Dolby Digital. |
| **FLAC** | `-c:a flac` — lossless compressed. |
| **No Audio** (Mute/Strip) | `-an` — strips the audio track entirely. |

---

### 2.4 Execution & Hardware Optimization (VRAM Management)

These controls live across the **Processing Settings**, **Device Management**, **Memory (BlockSwap)** *[Advanced]*, **VAE Tiling**, **Performance** *[Advanced]* and **Model Cache** *[Advanced]* groups.

#### Batch Size & Uniform Batch Size

| Control | Type | Range / Rule | Effect |
|---------|------|--------------|--------|
| **Batch Size** | Stepper (− / +) | Must be `4k+1` (1, 5, 9, 13, 17, …); typed values snap automatically; ±4 buttons step it | Number of frames processed in parallel per pass. **This is the dominant VRAM lever** — memory scales roughly linearly with batch size. Larger batches are faster and give the temporal model more context, but are the first thing to reduce on OOM. |
| **Uniform Batch Size** | Checkbox | — | Forces every batch to the same size (no smaller "remainder" final batch), which keeps VRAM behavior predictable across the whole job. |

> [!WARNING]
> Batch size and target resolution together determine peak VRAM. If you hit Out‑Of‑Memory, reduce **Batch Size** by one `4k+1` step before touching anything else.

#### Offloading Toggles (Trading RAM for VRAM safety)

| Control | Type | Options | Effect |
|---------|------|---------|--------|
| **DiT Offload to CPU** | Dropdown | `none`, `cpu` | Moves DiT weights to system RAM when idle, freeing VRAM for active computation. |
| **VAE Offload to CPU** | Dropdown | `none`, `cpu` | Offloads the VAE to RAM between phases — valuable because VAE decode is the heaviest VRAM stage. |
| **Tensor Offload** | Dropdown | `none`, `cpu` | Offloads intermediate tensors to RAM. |

Offloading **trades system RAM (and a little PCIe transfer time) for VRAM headroom**. On constrained GPUs, enabling CPU offload for DiT and/or VAE is often the difference between completing a job and crashing.

#### Memory (BlockSwap) *[Advanced]*

| Control | Type | Range | Effect |
|---------|------|-------|--------|
| **Blocks to Swap** | Spin | `0`–`36` (`0` = disabled) | Swaps a number of transformer blocks between GPU and CPU on demand, sharply lowering peak VRAM at the cost of throughput. |
| **Swap I/O Components** | Checkbox | — | Also swaps the model's input/output components for additional VRAM savings. |

#### Attention Modes *[Advanced]*

A single **Attention Mode** dropdown selects the attention kernel used inside the transformer:

| Mode | What it does |
|------|--------------|
| `sdpa` | PyTorch's built‑in scaled‑dot‑product attention — the universal, always‑available baseline. |
| `flash_attn_2` | **Flash Attention 2** — fused, memory‑efficient attention; speeds up the transformer and lowers attention memory on supported GPUs. |
| `flash_attn_3` | **Flash Attention 3** — newer FA generation with further speedups on the latest hardware. |
| `sageattn_2` | **SageAttention 2** — quantized attention kernel for faster inference. |
| `sageattn_3` | **SageAttention 3** — latest SageAttention; maximizes attention throughput on capable GPUs. |

> [!NOTE]
> Flash/Sage kernels accelerate the **attention** portion of the DiT and reduce its memory use, but they require a compatible GPU and a correctly built backend (e.g. **Triton** support underpins several of these kernels). If a kernel isn't available on your system, fall back to `sdpa`.

#### Model Caching (`--cache_dit` / `--cache_vae`) *[Advanced]*

| Control | Type | Effect |
|---------|------|--------|
| **Cache DiT** | Checkbox | Keeps the DiT model resident/cached across work units. |
| **Cache VAE** | Checkbox | Keeps the VAE resident/cached across work units. |

Caching keeps models loaded so they aren't repeatedly re‑initialized — a clear win when **streaming** a long video or **batch‑processing a directory**, where the same model is reused across many chunks/files. It is **bypassed for trivial single‑frame work** (e.g. a single‑image preview), where there's nothing to amortize the cache against. Note also that on **multi‑GPU without streaming**, caching is automatically disabled because workers must cache within their own chunk loops.

#### VAE Tiling & Decode Tile‑Size Clamping

| Control | Type | Range | Effect |
|---------|------|-------|--------|
| **Encode Tiled** | Checkbox | — | Tiles the VAE *encode* pass to bound its VRAM. |
| **Encode Tile Size** | Spin | `128`–`4096` (default `1024`) | Tile dimension for encode. |
| **Encode Tile Overlap** | Spin | `0`–`512` | Overlap between encode tiles (reduces seams). |
| **Decode Tiled** | Checkbox | — | Tiles the VAE *decode* pass — the heaviest stage. |
| **Decode Tile Size** | Spin | `128`–`4096` (default `1024`) | Tile dimension for decode. |
| **Decode Tile Overlap** | Spin | `0`–`512` | Overlap between decode tiles. |
| **Tile Debug** | Dropdown | `false`, `encode`, `decode` | Visual/diagnostic tiling overlay. |

> [!WARNING]
> **VAE Decode Tile‑Size Clamping (1024 → 256).** VAE decode is where most OOM crashes happen. To prevent them, the pipeline **throttles the decode tile size down from 1024 toward 256** as part of its automatic recovery stages — decoding the frame in smaller tiles uses far less peak VRAM. This clamp is what lets very high‑resolution decodes complete on limited GPUs instead of crashing.

#### Streaming Mode & Chunk Controls

| Control | Type | Range / Default | Effect |
|---------|------|-----------------|--------|
| **Enable Video Chunking** | Checkbox | — | Splits long videos into time‑based segments processed one at a time. |
| **Chunk Duration (Minutes)** | Spin | `1`–`120` min (default `3`) | Segment length. Converted at runtime as `minutes × 60 × source_FPS` frames. |
| **Temporal Overlap** | Spin | Int | Frames shared between consecutive chunks to keep motion continuous across segment boundaries. |
| **Prepend Frames** | Spin | Int | Extra leading frames fed into each chunk for temporal context. |
| **Skip First Frames** | Spin | Int | Skips N frames at the start of the source. |
| **Load Cap Frames** | Spin | `0` = all | Caps how many frames are loaded (Preview temporarily sets this to a small value and restores it afterward). |
| **Only Frames** | Spin | `0` = no limit | Caps frames per VAE‑decode chunk to prevent OOM. |

**How huge archives stay safe.** With chunking enabled, the pipeline splits a massive video — think **50,000+ frame** 20th‑century archives — into digestible segments (for example, **~2,700‑frame** chunks for a few minutes of footage), upscales each segment independently with a temporal overlap to avoid visible seams, and streams the results to disk. Because only one chunk's worth of frames is ever resident at once, peak VRAM stays bounded no matter how long the source is. Chunked outputs are written as ordered parts (e.g. `seedvr2_output_part_NNN_MMMMM.<ext>`).

#### Device Management

| Control | Type | Effect |
|---------|------|--------|
| **GPU Device** | Checkable dropdown | Select one or more GPUs. `Auto`/`CPU` are exclusive; multiple `GPU N` entries can be checked together for multi‑GPU inference. |

---

### 2.5 Quality Control *[Advanced]* & Color

| Control | Type | Range | Effect |
|---------|------|-------|--------|
| **Input Noise Scale** | Double spin | `0.0`–`1.0` | Injects noise on the input. Reduces artifacts at high resolutions. |
| **Latent Noise Scale** | Double spin | `0.0`–`1.0` | Noise in latent space — softens detail when needed. |
| **Color Correction** | Dropdown | `lab`, `wavelet`, `wavelet_adaptive`, `hsv`, `adain`, `none` | Post‑process color matching of the result back toward the source. |

### 2.6 Debug *[Advanced]*

| Control | Type | Effect |
|---------|------|--------|
| **Verbose Debug** | Checkbox | Enables detailed logging from the engine into the log pane. |
| **Auto Tune** | Checkbox | Pre‑flight VRAM detection; forces VAE tile overlap to 32 and auto‑reduces batch size on OOM (up to 5 retries) instead of crashing. |
| **Sound Notifications** | Checkbox | Plays a system sound on completion/error. |

---

### 2.7 Verification & Preview

The preview workflow lets you judge results on a single frame before committing to a full render.

#### Preview Capture Button (⚡)

Clicking **⚡ Preview** captures the current frame from the timeline, upscales it as a single frame, and shows original vs upscaled side‑by‑side. Internally it:

- Saves the captured source frame as a temporary 16‑bit TIFF (`seedvr2_preview_input_frame_001.tiff`).
- Temporarily sets batch size to 1 (single‑frame) and a small load cap, then **restores your real settings** when the preview finishes.
- Writes the upscaled result as `seedvr2_preview_frame_<timestamp>.tiff`.

#### Direct Output Generation (no subfolders)

The preview output is written **directly beside the source video**, not into a generated subfolder. The pipeline detects that the preview target is a concrete image file path (ending in an image extension such as `.tiff`) and writes to that exact filename, creating only its parent directory — so the comparison engine always finds the file exactly where it expects it.

#### Live Split‑Screen Mode

When the preview completes, the dual **QPixmap** comparison engine:

1. Loads **both** TIFFs directly from their tracked on‑disk paths — bypassing the UI text fields entirely.
2. Validates each load (`QPixmap` not null) before assigning the left (original) and right (upscaled) sides.
3. **Programmatically forces the UI into split‑view** instantly (checks the split‑mode button and switches the viewer layout), so you drop straight into a draggable‑divider, zoom/pan comparison without any extra clicks.

---

## 3. Optimization & Archive Restoration Recommendations

### Maximizing modern high‑end NVIDIA GPUs (≈16 GB VRAM)

> [!TIP]
> **16 GB sweet spot.**
> - **DiT Model:** `seedvr2_ema_3b-Q8_0.gguf` (default) for the best quality/VRAM balance; step up to a 7B FP8 variant only if you have headroom to spare.
> - **Attention:** Flash Attention 2/3 or SageAttention 2/3 if your build supports them — faster and lighter than `sdpa`.
> - **Batch Size:** Start around `9`–`13` (`4k+1`) at 1080p targets and increase while VRAM allows; back off one step on OOM.
> - **VAE Decode:** Enable **Decode Tiled** and rely on the **1024 → 256 tile clamp** for high‑resolution targets.
> - **Offload:** Leave DiT/VAE offload at `none` while you have VRAM headroom; flip VAE offload to `cpu` first if you approach the limit.
> - **Auto Tune:** On, as a safety net for occasional OOM spikes.

### Restoring heavy / long 20th‑century film archives (OOM‑safe)

> [!TIP]
> **Long‑archive recipe (avoid OOM on 50k+ frame sources):**
> - **Enable Video Chunking** with a **Chunk Duration** of `2`–`3` minutes so each segment is a few thousand frames (≈2,700) rather than the whole film.
> - Set a small **Temporal Overlap** (and a few **Prepend Frames**) to keep motion seamless across chunk boundaries.
> - Turn on **Cache DiT** and **Cache VAE** — caching pays off precisely in streaming/long jobs because the models are reused across every chunk.
> - Enable **DiT Offload → cpu** and **VAE Offload → cpu** to trade abundant system RAM for scarce VRAM.
> - Keep **Batch Size** conservative (`5`–`9`) and **VAE Decode Tiled** on so the heavy decode phase stays clamped.
> - For a true archival master, export an **Image Sequence** in **16‑bit TIFF** (or ProRes 4444 XQ for a video master) rather than a lossy MP4.
> - Use a fixed deterministic seed (the GUI default) so re‑renders of the same reel match exactly.

---

## 4. Changelog & Recent Upgrades

### Latest crucial updates

- **Fixed Preview subfolder bug.** Preview outputs now sit **cleanly next to the input video** instead of being dropped inside a newly created subfolder. The saver detects a direct image file path (e.g. `..._preview_upscaled.tiff`), creates only the parent directory, and writes the upscale to that exact filename — so the split‑screen view always locates the result.
- **Fixed Split‑Screen view responsiveness.** The comparison engine now performs **automatic internal path tracking** (loading both TIFFs from their tracked disk paths), **load validation** (skipping null pixmaps and falling back gracefully), and **automated layout switching** (programmatically forcing the UI into split‑view the instant a preview completes).
- **Safe dynamic‑shape execution.** Implemented `compile_dynamic=True` with CUDA‑Graph capture disabled for dynamic‑shape‑safe runs, plus **dynamic‑shape memory clamp adaptations** (including the VAE decode tile‑size step‑down) so variable‑resolution and variable‑length jobs complete without VRAM crashes.

### Earlier in v1.7 Beta

- **Auto Tune** (replaces "Auto Safeguard"): pre‑flight VRAM detection, forced VAE tile overlap of 32, and automatic batch‑size reduction on OOM (up to 5 retries).
- **Container‑driven codec selection**: the **Container** dropdown filters the **Video Codec** list to compatible codecs only.
- **Broader hardware detection**: NVIDIA CUDA, Intel XPU, AMD ROCm and CPU fallback.

---

## License

Apache License 2.0 — see [LICENSE](LICENSE).

© ByteDance Seed (SeedVR2 model) · Naxci1 (1‑Click GUI)

## Links

- 🌟 [GitHub Repository](https://github.com/naxci1/1Click_SeedVR2.5)
- 🔬 [Original SeedVR2 Project](https://github.com/ByteDance-Seed/SeedVR)
