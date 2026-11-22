# llama.cpp Inkling-2 Mini state

Recorded 2026-11-22 so this is not re-derived.

## Local checkouts

- `/home/lu_zero/Sources/llama.cpp`: was at `80d2c0c2c` (442 commits behind), fast-forwarded
  to `e300919cc` (current upstream master) with Inkling-2 support included. The old tip is
  preserved on branch `backup/pre-inkling2-update`. CPU-only build in `build-cpu/`.
- `/tmp/opencode/llama.cpp-olala`: OVH fork, Olala oracle only, no Inkling-2. Leave it alone.
- `/tmp/opencode/inkling2-32694.patch`: PR #32694 conversion patch (8 commits), not needed
  after the fast-forward since #32694 is merged upstream.

## Merged fixes

- #32694 "Inkling 2 Mini" merged 2026-11-20.
- #32704 "re-share the loop slots after the scale pass" merged 2026-11-20 (NVFP4 per-expert
  scales on 2nd/3rd loop passes). The older README warning about PPL 130.43 is obsolete.

## Bug #32798 - tokenizer ignore_merges (already covered upstream)

`src/llama-vocab.cpp`, `tokenizer_pre == "inkling2"` branch omitted `ignore_merges = true`.
Verified locally with `llama-tokenize` on `Inkling-2-Mini-UD-IQ1_M.gguf`:

- unpatched: `[1719, 149983, 7, 9, 70747, 11, 197033, 86226, 70, 82, 8, 25]`
- patched:   `[1719, 149983, 7, 9, 70747, 11, 197033, 87550, 8, 25]` (matches HF)

Local fix committed as `8acc0f28e` on `fix/inkling2-ignore-merges` in the llama.cpp checkout.
Do NOT open a PR: **rinpatch already has #32799**, byte-identical one-line fix, open since
2026-11-22T10:21Z, paired with a ggml-org/vocabs PR adding inkling2 vocab tests.

## Bug #32778 - prompts over ~96 tokens give all-'?' output

OPEN, no fix PR found. Reproduces on CPU and CUDA, b12600/b12619. Threshold ~96 tokens.
Not the same as #32752 ("build mask in chunks"), which is a memory/speed PR.

## vllm.cpp side (peer reports)

Their Inkling-2 bug was in vllm.cpp's own GGUF reader, not llama.cpp: ggml type ids 3 (Q4_1)
and 7 (Q5_1) were missing from `FindGgmlTraits`, so a 20 GB APEX GGUF was rejected. Fixed
in-tree by two table rows plus a parse-only `inkling2_loop` config builder. No llama.cpp
patch exists on that side; do not look there for one.
