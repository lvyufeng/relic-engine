# relic-engine

**A frozen archive of the retired C++ inference engine.** It was orphaned when the PocketLLM
monorepo was split into [PocketLLM](https://github.com/lvyufeng/PocketLLM) (single-card / edge),
[RelicLLM](https://github.com/lvyufeng/RelicLLM) (multi-GPU PyTorch runtime) and
[relic-core](https://github.com/lvyufeng/relic-core) (shared torch operator library). No repository
depends on this one, no CI runs here, and nothing is built, released or maintained.

## Language convention

**All Markdown and code comments are written in English.** Commit messages, docstrings and every
`.md` file. A Chinese version of a document is a separate file (`README.md` / `README_CN.md`), never
a mixed-language one.

## Why the tree is kept

It is not a museum piece — two capabilities still have work in progress against them, and this tree
holds the only copy of how they were done once:

- **The continuous-batching scheduler** (`cpp_engine/engine/batch_scheduler.cpp`,
  `engine/qwen_batch_scheduler.cpp`). RelicLLM issue #18 ports its loop to a pure-Python scheduler.
- **The quantized KV cache** (`cpp_engine/include/block_pool.hpp`, `block_table.hpp`, and the
  multi-slot KV layout in `qwen_batch_scheduler.cpp`). RelicLLM issue #11 makes the torch KV dtype
  selectable — FP8 and TurboQuantK8V4 — and this is where that was solved.

It is also the measurement record for the Ascend 910B work: `docs/performance/ascend_*.md` and
`docs/archive/phase2-phase3/*` report numbers taken through this engine.

## The two invariants it was built to hold

Recorded because they outlive the engine and the live repositories still cite them:

- **Kernels stay behind the C ABI.** `cpp_engine/include/cuda_ops.hpp` declares **93 ops, 92 of
  which take a `void* stream`**, and includes only `<cstddef>` and `<cstdint>` — **no CUDA header at
  all**. `cpp_engine/engine/` and `cpp_engine/core/` likewise contain **zero `<<<` launches**; all
  **371** are under `cpp_engine/backends/`.

  *(Measured on this tree, 2026-10-01. The monorepo's `CLAUDE.md` gives 93/91 ops and 359 launches;
  the op count agrees, the other two do not — this tree is the authority for what it contains.)*
- **Backend selection happens at build time.** Per-hardware tuning was not traded away for
  portability: the 2080 Ti and 910B paths are separate code, not runtime switches.

## Building it

**Do not expect this to build, and do not fix it if it does not.** The toolchain was a CUDA
11.8/12.4/13.0 host with `CMAKE_CUDA_ARCHITECTURES=75`; `cpp_engine/README.md` has the original
instructions. The Ascend path required `source scripts/ascend_env.sh` first — an ACL binary launched
without CANN's `set_env.sh` does not fail, it **hangs** before `aclInit` returns.

## Known dangling links

`docs/` here is a subset of the monorepo's, so relative links that pointed at pages which stayed
behind do not resolve: **23 links across 10 pages**. They are left as written on purpose. This is an
archive, and rewriting them would make the record claim a repository structure that did not exist
when the measurement was taken.

## Git workflow

**Never commit directly to `master`.** Every change goes on a branch and through a pull request —
the one exception being the archival bootstrap itself, which had to land before there was anything
to review.

Branch prefixes: `feature/`, `fix/`, `refactor/`, `docs/`, `perf/` — `<prefix>/<description>`.

Commits: a one-line summary under 72 characters, a blank line, then the explanation starting on
line 3. **Every commit message ends with:**

```
Co-Authored-By: Claude Code <noreply@anthropic.com>
```

PRs: a title under 72 characters, a body covering summary, implementation details and testing
status, **one concern per PR**, and the body ends with:

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

Merged branches are not reliably deleted on `origin`; delete yours locally and remotely.

## These facts are per-host

The toolchain and checkpoint paths quoted anywhere in this archive were true of one x86_64 box with
4 × RTX 2080 Ti, and of one aarch64 box with 8 × Ascend 910B. Neither is a universal claim. The CUDA
path does not exist on the Ascend host at all, and vice versa.

## Provenance

Extracted with `git-filter-repo --paths-from-file` from the PocketLLM monorepo: **381 commits**,
history preserved from the first commit that touched `cpp_engine/`, with the tree **byte-identical**
to the monorepo at `1f22016`. Kept alongside it are the 22 `scripts/` entries whose only subject was
the binary this engine built, and the 36 `docs/` pages that record its architecture and behaviour.