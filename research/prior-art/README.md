# Prior-Art Studies

*Follows the study template in [`../README.md`](../README.md).*

| System | Primarily informs | Status |
|--------|-------------------|--------|
| [Kubernetes](kubernetes.md) | Resource model, controllers, API conventions (Books 02, 03, 07) | Accepted (exemplar) |
| [Linux](linux.md) | Microkernel vs. monolith trade-offs, syscall stability (Books 03, 00) | Accepted |
| [LLVM](llvm.md) | Two-level IR, pass framework, lowering (Books 04, 05) | Accepted |
| [MLIR](mlir.md) | Multi-dialect IR, progressive lowering (Book 04) | Accepted |
| [Temporal](temporal.md) | Durable execution, determinism, retries (Books 06, 07) | Accepted |
| [Git](git.md) | Content addressing, immutable history (Books 02, 04) | Accepted |
| [Terraform](terraform.md) | Desired-state, plan/apply, providers (Books 02, 06, 12) | Accepted |
| [Docker](docker.md) | Packaging, isolation, image distribution (Books 06, 12) | Accepted |
| [Cloudflare Workers](cloudflare-workers.md) | Runtime isolation, edge execution (Book 06) | Accepted |
| [PostgreSQL](postgresql.md) | Extensibility, query planner/optimizer, stability (Books 05, 12) | Accepted |
| [Rust / Cargo](rust-cargo.md) | RFC process, package ecosystem, stability guarantees (process, Book 12) | Accepted |

Each study is authored against the template. The Kubernetes study above is the exemplar establishing the expected depth. All eleven are `Accepted` — without independent review, and written after the specification they describe; see [Provenance and its limits](../README.md#provenance-and-its-limits) before relying on the set.
