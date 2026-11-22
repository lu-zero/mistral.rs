# Olala state

## PR split

- #2563 (`pr/12-olola-model` @ `9f33bca65`): native Olala GGUF support plus the three
  backend-agnostic perf commits (quantized expert gather, `forward_group` shared-input
  projections, host-cached router bias). No `target_arch` / `is_cpu` code. Works on CUDA.
- #2565 (`pr/17-olala-cpu-kernels` @ `0bedd5627`): stacked on #2563. Gated aarch64 NEON
  Mamba scan, three numerical harnesses, and the tensor-fallback shape fix. No-op on CUDA,
  Metal, and non-aarch64. `OLALA_TENSOR_MAMBA=1` forces the exact Candle path.
- Merge order: #2563 then #2565.

## Numerics

- Single-step harness: fused/NEON path matches Candle tensor ops to <2e-7.
- Full model: hand-written f32 rounding differs by ~2e-7 per step and compounds over 36
  layers. Logit cos vs the OVH fork oracle: tensor path 0.9987 / 0.99985, NEON 0.9986 /
  0.9979 on the two checked tokens. Argmax differs on the second token. Relaxed bar accepted.
- Parity oracle lives in `olala_cpu.rs::tensor_mamba_step`. Diff against it when the tensor
  fallback misbehaves.

## Perf measurements (1.0 GHz, q8_0, this host)

| Config | Decode |
|---|---|
| default 128 threads, NEON | ~438 ms/tok (depth 4) |
| 16-32 threads, NEON | ~225 ms/tok |
| OVH llama.cpp fork | ~80 ms/tok (12.5 tok/s) |

Thread count is the big lever; see `PERF-LEVERS.md`. The NEON scan is only ~2% of the
16-thread profile.

## What is left

1. Port the parallel stacked-expert gather from `cpu-decode-bandwidth:mistralrs-quant/src/moe_gather_cpu.rs`
   if the indexed GEMV path ever declines (unquantized fallback). Do not rewrite it.
2. Re-sweep `MIN_PARALLEL_OUTPUTS` and thread count at 3.4 GHz under `performance`, then
   restore `schedutil`. The previous optima were found at 1.0 GHz.
3. Look at PR #2489 (mixed row-count CPU GGUF matmul) for prefill; prompt lengths after chat
   templating are rarely 1 or divisible by 4.
4. Fuse small elementwise ops (relu^2, gate, geodesic, norms) if the 16-thread profile still
   shows many tiny barrier syncs after the thread cap.

## What not to redo

- Scalar fused Mamba: tried, slower in release, parity drift. Kept as harness only.
- Closed-form batched prefill: tried, exact but ~8x slower. Kept as harness only.
- NEON state scan without a thread cap: 1.35x, which is dwarfed by the 2.2x thread-cap win.
- Any GGUF expert dequantization work: already fixed on `cpu-decode-bandwidth`.
