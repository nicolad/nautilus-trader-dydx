# ChatGPT Conversation Notes

These notes summarize key information preserved from a conversation about the `nautilus-trader-dydx` repository.

## Summary
- The README documents a three-phase plan for porting the dYdX adapter to Rust.
- Phase 1 implements typed clients for HTTP, WebSocket, and gRPC with tests.
- Phase 2 exposes these Rust clients to Python via `pyo3` and reroutes the market data path.
- Phase 3 re-implements all adapter components in Rust and refactors order submission.

## Existing Clients
The current Python-based adapter already includes HTTP, WebSocket, and gRPC clients. These will be replaced by strongly typed Rust equivalents.

## Repository Notes
- Commit `84738a4` merged PR `#1` titled "Add basic implementation plan".
- PR [`#2610`](https://github.com/nautechsystems/nautilus-trader/pull/2610) in the main Nautilus Trader repo introduces an OKX port that follows a similar approach.
- No remote repository is configured here (`git remote -v` outputs nothing), so no additional NR repository links are available.

These details expand on the existing documentation and capture the context discussed during the conversation.
