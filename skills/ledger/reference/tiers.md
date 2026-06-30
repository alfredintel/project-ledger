# Tiers — minimal vs full

The ledger scales from a weekend project to a multi-quarter platform. Pick a tier at
`bootstrap` (stored as `tier` in `.ledger/ledger.json`); upgrade `minimal` → `full`
any time by scaffolding the missing artifacts.

## `minimal`

The irreducible core: **a governance contract + status honesty + a reactivatable backlog.**

- **Contract** (`<PREFIX>-CONTRACT.md`) — the rules of engagement for any agent. Cheap
  and safety-critical, so it ships in **both** tiers.
- **Scoreboard** (`docs/<PREFIX>_BUILD_STATUS.md`) — every node + honest status + live
  operational state. Start here.
- **Queue** (`<PREFIX>_OPEN_ITEMS.md`) — open items, each with a trigger; resolved trail.
- **Index** — the README "Where are we?" block.

No required session cadence, no variance log, no per-session journal, no
runbook/testing/bugs/research. Update the scoreboard and queue as work lands. Best for
small, solo, or short-arc projects where full ceremony would just get skipped — and a
skill you skip isn't deployed.

## `full`

Everything in `minimal`, plus the durable narrative, the divergence record, and the
operational artifacts:

- **Frame** (`<PREFIX>-OVERVIEW.md`) — what & why, the north star.
- **Variance Log** (`<PREFIX>_VARIANCE_LOG.md`) — `V-#` operational, `VAR-#` doc concerns.
- **Bugs Log** (`docs/bugs/bugs_log.md`) — defects found → fixed (`BUG-#`). Distinct
  from the variance log: defects, not design/doc divergences.
- **Runbook** (`docs/deploy/deploy_runbook.md`) — deploy, access live state, roll back,
  troubleshoot. Refreshed when deploy/infra reality changes.
- **Testing Procedure** (`docs/testing/testing_procedure.md`) — how to run the tests,
  harnesses, pass/fail. Refreshed when the test setup changes.
- **Research** (`docs/research/`) — deep-dives, comparisons, explorations, with an index
  doc. Grows over time.
- **Journal** — paired `BRIEF-session-NN` (intent) + `SESSION-NN-close-out` (outcome).
- **Paired session cadence** — `open` before work, `close` after. This is what cures
  cold-start amnesia on long build arcs.
- **Digest** (`docs/digests/`) — on-demand per-window stakeholder summaries via
  `/ledger digest`. Needs the journal, so it's full-tier only.

Best for platform work, long arcs, anything a stakeholder or a future teammate will
need to reconstruct.

Tier controls what `bootstrap` scaffolds and whether the session cadence is expected.
Both tiers work with any mirror adapter, including `none`.
