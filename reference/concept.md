# Project Ledger — the concept

A lightweight project operating system. A small set of living documents, each with
exactly one job, plus a session cadence that keeps them honest. Publishing is **a
per-project choice, not a built-in**: a mirror adapter (`none` for local-only, the
default; `atlassian` for Confluence + Jira; room for others) publishes the state
one-way where a team can read it. It was extracted from two projects (ConvMatchEng and
ConvMatchEng-Admin) that independently arrived at the same shape.

## The problems it solves

1. **Vision/reality blur.** On any roadmap or architecture diagram, a built node and
   a dreamed node render identically. Left alone, a project's story drifts ahead of
   its truth. The scoreboard exists to stop that — every node carries an honest
   status label so nobody mistakes `Planned` for `Live`.
2. **Cold-start amnesia.** A long build arc holds enormous working context — the
   deployed state, the gotchas, the open questions. When the conversation resets or a
   new person picks it up, that context is expensive to re-derive from commits. The
   ledger holds it durably.
3. **Invisible status.** Stakeholders shouldn't have to clone the repo to learn the
   roadmap or what shipped. When a publishing mirror adapter is configured, it pushes
   both the roadmap and the open items to where the team already looks (e.g. Confluence
   + Jira). Solo or local-only projects skip this — the files in the repo are enough.

## The invariants (the actual product)

The files are just paper. These disciplines are what make it work:

- **Status honesty.** Every capability is pinned to an honest label on a fixed
  spectrum: *proven-live → built-but-unproven → planned → speculative*. The exact
  words flex per project (`Live / Current / Planned` for deployed software;
  `Current / Built-mock / Planned / Research` for greenfield), but the spine is the
  same. Uncertain is marked uncertain, never optimistic.
