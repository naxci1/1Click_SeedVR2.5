# 1-Click SeedVR 2.5 — Release Notes (v1.9.58b)

Everything new since v1.9.57b.

---

## v1.9.58b — FP8 model crash fix, Send Logs wiring

### Fixed: FP8 model crash (critical)
- Runs with an **FP8 DiT model** (`seedvr2_ema_*_fp8_e4m3fn.safetensors`) crashed
  with `RuntimeError: Promotion for Float8 Types is not supported, attempted
  to promote Float and Float8_e4m3fn`.
- Root cause: `CustomRMSNorm` deliberately skipped the dtype cast for FP8
  weights, so `F.rms_norm(float_input, fp8_weight)` hit PyTorch's unsupported
  float8 promotion. `CustomLayerNorm` already handled this correctly.
- Fixed in both the 3B and 7B model paths: the norm weight is now always
  cast to the activation dtype (FP8 → fp16/bf16 cast is fully supported).
- Verified by reproducing the exact crash synthetically before the fix and
  confirming clean output afterwards, plus regression checks for the
  GGUF (Q8) fast path.

### Fixed: Send Logs button on unhandled-exception dialog
- The error dialog shown for GUI-internal unexpected exceptions had a
  **Send Logs button that was never connected** — clicking it did nothing.
  It is now wired to the background log-email flow, so every error path
  (processing failures and internal exceptions) can report by mail.
- The mail pipeline was live-tested end-to-end over SMTP.

### Notes for users seeing old error screenshots
- The "Send Logs" button exists on the processing-failure dialog since
  v1.9.48b — screenshots without it come from older releases; updating
  fixes that.
