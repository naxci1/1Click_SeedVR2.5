# 1-Click SeedVR 2.5 — Release Notes (v1.9.57b)

Everything new since v1.9.56b.

---

## v1.9.57b — VAE tiling controls, final shape

### Tile size (encode & decode)
- Slider range **64–1024 in steps of 16** (64, 80, 96, …) — the slider
  itself never exceeds 1024.
- **Manual entry up to 2048**: click the number and type any value —
  it is kept exactly as typed (not rounded to the 16 grid); the slider
  parks at its maximum while the typed value stays in effect.
- Typed values are clamped to 64–2048; invalid text keeps the previous
  value. Default remains **512**.

### Tile overlap (encode & decode)
- Now a **slider with exactly three stops: 32 / 64 / 128** (default **32**).
- Typing a number snaps to the nearest stop (50 → 64, 100 → 128), so only
  the three allowed values can ever be sent.
- Legacy saved values migrate to the nearest stop (old 0 → 32).
- The GUI still caps the overlap at half the tile size.

### Notes
- Manually-entered exact tile values survive saves, presets and restarts.
- All behaviors covered by automated offscreen UI tests (slider stops,
  manual entry, clamping, migration, preset restore, regression on the
  other sliders).