- **Paired session cadence.** Work happens in sessions. Each opens with a **brief**
  (one bounded goal, intent before code) and closes with a **close-out**
  (disposition, evidence, what's next). The close-out reconciles the canonical files.
  The journal is append-only; the scoreboard and queue are mutable.
- **The Contract binds the agent.** A standing set of rules of engagement governs how
  any agent works on the project: stop on uncertainty (ask, never guess), no
  destructive action without an in-the-moment human yes, a senior-engineer quality bar,
  and the honest-status obligation above. Amended by appending dated entries, never
  rewritten. It's the cheap guardrail that ships in both tiers. So the rules are
  in-context at the start of every session — not just sitting in a doc someone has to
  remember to open — `bootstrap` wires a pointer to the Contract into the project's
  agent-context file (`CLAUDE.md` / `AGENTS.md`): a short section with the load-bearing
  rules in brief and a link to the full `<PREFIX>-CONTRACT.md`.

## The artifacts

The **Tier** column marks what `bootstrap` scaffolds — `both` (minimal + full) or
`full` only.

| Artifact | Role | Tier | Mutability | ID scheme |
|---|---|---|---|---|
| **Contract** | The rules of engagement for any agent — stop-on-uncertainty, no destructive action without a human yes, the senior-engineer quality bar, honest status. The governance layer. | both | Amend-only (dated) | — |
| **Frame** | What we're building and why — thesis, destination architecture, the editorial constraints. The stable north star. | full | Rare edits | — |
| **Scoreboard** (Build Status) | Where are we — every node pinned to a status, the session that touched it, what's next, plus the *live operational state* to carry between sittings. **The "start here" file.** | both | Every session | status labels |
| **Queue** (Open Items) | What's next — deferred scope, hardening, open questions, each with a stable ID and a **trigger** (the condition that should reactivate it). Plus a resolved trail. | both | Every session | `X-#`, `OQ-#` |
| **Variance Log** | What diverged from intent — `V-#` shipped/operational variances (a wrong default, an infra reality the plan missed) and `VAR-#` documentation concerns (a claim that's stale or best-match and could be mistaken for confirmed fact). | full | As it happens | `V-#`, `VAR-#` |
| **Bugs Log** | Defects found, reported, and fixed. Distinct from the variance log: a *defect was found and fixed*, not a design/doc divergence. New bugs under Open; fixed bugs move to Fixed keeping their ID. | full | Every session | `BUG-#` |
| **Runbook** | Operational procedures — how to deploy, reach live state, roll back, troubleshoot. | full | On infra change | — |
| **Testing Procedure** | How to run the tests, what harnesses exist, pass/fail criteria. | full | On test-setup change | — |
| **Research** | All project research — deep-dives, comparisons, explorations — in `docs/research/` with an index doc. Grows over time. | full | Grows | — |
| **Journal** (Briefs) | The session record — one `BRIEF-session-NN` (intent) paired with one `SESSION-NN-close-out` (outcome). The narrative spine of how the project actually went. | full | Append-only | session NN |
| **Index** | The README "Where are we?" block — the front door that points at all of the above in one place. | both | On bootstrap | — |

Optional 8th artifact: the **Digest** (`docs/digests/`) — per-window summaries for
stakeholder visibility. Generated on demand by `/ledger digest` (not on every close),
each one synthesizes what shipped and what's pending over a time window from the session
close-outs and the resolved queue. Full tier (it reads the journal); local-first, then
mirrored if a publishing adapter is configured.

Optional extra: a **context handoff** for cold-starting a fresh conversation, when the
working context outgrows the scoreboard's live-state block. Fold it into the
scoreboard until it earns its own file.

## The ID schemes

Stable IDs are load-bearing — they let a trigger or a cross-reference survive across
sessions. Conventions:

- **Open items:** a short project prefix + number. `A-1` (admin work), `#73` (engine
  item), `OQ-1` (open question). Items move Open → Resolved but **keep their ID** and
  gain a one-line resolution. IDs are never recycled.
- **Variances:** `V-#` for operational variances, `VAR-#` for documentation concerns.
  Kept distinct on purpose — one is "the system surprised us," the other is "the docs
  might mislead someone."
- **Bugs:** `BUG-#` for defects found and fixed. Kept distinct from variances on
  purpose — a bug is "a defect was found and fixed," a variance is "the plan or system
  diverged from intent." Bugs move Open → Fixed but keep their ID. Never recycled.
- **Sessions:** zero-padded `NN`. A brief and its close-out share the number.

## The session loop

```
open  →  BRIEF-session-NN (bounded goal, eyeball pass)
          │
          ▼
        do the work
          │
          ▼
close →  SESSION-NN-close-out (disposition + evidence + next)
          │  reconcile: scoreboard · queue · variance log
          ▼
publish →  Confluence (hub + children + session log) + Jira (open items)
```

## The mirror (adapter-based)

Git is always the source of truth. Publishing is a **per-project adapter** (`none` for
local-only, `atlassian` for Confluence + Jira, room for others) chosen at bootstrap and
stored in `.ledger/ledger.json`. With `none`, the local files are the whole ledger. When
a publishing adapter is configured the mirror is generated, **one-way**, refreshed on
close, and never read back — keeping the discipline where the work happens and avoiding
two-master drift. The mapping a publishing adapter implements:

- **Roadmap** (the forward view) = the scoreboard's `Planned` nodes + the open-items
  queue + "what's next." Published as the *Build Status* and *Roadmap & Open Items*
  pages, and mirrored to a tracker (e.g. Jira) as issues.
- **Work done** (the backward view) = the session close-outs + the resolved trail +
  the proven-trail. Published as the *Session Log* pages.

Idempotency comes from a committed sync map (`.ledger/ledger.json`) holding each
page's ID and each open item's key under `mirror`. First publish creates; every publish
after updates by ID. No duplicates. Adapter contract + per-adapter procedure in
`adapters/`.

## Maintenance rules

- Update the scoreboard when a node changes status, when something is
  deployed/verified, or when a capability lands. Keep `Live` honest.
- Give every open item a **trigger**, not just a description — the condition that
  should pull it back onto the active path.
- When something diverges, log the variance the same session, while the gap is fresh.
- Reconcile and publish at close, not "later." The mirror is only useful if it's
  current.
