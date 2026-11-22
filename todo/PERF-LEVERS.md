# Known perf levers

Do not re-measure these from scratch unless the stated conditions have changed.

## Work partitioning / thread count

This is the largest known lever on this host and it has been found twice.

- Prior finding (Kolibri-1, `cpu-decode-bandwidth:todo/README.md`): changing
  `MIN_PARALLEL_OUTPUTS` from 1024 to 512 gave **2.35x**. It is a work-partitioning threshold,
  and its optimum was found at 1.0 GHz. That note explicitly says to re-sweep at 3.4 GHz.
- Olala re-measurement (2026-11-22, 1.0 GHz, q8_0, prompt-len 8, gen-len 16, depth 1,
  aarch64 host, release build of `pr/17-olala-cpu-kernels`):

  | `CANDLE_NUM_THREADS` | TPOT |
  |---|---|
  | unset (128) | 497.6 ms |
  | 8 | 241.1 ms |
  | 16 | 225.4 ms |
  | 24 | 242.5 ms |
  | 32 | 224.1 ms |
  | 48 | 345.8 ms |
  | 64 | 418.4 ms |
  | 96 | 356.6 ms |

  So 16-32 workers give ~2.2x over the default. Above 32 it degrades sharply.

  **Correction: the earlier sweep only capped Candle's barrier pool.** The `gemm` crate
  (F32 matmul) uses rayon, so `RAYON_NUM_THREADS` matters too. Re-measured 2026-11-22:

  | settings | TPOT |
  |---|---|
  | neither set | 618.5 ms |
  | `CANDLE_NUM_THREADS=16` | 284.4 ms |
  | `RAYON_NUM_THREADS=16` | 224.5 ms |
  | both =16 | **196.3 ms** (3.15x) |
  | both =8 | 212.0 ms |

  Paired A/B (`scripts/ab.py`, 6 rounds, alternating): wall ratio 0.5314, 95% CI
  [0.4818, 0.5855] (resolved); CPU time ratio 0.1216 (8.4x less CPU burned); RSS ratio
  0.9984 (unresolved, effectively unchanged). Logits are bit-identical between thread
  settings (max abs diff 0.0 on the two-token oracle).

  Mechanism: 128 workers spinning at barriers and rayon's F32 GEMM both waste CPU. The
  `--cpu-threads N` CLI flag on `pr/18-cpu-threads-flag` sets both env vars before
  `init_global_threadpool()`. End-to-end: 189 ms vs 427 ms per token on that build.

  This is the same class of problem as the upstream GEMV scaling issue below.

`CANDLE_NUM_THREADS` is read when Candle's barrier pool is first created, so it must be set
in the environment before the process starts. There is no in-repo knob that reconfigures
the pool after creation. `mistralrs-core/src/lib.rs` and
`attention/backends/cpu/threading.rs` only *read* `get_num_threads()`.

## Upstream defects already identified

- `huggingface/candle#1103` and `sarah-ek/gemm#20`: m=1 CPU GEMV does not scale with thread
  count; open since 2023. Fix written in `sarah-ek/gemm` PR #41. Evidence and aarch64 numbers
  are in `cpu-decode-bandwidth:todo/candle/01-cpu-decode-gemv-single-threaded.md`. Do not
  refile.
- aarch64 packed MoE GEMV is serial: `candle-core::quantized::repack::try_indexed_gemv` has
  zero parallel constructs at both pinned `66a8cf1` and upstream `12bb5d1`. A bit-exact
  partitioning example for the *unquantized* stacked gather lives in
  `mistralrs-quant/src/moe_gather_cpu.rs` on `cpu-decode-bandwidth`. See
  `cpu-decode-bandwidth:todo/candle/03-aarch64-packed-moe-gemv-serial.md`.
- GGUF MoE dequantizing every expert on every call was fixed and made bit-exact on
  `cpu-decode-bandwidth`; see `cpu-decode-bandwidth:todo/mistralrs/02-cpu-gguf-moe-dequant-every-call.md`
  and PRs #2509 / #2558. Do not redo that work.

## Other measured context

- Host governor is `schedutil`, which holds 1.0 GHz out of 3.4 GHz. `performance` is for
  top-speed measurement only and must be restored afterwards. All Olala numbers so far are
  1.0 GHz numbers.
- The OVH llama.cpp fork on this box: ~12.5 tok/s decode, ~29.3 tok/s prefill. That is the
  gap to close, measured at the same governor.
- Olala with NEON + default 128 threads: ~438 ms/tok decode at depth 4. With 16-32 threads:
  ~225 ms/tok. The fork's ~80 ms/tok still leads by ~2.8x.
- The remaining gap is mostly many small ops per token and the upstream serial/badly
  partitioned kernels, not a missing Olala-specific kernel. The NEON Mamba scan itself is
  only ~2% of the 16-thread profile.

## Doing a new perf run

Record all of: git revision printed by the binary, `CANDLE_NUM_THREADS`, governor,
`--prompt-len`, `--gen-len`, `--depth`. Use `perf record -F 199 -g` and
`perf report --children`; at 128 workers the self-cost view is useless because every worker
gets ~0.7% and the real kernels are inlined into closures.
