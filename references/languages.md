# Per-language mechanics

| | Isolation | Drill file | Blank marker | Run one drill (test) | Demo | Fake time | Build dir to ignore |
|---|---|---|---|---|---|---|---|
| Rust | standalone crate; root `Cargo.toml` must have no `[workspace]` (else add `exclude`) | `src/bin/exN_*.rs`, tests in `#[cfg(test)]` | `todo!("fill the blank")` | `cargo test --bin exN_*` | `cargo run --bin exN_*` | tokio `test-util`, `#[tokio::test(start_paused = true)]` | `target` |
| TypeScript | own `package.json` + `tsconfig`; exclude dir from root tsconfig/workspaces | `src/exN_*.ts` + `exN_*.test.ts` (or in-source `import.meta.vitest`) | `throw new Error("fill the blank")` | `npx vitest run exN` | `npx tsx src/exN_*.ts` | `vi.useFakeTimers()` | `node_modules`, `dist` |
| Python | own `pyproject.toml`; exclude from root pytest `testpaths` | `exN_*.py`, tests as `test_*` fns in same file | `raise NotImplementedError("fill the blank")` | `pytest exN_*.py` | `python exN_*.py` | inject clock fn / `freezegun` / `asyncio` loop time | `.venv`, `__pycache__` |
| Go | own `go.mod` (separate module = isolated) | `exN_*/main.go` + `main_test.go` | `panic("fill the blank")` | `go test ./exN_*` | `go run ./exN_*` | inject `func() time.Time` / clock interface | none |

Other language: ask the user for the four things — isolation, blank marker,
per-drill test command, fake clock.

Verification with solutions: copy lab to scratchpad, overlay `solutions/` onto
the drill files, run the full suite there — never overwrite the blanks in the
repo.
