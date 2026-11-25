# todo: CPU perf state

Start here before doing CPU perf work on Olala (or any other model). This branch carries
only notes; it is based on `master` and has no code changes.

The point of this directory is to stop re-deriving known results. Before measuring anything,
read `PERF-LEVERS.md`. Before starting a branch, read `BRANCHES.md` and check whether a fix
already exists on one of them.

## Files

| File | What it holds |
|---|---|
| `START-HERE.md` | Checklist to run before starting perf work |
| `BRANCHES.md` | Every local branch and open PR: tip, content, folded-in status |
| `PERF-LEVERS.md` | Known perf levers, measured numbers, and upstream issue references |
| `OLALA.md` | Olala-specific PR split, parity constraints, and remaining work |

## Where the older notes live

Two older branches already have `todo/` trees with detailed evidence. Do not duplicate them:

- `cpu-decode-bandwidth:todo/` - Kolibri/Flint CPU work, candle issue drafts, oracle scripts.
- `docs/perf-record:todo/` - same tree plus the CPU decode performance record docs.

Those entries are about Kolibri-1 / Flint / IQ GGUF. This directory is the cross-branch index
plus the Olala-specific state that was missing when those were written.

## Status vocabulary

- `folded` - the work is in an open PR with a branch pushed to `fork`.
- `local only` - the commit exists on a local branch but is not in any PR.
- `superseded` - a later branch or PR replaced it; keep for history only.
- `open` - known problem, no fix landed.

## Host

All numbers in these files were measured on the aarch64 Ampere-1a box (128 cores, 4 NUMA
nodes, 1.0 GHz default governor, 3.4 GHz only under explicit `performance`). See
`cpu-decode-bandwidth:todo/evidence-host.md`. Numbers do not transfer to other hosts.

## Durable copy

The full Olala CPU experiment record is kept outside the repo (so it survives rebases and
force-pushes) at `/home/lu_zero/Sources/mistral-rs-experiments/olala/`. Start with its
`README.md`; it indexes all experiments and outcomes. This in-repo `todo/` tree stays as the
working index; update the external copy when the campaign ends.
