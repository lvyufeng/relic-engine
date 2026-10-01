# relic-engine

**A frozen archive.** This repository is the C++ inference engine that the
[PocketLLM](https://github.com/lvyufeng/PocketLLM) monorepo grew, extracted verbatim when the
monorepo was split into three live repositories:

| Repository | What it is |
|---|---|
| [PocketLLM](https://github.com/lvyufeng/PocketLLM) | single-card / edge runtime |
| [RelicLLM](https://github.com/lvyufeng/RelicLLM) | multi-GPU PyTorch runtime and serving shell |
| [relic-core](https://github.com/lvyufeng/relic-core) | shared torch operator library (CUDA sm_75, CPU host ops) |
| **relic-engine** (this one) | the retired native C++ engine |

**Nothing here is built, released or maintained.** No repository depends on it and no CI runs. It is
kept because two pieces of it are still the reference for work in progress, and deleting the tree
would delete the only copy of them:

- **`engine/batch_scheduler.cpp`** — the continuous-batching scheduler. RelicLLM issue #18 is porting
  its loop to a pure-Python scheduler; the four torch backends still declare `supports_batch=False`.
- **The quantized KV cache** — `include/block_pool.hpp`, `include/block_table.hpp` and the
  multi-slot KV layout in `engine/qwen_batch_scheduler.cpp`. RelicLLM issue #11 is making the torch
  KV dtype selectable (FP8, TurboQuantK8V4) and this is where that was solved once.

It is also the measurement record for a body of Ascend 910B work: `docs/performance/ascend_*.md` and
`docs/archive/phase2-phase3/*` report numbers taken through this engine, and they are cited from the
new repositories' histories.

## What it contains

| Path | What it is |
|---|---|
| `cpp_engine/` | the engine — `core/`, `engine/`, `backends/{api,cuda,ascend}/`, `include/`, `python/`, `tools/`, `tests/`, `cmake/` |
| `cpp_engine/include/cuda_ops.hpp` | the C ABI: 93 ops, 91 of which take a `void* stream`, with no CUDA header included |
| `scripts/` | 22 scripts that drove the built binary (`cpp_engine/build/pocketllm_engine`, `cpp_engine/build-ascend/pocketllm_engine`) — the Qwen drafter/MTP/DSpark/DFlash2 benches, the Ascend ladder, the adapter-parity verifiers |
| `docs/` | 36 pages: the engine's own architecture and multi-backend plan, its Phase 2/3 build-out records, the Ascend profile, and the serving-latency / throughput / correctness results measured through it |

The two invariants the engine was built to hold, recorded here because they outlive it:

- **Kernels stay behind the C ABI.** `include/cuda_ops.hpp` includes no CUDA headers, and
  `cpp_engine/engine/` contains zero `<<<` launches — every one is under `backends/`.
- **Backend selection happens at build time.** Per-hardware tuning was not traded away for
  portability; 2080 Ti and 910B paths are separate code, not switches.

## Building it

It is not expected to build. The original instructions are `cpp_engine/README.md`, and the toolchain
was a CUDA 11.8/12.4/13.0 host with `CMAKE_CUDA_ARCHITECTURES=75`. The Ascend path needed
`source scripts/ascend_env.sh` first — an ACL binary launched without CANN's `set_env.sh` does not
fail, it hangs before `aclInit` returns.

## Known dangling links

`docs/` here is a subset of the monorepo's `docs/`, so relative links that pointed at pages which
stayed behind (or moved to another repository) do not resolve. **23 links across 10 pages** — mostly
the Ascend performance records pointing at their model guides, and `archive/phase2-phase3/index.md`.
They are left exactly as written rather than rewritten to URLs, because this is an archive: the text
should read as it did when the measurement was taken, and rewriting it would make the record claim a
repository structure that did not exist at the time.

## Provenance

Extracted with `git-filter-repo --paths` from the PocketLLM monorepo: 381 commits, history preserved
from the first commit that touched `cpp_engine/` through #500. The tree is byte-identical to the
monorepo at `1f22016`.