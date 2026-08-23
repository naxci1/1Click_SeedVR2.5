# 1-Click SeedVR 2.5 — Release Notes (v1.9.56b)

Everything new since v1.9.55b.

---

## v1.9.56b — VAE tiling controls rework, exact manual tile sizes

### VAE tile size (encode & decode)
- Range is now **64–2048 px** (was 64–1024); default **512** for both.
- The slider moves in **steps of 16** (64, 80, 96, …).
- **Click the number to type an exact value** — manual entries are kept
  as typed (not rounded to the 16-px grid), clamped to the 64–2048 range.
  Invalid text keeps the previous value; nothing can crash the run.
- Exact manually-entered values survive settings saves and preset reloads.

### Tile overlap (encode & decode)
- Simplified to fixed choices **32 / 64 / 128** (default **32**, both).
- Legacy saved values (e.g. the old default 128 or 0) migrate gracefully:
  out-of-list values fall back to 32 with no error.
- The GUI additionally caps the overlap at **half the tile size**, so a
  small tile can never be paired with a larger overlap.

### Downscale filter naming
- Filter names in the Pre-downscale dropdown now use the familiar
  conventions: **Bicubic + Anti-alias** (default), Bicubic,
  Bilinear + Anti-alias, Bilinear, Area (Box), Nearest, **Lanczos**,
  **Mitchell-Netravali**.

### Other
- CLI defaults synchronized: `--vae_*_tile_size 512`,
  `--vae_*_tile_overlap 32`.
