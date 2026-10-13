# Lab anatomy

`<…>` = fill in. Worked example throughout: `latency-lab` (rpc-plus-plus,
branch `feat/latency-lab`). Commands shown for Rust; swap per `languages.md`.

```
<topic>-lab/
  <manifest>            # isolated package, minimum deps
  README.md
  CARD.md
  <lib>                 # scaffolding: stand-ins + fakes
  <drills dir>/exN_<failure_slug>.<ext>
  solutions/            # gitignored, never compiled
```

Root `.gitignore`: `/<topic>-lab/<build dir>` and `/<topic>-lab/solutions/`.

## README.md

````markdown
# <topic>-lab

<N> drills for <feature> — <gap/spec refs> — done before it is written.
Plan: `<plan path>`.

Standalone <package> — <why the main build cannot see it>, so nothing here
reaches the main build, lockfile or CI.

## The shape of every file

**GIVEN:** the fakes, the counters, the clock, and every assert. The assert
messages say what a failure *means*, so read them — they are the teaching.
Tests marked "GIVEN, the trap" pass before you write anything: they show the
failure the blank exists to fix.

**BLANK:** one function, signature and contract given, body <marker>. That
function is the primitive.

```
<test cmd for exN>   # red: fails at the blank. Read the asserts.
#   ... fill the blank ...
<test cmd for exN>   # green: that is the grade
<demo cmd for exN>   # the demo — compare against your prediction
```

Two rules:

1. The `PREDICTION` block is written **before** the code. Every time.
2. Buzzer beats perfection. At the budget (in each file's header), write the
   `RESULT` block and move on — a drill that ended in a surprise is worth more
   than one polished into agreement.

Then add a line to `CARD.md` and commit.

## Order and budgets

| Drill | Budget | Blank | Owns |
|---|---|---|---|
| `ex1_<slug>` | <min> | `<fn>` | <failure mode, one line> |

**Never cut <exA> or <exB>** — <why>. Cut list if the day slips: <ordered>.

## What the drills say about `<src>/`

Read these after the drills, not before:

- `<RealType>` (`<path>`) <does X>. ExN is what that costs.

## Stuck?

`cat solutions/exN_*` — worked versions, gitignored and outside the compiled
tree. Read it, close it, **retype it from memory**. That last step is the
point; copy-paste teaches nothing.

`<lib>` is scaffolding, not lesson: `<Stand-in>` stands in for `<Real>`
(<same constants>), plus <departures>. `<Fake>` stands in for `<real call>`;
its behaviour can change mid-test.
````

## CARD.md

```markdown
# The card

Twenty lines, no more. If a rule does not fit, it was not the 20%. Fill one
block per exercise, in the exercise's own words — not the docs'.

## <theme> (Ex1, Ex7)

## <theme> (Ex2, Ex3)

## The loop (Ex8)
```

Group drills by idea, 4–6 headings, left empty.

## Scaffolding module header

```
Shared scaffolding for the drills. Nothing here is the lesson — it is the
stand-in for `<Real>` (`<path>`) and `<Real>` (`<path>`), so each exercise only
contains the primitive it is about.

One deliberate departure from `<src>/`: <what + which drill needs it>.
```

## Drill file (Rust shown; latency-lab ex3)

```rust
//! Ex3 — the fastest upstream is the broken one · budget 40 min · DO NOT CUT
//!
//! Buys: <what the real impl gets by knowing this — and the failure it avoids,
//! stated concretely with numbers>.
//!
//! PREDICTION (fill before you write code):
//! - A fails every call in 2ms. B succeeds every call in 200ms. Which one does
//!   ex1's `window_mean` rank first?
//! - 90 calls OK at 100ms, 10 failed at 1ms, the penalty is 3s. Score?
//! - `src/`'s `Snapshot` has `duration_micros_total` and nothing else on time.
//!   Can this score be computed from it? What field does the impl have to add?
//!
//! RESULT:
//! -
//!
//! Assert: see the tests. Run them red first and read the messages.

use <lab>::{...};

/// GIVEN, <earlier drill>'s answer: <the naive version, so the trap is visible>.
pub fn naive_mean(window: &Snapshot) -> Option<Duration> { window.lifetime_mean() }

/// The latency score of `window`, lower is better.
///
/// Contract:
/// - <rule>;
/// - <rule>;
/// - `None` for an empty window.
pub fn score(window: &Snapshot) -> Option<Duration> {
    // <one-line hint, never the answer>
    todo!("fill the blank")
}

fn <fixture>() -> Snapshot { /* GIVEN data */ }

fn main() { /* print naive vs blank, measured */ }

#[cfg(test)]
mod tests {
    #[test]
    fn a_fast_failure_does_not_win_the_head() {
        assert!(naive_mean(&broken) < naive_mean(&healthy),
            "GIVEN, the trap: by plain latency the 500-ing upstream is 100× better");
        assert!(score(&broken) > score(&healthy),
            "a call that failed in 2ms did not answer in 2ms — it never answered");
    }
    #[test]
    fn failures_cost_the_penalty_in_proportion() {
        // exact number, message shows the arithmetic
        assert_eq!(score(&s), Some(ms(390)),
            "(90×100ms + 10×3s) / 100 — the failures' own 1ms is thrown away");
    }
    // + edge cases: clean input, empty input
}
```

Test names are sentences stating the property. 3–5 tests per drill.

## Plan drills table (Phase 1)

```
| # | File | Budget | Blank | Owns failure mode |
|---|---|---|---|---|
| 1 | `ex1_window_mean` | 20 | `window_mean(cur, base) -> Option<Duration>`: diff total / diff count; None on 0 | ranking on lifetime avg — slow *now* still looks fast |
```
