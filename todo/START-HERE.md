# Start here

Before touching Olala CPU performance:

1. Read `PERF-LEVERS.md`. Thread count and work partitioning were already measured twice;
   the 2.2x result is a re-measurement, not a new finding.
2. Check `BRANCHES.md` for an existing fix. `cpu-decode-bandwidth` already has parallel
   gather, attention sinks, IQ NEON, and Q6_K/Q3_K/Q5_K expert kernels. Port from there
   instead of rewriting.
3. Record the governor. Default `schedutil` holds 1.0 GHz; `performance` is a deliberate,
   temporary switch and must be restored after the run.
4. Record `CANDLE_NUM_THREADS` / `RAYON_NUM_THREADS` used for any benchmark, together with
   `--depth` and prompt length. Without those the number is not comparable.
5. After a perf change, run the numerical harnesses on `pr/17-olala-cpu-kernels`
   (`cargo test -p mistralrs-core --lib olala`) before benchmarking. Hand-written f32
   drifts from Candle's tensor path; the harness is the guard.

## Known traps

- `OLALA_TENSOR_MAMBA=1` used to crash with a shape mismatch (`k_state.zeros_like()` instead
  of `s_state.broadcast_mul(&alpha_b)`); fixed in `0bedd5627` on `pr/17-olala-cpu-kernels`.
  If the tensor fallback crashes again, diff it against the oracle in
  `olala_cpu.rs::tensor_mamba_step` first.
- A shared `target/` directory with the Inkling-2 worktree can serve a stale binary that
  reports `Unknown normal-model GGUF architecture 'olala'`. Check the printed git revision
  in the benchmark output before trusting a result.
- The scalar fused Mamba kernel and the closed-form prefill are parity/measurement harnesses,
  not shipping paths. The scalar one is slower in release; the closed-form one is exact but
  ~8x slower. Do not enable either by default.
