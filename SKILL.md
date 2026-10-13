---
name: drill-lab
description: Scaffold a throwaway "lab" of GIVEN/BLANK drills that teach the primitives and failure modes of a feature before it is built — like tokio-lab, dedup-lab, latency-lab. Language-agnostic. Use when the user says "make a lab", "drills for X", "exercises for X", "a lab like tokio-lab", or wants to learn X before implementing it.
---

# drill-lab

A lab is a standalone package at the repo root, `<topic>-lab/`, isolated from the
main build. It holds N drills (usually 6–9). Each drill owns **one named failure
mode** and has **one blank**: a function with signature + contract given and
body left as the language's "not implemented" marker. Everything else — fakes,
clock, asserts — is GIVEN. Asserts teach: their messages say what a failure
*means*.

Templates: [references/anatomy.md](references/anatomy.md).
Per-language mechanics: [references/languages.md](references/languages.md).

## Phase 1 — Plan, then stop

1. Read the sources the user names (gap doc, spec, issue, the `src/` files the
   real impl will touch). Find what the real impl will need that the user has
   not done before: the mental models, and the ways each goes wrong.
2. Detect language, test runner, how to isolate a sub-package from the main
   build, and how to fake time. See `languages.md`; ask if not covered.
3. Find the repo's commit style (CLAUDE.md, `docs/commit-style.md`, git log) and
   whether earlier labs exist (`git branch -a | rg lab`). If they do, read one
   — match its wording and layout over this skill's templates.
4. Write a plan (plan mode if active), terse:
   - **Context** — gap/spec, where the impl will land, the new mental models.
     Goal: drills only on this branch; real impl later.
   - **Layout** — branch `<type>/<topic>-lab` off main; files; dependencies
     (minimum); gitignore lines.
   - **Drills table** — `# | file | budget (min) | blank signature + one-line
     contract | owns failure mode`. Order simple → end-to-end; last drill
     usually composes the earlier blanks into the real loop.
   - Total budget (default ≈ one day incl. setup + card). **Never cut** (the
     1–2 drills that own the core failure). **Cut list** in order.
   - Build steps, verification, unresolved questions.
5. Stop. Wait for approval.

## Phase 2 — Build

1. Branch. Scaffold the isolated package. Add the build dir and `solutions/` to
   the root `.gitignore`.
2. Scaffolding module (`lib`): stand-ins for the real types — same names, same
   constants, cite the real file paths — plus fakes whose behaviour can change
   mid-test and that count what happened (entered/finished/calls). Header says
   it is not the lesson and lists every deliberate departure from real code.
3. Drills, in the drill-file template. Each:
   - header: `ExN — <title> · budget M min [· DO NOT CUT]`, **Buys:**,
     **PREDICTION** (≈3 concrete questions with numeric/yes-no answers; at
     least one about the real codebase), empty **RESULT:**;
   - one blank (two or three only when the primitive truly is a pair);
     contract as bullet doc; at most a one-line hint, never the answer;
   - a `main`/demo that prints measured numbers to check the prediction;
   - tests: at least one **"GIVEN, the trap"** assert that passes before the
     blank is filled, showing the failure; the rest grade the blank.
   - deterministic: fake/paused clock, fixed seeds, no real network.
4. `solutions/exN_*` — worked version of every blank (full file), outside the
   compiled tree.
5. `README.md` and `CARD.md` from the templates.

## Phase 3 — Verify, then stop

- Each drill fails at its blank (red), and only there — no compile errors.
- Copy the lab to the scratchpad, overlay `solutions/`, run all tests: all
  green. Report pass count.
- Run each demo; numbers match the README/PREDICTION story.
- Main project build/test untouched.
- Stop for review. Propose a commit message in the repo's style (fallback:
  `🏗️ test(tests): add <topic>-lab, <n> drills for <subject>`). Do not commit
  unless the user asks.

## Rules

- A drill without a named failure mode is a tutorial — cut it.
- Trap first: the student should see the wrong answer pass before writing the
  right one.
- Assert messages are the teaching. `"expected 390ms"` is weak;
  `"(90×100ms + 10×3s)/100 — the failures' own 1ms is thrown away"` is the bar.
- Mirror the real codebase's vocabulary so the lab maps 1:1 onto the later impl.
- README's "What the drills say about `src/`" names, per drill, the line in the
  real code it exposes — to be read after the drills.
