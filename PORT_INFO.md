# Information on Nautilus Trader dYdX Port

This repository `nautilus-trader-dydx` currently contains a minimal README documenting a proposed implementation plan.

## Implementation Plan
The README outlines a three-phase approach:
1. **Core Typed Transport Clients** – implement Rust clients for HTTP (REST), WebSocket, and gRPC with tests and benchmarks while keeping the existing Python adapter.
2. **Python Bridge & Market-Data Path** – expose Rust clients to Python via `pyo3` and route the `MarketDataClient` through the Rust implementation, transparently for users.
3. **Full Adapter in Rust** – re-implement `InstrumentProvider`, `DataClient`, and `ExecutionClient` in Rust, refactor order submission methods, and run latency and soak tests.

## References and Links
- The README indicates that this repository integrates dYdX with Nautilus Trader.
- Commit `84738a4` references a merge of pull request `#1` titled "Add basic implementation plan".
- Pull request [`#2610`](https://github.com/nautechsystems/nautilus-trader/pull/2610) introduces a new OKX port for Nautilus Trader.
- The commit history does not reference other repositories or PRs with additional details.
- There are no other files or documentation in this repository at this time.

## Notes on NR Repository Links
The user request mentions "links to NR repo" with details on PRs and files. No remote repository is configured (`git remote -v` shows none), and no additional references exist in commit messages or documentation. As a result, detailed links to PRs or files in an external NR repository could not be provided from the data available within this repository.

