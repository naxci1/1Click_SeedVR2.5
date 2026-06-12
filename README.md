# 1Click_SeedVR2.5 by Naxci1

> **Version 1.7 Beta** — Professional AI Video Upscaler powered by SeedVR2

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![GitHub](https://img.shields.io/badge/GitHub-naxci1%2F1Click__SeedVR2.5-black)](https://github.com/naxci1/1Click_SeedVR2.5)

---

## What Is This?

1Click_SeedVR2.5 is a one-click GUI and CLI wrapper for the SeedVR2 AI video upscaling model by ByteDance Seed. It upscales video to higher resolutions (up to 4K) using a 3B or 7B parameter diffusion transformer.

---

## What's New in v1.7 Beta

- **Auto Tune**: Replaces "Auto Safeguard". Performs pre-flight VRAM detection, forces VAE tile overlap to 32, and automatically reduces batch size on OOM (up to 5 retries without crashing).
- **Container-driven Codec Selection**: A new "Container" dropdown (MP4 / MKV / MOV) filters the "Video Codec" list to only show compatible codecs.
- **Clean Codec List**: Streamlined to: ProRes 422 HQ, ProRes 4444 XQ, H.264 High, H.265 Main, H.265 Main10, AV1, VP9.
- **Intel & AMD GPU Support**: Hardware detection now covers NVIDIA CUDA, Intel XPU, AMD ROCm, and CPU fallback.
- **Debug Mode Fixed**: Verbose debug logging now works in both Simple and Advanced mode.
- **No Resolution Cap in Simple Mode**: User-defined resolution is always respected.
- **Updated Branding**: Version 1.7 beta by Naxci1.

---

## System Requirements

| Resolution | Min VRAM | Recommended VRAM | Min RAM |
|------------|----------|------------------|---------|
| 480p → 720p | 8 GB | 12 GB | 16 GB |
| 720p → 1080p | 10 GB | 16 GB | 24 GB |
| 1080p → 2K | 12 GB | 20 GB | 32 GB |
| 1080p → 4K | 16 GB | 24 GB | 48 GB |

> **Note**: These are estimates. Actual VRAM usage depends on batch size, tiling settings, and model size (3B vs 7B).

---

## Hardware Support

| Hardware | Status | Notes |
|----------|--------|-------|
| NVIDIA (CUDA) | ✅ Full | RTX 20xx–50xx series recommended |
| AMD (ROCm) | ✅ Supported | Requires PyTorch ROCm build |
| Intel (XPU) | ✅ Supported | Requires PyTorch XPU build |
| CPU | ✅ Fallback | Very slow; for testing only |
| Apple MPS | 🔜 Planned | macOS Metal GPU support |

---

## Installation

### Prerequisites

- Python 3.10 or 3.11
- PyTorch 2.0+ (with CUDA, ROCm, or XPU support matching your hardware)
- FFmpeg in PATH (for video encoding)

### Quick Setup

```bash
git clone https://github.com/naxci1/1Click_SeedVR2.5.git
cd 1Click_SeedVR2.5
pip install -r requirements.txt
```

### Models

Download the SeedVR2 models and place them in `models/SEEDVR2/`:

- `seedvr2_ema_3b-Q8_0.gguf` — DiT model (3B, recommended)
- `seedvr2_ema_7b-Q8_0.gguf` — DiT model (7B, higher quality)
- `ema_vae_fp16.safetensors` — VAE decoder

---

## Usage

### GUI Mode

```bash
python -m gui.app
```

The GUI provides:
- **Simple Mode**: Drag-and-drop video input, choose resolution, click Export
- **Advanced Mode**: Full control over batch size, tiling, model caching, hardware

### CLI Mode

```bash
python inference_cli.py \
  --input video.mp4 \
  --output output/ \
  --resolution 1080 \
  --batch_size 41 \
  --dit_model seedvr2_ema_3b-Q8_0.gguf \
  --vae_model ema_vae_fp16.safetensors
```

Key CLI arguments:

| Argument | Default | Description |
|----------|---------|-------------|
| `--input` | required | Input video or image path |
| `--output` | required | Output directory |
| `--resolution` | 720 | Target short-side resolution (pixels) |
| `--batch_size` | 41 | Frames per batch (4n+1 values recommended) |
| `--auto_tune` | off | Enable Auto Tune OOM recovery |
| `--debug` | off | Verbose debug logging |
| `--vae_encode_tiled` | off | Enable VAE encode tiling |
| `--vae_decode_tiled` | off | Enable VAE decode tiling |

---

## Auto Tune

When `--auto_tune` is enabled (or "Auto Tune" checkbox in GUI):

1. **Tile Overlap**: Forced to 32 for both encode and decode to reduce VRAM fragmentation.
2. **OOM Recovery**: If a CUDA Out of Memory error occurs during processing, the system automatically:
   - Reduces `batch_size` by 4
   - Clears the CUDA memory cache
   - Retries the operation
   - Repeats up to **5 times** before failing
3. This prevents crashes on systems near their VRAM limit.

---

## Container & Codec Selection

The GUI now uses a two-step workflow:

1. **Select Container** (MP4 / MKV / MOV)
2. **Select Codec** — only codecs compatible with the chosen container appear

| Container | Available Codecs |
|-----------|-----------------|
| MOV | ProRes 422 HQ, ProRes 4444 XQ |
| MP4 | H.264 High, H.265 Main, H.265 Main10, AV1 |
| MKV | VP9 |

---

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.

© ByteDance Seed (model), Naxci1 (GUI/CLI wrapper)

---

## Links

- 🌟 [GitHub Repository](https://github.com/naxci1/1Click_SeedVR2.5)
- 📺 [Tutorial Videos](https://www.youtube.com/@AInVFX)
- 🔬 [Original SeedVR2 Paper](https://github.com/ByteDance-Seed/SeedVR)
