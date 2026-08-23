# 1-Click SeedVR 2.5 — Release Notes (v1.9.48b → v1.9.54b)

Everything new since v1.9.47b.

---

## v1.9.54b — last_used.json relocation

- **last_used.json moved next to config.json** (project root) — processing
  settings and app settings live side by side; named presets stay in the
  configurable presets directory.

## v1.9.53b — Temp directory, persistent logs, one-click log shipping

- **Temp moved to `C:\1Click_SeedVR2.5\temp`** — all transient work (queue
  files, previews, chunk runs) lives with the other shared assets
  (`python_embeded`, `models`, `presets`), configurable via the Temp
  directory setting.
- **"Log directory" setting row** in Settings — where `app.log`,
  `error.log` and failure reports are written.
- **Debug output ON by default.**
- **access.log** — every CLI log line is appended (timestamped) across
  sessions. **Show Log opens with previous-run history** (last 3000 lines)
  instead of a blank window; Clear wipes both view and file.
- **Send Logs button** on the error dialog — zips `error.log`,
  `access.log`, `app.log` and `config.json` and emails the archive to the
  developer fully in the background. SMTP credentials are embedded in the
  binary (obfuscated, never shown in the UI) — zero configuration.

## v1.9.52b — Single config.json + dedicated log/presets directories

- **One config.json, in the project root only.** The EXE (inside `dist\`)
  anchors one level up, so source runs and the frozen EXE read/write the
  same file — `dist\config.json` and `dist\temp` are gone for good.
- **config.json contains only the Settings-dialog values** (runtime paths,
  temp/presets/log directories, alarm, systray, session I/O).
- Processing settings (batch, tiles, model, offloads…) live in
  `presets\last_used.json` — separated by design.
- **New `log\` directory** for app/error/failure logs, configurable via the
  "Log directory" setting.
- **Presets directory is configurable** ("Presets directory" setting,
  default `C:\1Click_SeedVR2.5\presets`).

## v1.9.51b — Executable renamed

- `SeedVR2_GUI.exe` → **`1-Click_SeedVR_2.5.exe`** (InternalName and
  OriginalFilename metadata updated as well).

## v1.9.50b — Application renamed

- Display name is now **"1-Click SeedVR 2.5"** (window title, tray tooltip,
  EXE file description, product name).

## v1.9.49b — System tray toggle

- New **System tray** on/off switch in Settings (below Alarm sounds),
  persisted in config.json, default on.
  - **ON:** closing the window hides the app to the tray; processing
    continues in background; full quit via the tray menu.
  - **OFF:** no tray icon; closing the window really quits the app.
- Applies immediately — no restart needed.

## v1.9.48b — Standard output colors, error reporting, voice alerts

### Standard-compliant output colors (breaking fix)
- The raw RGB pipe reached NVENC and was encoded as GBR **4:4:4**
  (`Main 4:4:4` + identity matrix) — many editors/players misinterpret it
  and shift colors. The writer now forces `-pix_fmt yuv420p` plus an
  explicit bt709 RGB→YUV matrix (`-vf scale=out_color_matrix=bt709:out_range=tv`),
  so output is `Main / yuv420p / bt709 / tv` — the same standard combo
  ComfyUI writes. Verified with a real NVENC encode + ffprobe.
- Image-sequence export gets the same matrix fix. RGB targets (FFV1,
  ProRes 4444) unaffected.

### Detailed failure reporting
- Failures no longer show a bare "Exit code 1": the dialog opens with
  details expanded, headed by the most meaningful error line, and the full
  report (command + last 100 log lines) is saved to
  `log\cli_failure_<timestamp>.log` with Copy-to-Clipboard.

### Voice (TTS) notifications
- Success/failure alarms now speak via Windows SAPI ("Processing completed
  successfully." / "An error occurred during processing."), falling back to
  the old chime when no voice is installed.

### Project panel outcome colors
- Finished files turn green, failures red, cancellations amber.

### Export defaults
- Quality level defaults to **High** (codec default was already H265).

### Other
- Removed the non-functional "Side by Side" preview button.
- 7B config selection is case-insensitive — a file named with an uppercase
  "7B" no longer silently loads the 3B config and fails with a confusing
  architecture mismatch. Dry-run structure test passes for all 9 registered
  DiT models (3B/7B/sharp, GGUF/fp8/fp16).
