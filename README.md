# Scale testing a production PostgreSQL system

A study of where a live PostgreSQL 16 application breaks under roughly a hundred times its real
load — what failed, how it was diagnosed, what fixed it, and what did not.

**[Read the findings →](findings.md)**

---

## What it found

Three classes of scale failure, different in kind rather than degree — each detected by a different
instrument and remedied by a different change.

- **Memory** — a bulk-read path spilling 1,437 MB to temporary files, where the obvious remedy
  cannot work. Along the way: the fastest configuration measured had the *worst* cache hit ratio.
- **Statistics** — 7.6× degradation with no memory pressure involved at all.
- **Planner boundary — and this one runs backwards.** A parallel-only plan becomes available when a
  table crosses 8 MB, so the system gets *faster* as it grows across that threshold. The dangerous
  region is below the cliff, not above it.

## What it is really about

The fixes were the easy part. **Eight instrument designs failed before one worked, and four of them
produced plausible numbers that were wrong** — three reported a *saving* on an index insert, which
is physically impossible. One design passed three registered gates and five negative tests before a
null distribution falsified it.

Three sessions were lost to a gate rather than a protocol. The fix was deleting an assertion.

The longest section of the document is about how the numbers were made trustworthy, not what they
are. §8 turns that into a checklist for a system you did not build.

## Honest limits, up front

- **Production holds 238 completed workouts.** The test mirror holds 28,053. Every figure was
  measured at mirror scale and none describes production today — these are predictions about a
  system growing into them.
- **The volume is synthetic**, calibrated against distributions measured from 19 real users, one of
  whom accounts for 74% of activity.
- **Everything is single-connection.** No concurrency evidence at all.
- **One person's work on one system, with no reviewer.** The pre-registration and null-distribution
  discipline described in the document exists as a partial substitute for that.

The document has its own section on what it does not show, and it is deliberately long.

## About the source system

The application is a fitness tracker I built and run — TypeScript throughout, React front end,
Node/Express back end, PostgreSQL 16 on the raw driver with no ORM, live with real users since
January 2026.

**That repository is private.** The findings document is therefore written so nothing needs to be
taken on trust: every mechanism it claims is a property of PostgreSQL, verifiable independently
without access to the code. §7 lists them, along with the environment and the method, so the study
can be reproduced elsewhere. The repository can be opened to a named reviewer on request.

## Status

**Draft 8, 25 August 2026.** Section 5 previously described a migration plan that had not executed.
It has now shipped — both indexes were created against live traffic and adopted by the planner — and
§5 reports the result, including a correction to a claim that was already stale when draft 7 went
out.

The canonical copy lives in the private repository alongside the evidence it cites; this is a
published snapshot.
