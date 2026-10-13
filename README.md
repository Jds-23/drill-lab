# drill-lab

A [Claude Code](https://claude.com/claude-code) skill that scaffolds a throwaway **lab** of fill-in-the-blank drills, so you learn the primitives and failure modes of a feature *before* you build it.

Ask for "a lab for retry backoff" and Claude reads your spec and code, plans 6–9 drills, and builds them as a standalone package (`<topic>-lab/`) that stays out of your main build. Each drill has one named failure mode and one blank function to fill in. Works with Rust, TypeScript, Python and Go, and with other languages if you tell it how.

## Install

Clone into your personal skills directory to use it in every project:

```bash
git clone https://github.com/Jds-23/drill-lab.git ~/.claude/skills/drill-lab
```

Or install it for one repo only:

```bash
git clone https://github.com/Jds-23/drill-lab.git .claude/skills/drill-lab
```

Restart Claude Code (or start a new session). `drill-lab` should now appear in the skill list.

To update later:

```bash
git -C ~/.claude/skills/drill-lab pull
```

## Usage

In a Claude Code session inside your project, invoke it directly:

```
/drill-lab retry with exponential backoff — see docs/gaps/retry.md
```

Or just ask. Claude loads the skill when you say things like:

- "make a lab for the dedup cache"
- "drills for request hedging before I implement it"
- "a lab like tokio-lab for connection pooling"

> [!TIP]
> Point it at the sources the real implementation will touch: a gap doc, spec, issue, or `src/` files. Drills mirror your codebase's names, so the lab maps 1:1 onto the later implementation.

### What happens

1. **Plan, then stop.** Claude reads your sources, finds your language, test runner and commit style, and writes a plan: a table of drills (file, time budget, blank signature, failure mode it owns), a total budget, and a cut list. It waits for your approval.
2. **Build.** On a new branch `<type>/<topic>-lab`, it creates the isolated package: a scaffolding module with fakes, one file per drill, worked `solutions/` (gitignored), a `README.md` and a `CARD.md`.
3. **Verify, then stop.** Each drill must fail only at its blank. Every drill must pass with `solutions/` copied over it in a scratch dir. The main build must be untouched. Claude suggests a commit message and does not commit unless you ask.

### Working a drill

Every drill file has the same shape:

- **GIVEN:** fakes, counters, fake clock, and every assert. Assert messages explain what a failure *means*. Tests marked "GIVEN, the trap" already pass and show the bug.
- **BLANK:** one function with signature and contract, body left as `todo!()` / `throw new Error(...)` / `NotImplementedError` / `panic(...)`.
- **PREDICTION:** about three concrete questions you answer *before* coding. Then a **RESULT** block you fill in after.

Rust example (the generated lab README lists commands for your language):

```bash
cargo test --bin ex1_<slug>   # red: fails at the blank, read the asserts
# ... fill the blank ...
cargo test --bin ex1_<slug>   # green
cargo run  --bin ex1_<slug>   # demo: compare numbers to your prediction
```

> [!NOTE]
> Stop at each drill's time budget, even if the drill isn't finished. Write RESULT and move on. A drill that ended in a surprise teaches more than one polished until it agrees with you.

## Supported languages

| Language | Isolation | Run one drill |
|---|---|---|
| Rust | standalone crate | `cargo test --bin exN_*` |
| TypeScript | own `package.json` + `tsconfig` | `npx vitest run exN` |
| Python | own `pyproject.toml` | `pytest exN_*.py` |
| Go | own `go.mod` | `go test ./exN_*` |

For any other language, Claude asks how to isolate the package, mark the blank, run one test, and fake time.

## Repo layout

```
SKILL.md                 # the skill: phases and rules
references/anatomy.md    # templates for lab README, CARD, lib and drill files
references/languages.md  # per-language mechanics
```
