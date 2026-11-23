# Branch and PR inventory

Tip commits as of 2026-11-22. `fork` is `lu-zero/mistral.rs`; `origin` is upstream.

## Olala line

| Branch | Tip | PR | Status |
|---|---|---|---|
| `pr/12-olola-model` | `9f33bca65` | #2563 | folded: native support + backend-agnostic perf |
| `pr/13-olola-expert-kernels` | `302f464ba` | none | superseded: expert gather + forward_group folded into #2563 |
| `pr/14-olala-mamba-parity` | `a63127b98` | none | superseded: harnesses folded into #2565 |
| `pr/15-olala-moe-hostsync` | `5648bc2d8` | none | superseded: bias hoist folded into #2563 |
| `pr/16-olala-neon` | `3610c2c35` | none | superseded by pr/17 |
| `pr/17-olala-cpu-kernels` | `0bedd5627` | #2565 | folded: gated aarch64 NEON + 3 harnesses + fallback fix |
| `pr/18-cpu-threads-flag` | `c024927eb` | #2568 | `--cpu-threads N` caps Rayon + Candle pools before init; 3.15x on Olala |
| `pr/20-olala-gemv` | `725bc5a09` | none | local verification of #2514 + parity fix on Olala; delete after #2514 merges |


PR #2563 is the model-support PR: no CPU-specific code, fast on CUDA via `QuantMethod`.
PR #2565 is stacked on #2563: aarch64-CPU-only NEON scan, no-op elsewhere. Merge #2563 first.

Local-only Olala experiments that were deliberately not folded:

- scalar fused Mamba step: parity drift and slower in release (0.811 s vs 0.682 s).
- closed-form batched prefill: numerically exact but ~8x slower; kept as a harness.

## Kolibri / Flint / CPU decode line

| Branch | Tip | PR | Content |
|---|---|---|---|
| `pr/01-fp8-dequant-f32` | `52fd27d59` | #2505 | fp8 dequant in f32 |
| `pr/01-rmsnorm-scale-f32` | `bbdb8b832` | none | RMSNorm scale in f32 |
| `pr/02-cpu-decode-engine` | `f444bc83f` | none | output-parallel GEMV + per-expert gather |
| `pr/03-kolibri1-model` | `fa2f4f0cf` | none | Kolibri-1 model (earlier) |
| `pr/04-bfmmla-reduction-order` | `eacdc5f3e` | none | BFMMLA bf16 reduction order |
| `pr/05-layer-taps-and-bench` | `e801c61fe` | none | per-layer taps for cross-impl diff |
| `pr/06-cpu-attn-sinks` | `85f051173` | #2508 | attention sinks on CPU kernel |
| `pr/07-gguf-selected-experts` | `348c6a567` | #2509 | dequant only routed GGUF experts |
| `pr/08-iq-gguf` | `a91e3a3b2` | #2510 | IQ GGUF from block bytes |
| `pr/09-kolibri1-model` | `05b1e1f23` | #2511 | Kolibri-1 model |
| `pr/10-cpu-gemv` | `edae0710c` (rebase: `b0736f716`) | #2514 | split CPU decode GEMV across output rows + Olala parity fix |
| `pr/11-q6k-rows` | `92ea0b93a` | #2525 | dot dense Q6_K rows from blocks |
| `cpu-decode-bandwidth` | `578b99994` | none | integration branch for all of the above plus IQ NEON, fused gather, attention sinks, Q3_K/Q5_K expert dots |
| `docs/perf-record` | `ae03b119f` | none | perf record docs + detailed `todo/` tree |
| `apex-pr-stack` | `83e2cb4da` | none | merge of pr/10 into the stack |
| `flint_by_flint` | `e3a918490` | none | todo: Kolibri and Flint line is one tip |

The `rebase/prNN` branches are the rebased equivalents of their `pr/NN` counterparts and
are what the corresponding PRs point at. Prefer the `rebase/` tips when folding work in.

## Other open PRs relevant to CPU perf

| PR | Branch | Topic |
|---|---|---|
| #2489 | `cpu-gguf-mixed-dispatch` | split CPU GGUF matmul so packed kernels engage at any row count |
| #2558 | `perf-2557-cpu-moe-expert-cache` | dequantize and cache only routed experts on CPU MoE |
| #2556 | `fix-2492-cpu-offloaded-moe-experts` | run CPU-offloaded MoE experts with CUDA activations |

## Inkling-2 line (peer work)

| Branch | Tip | PR | Note |
|---|---|---|---|
| `feat/inkling2-mini` | `d6e5b6b28` | #2566 | peer owns the loop semantics |
| `inkling2-expert-scales` | `caad5e65e` | none | NVFP4 per-expert scale helper, ready to cherry-pick |

## Rule for new work

If a fix already exists on a branch in this table, cherry-pick or port it. Do not write a
second implementation. When a branch is superseded, update its row here rather than deleting
the branch.
