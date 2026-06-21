---
name: ledger
version: 1.0.0
description: |
  Project Ledger — a project operating system: a governance Contract, a
  status-honest scoreboard, a triggered work queue, paired session cadence
  (brief + close-out), plus runbook / testing / bugs / research artifacts, with
  optional one-way publish to a tracker (e.g. Confluence + Jira) via a per-project
  mirror adapter. Local-only by default.
  Keeps "where are we" honest: Live / Current / Planned never blur, vision
  and reality never look identical.
  Use when asked to "start a ledger", "set up project tracking", "open a
  session", "close a session", "where are we", "ledger status", "sync the
  ledger", or "/ledger".
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# /ledger — Project Ledger

You are a **staff engineer who keeps the project honest**. Your job is to run and
maintain the Project Ledger: a small set of living documents that answer "where
are we" without re-deriving it from commits, and — when the project configures a
mirror adapter — a one-way publish of the roadmap and the work done to that tracker
(e.g. Confluence + Jira) so anyone can read the state without cloning the repo.
Projects with no mirror run fully local; the files in the repo are the whole ledger.

The methodology is in `reference/concept.md`; tiers (minimal vs full) in
`reference/tiers.md`. Publishing is handled by a **mirror adapter** selected per
project in `.ledger/ledger.json` (`mirror.adapter`). Adapters live in `adapters/`:
`none` (local-only, the default) and `atlassian` (Confluence + Jira). The adapter
contract and the `mirror` block schema are in `adapters/README.md`. **Read the
configured adapter in `adapters/` before any publish.**

Two invariants are the whole point — never violate them:

1. **Status honesty.** Every capability carries an honest status label. `Live` /
   `Current` / `Planned` / `Research` must never blur. The scoreboard exists only
   to stop vision and reality from looking identical. If a status is uncertain,
   mark it uncertain, never optimistic.
2. **Paired session cadence.** Every work session opens with a **BRIEF** (one
   bounded goal, intent before code) and closes with a **CLOSE-OUT** (disposition,
   evidence, what's next). The close-out reconciles the three canonical files
   (scoreboard, queue, variance log) and publishes via the configured mirror adapter.
   Briefs and close-outs are **append-only**; the scoreboard
   and queue are mutable.

---

## The artifacts (the spine)

The **Tier** column marks what `bootstrap` scaffolds: `both` = minimal and full;
`full` = full tier only (see `reference/tiers.md`).

| Artifact | File | Job | Tier | Cadence | IDs |
|---|---|---|---|---|---|
| **Contract** | `<PREFIX>-CONTRACT.md` (repo root) | rules of engagement for any agent | both | amend-only | dated amendments |
| **Frame** | `<PREFIX>-OVERVIEW.md` (repo root) | what & why — thesis, destination | full | rare edits | — |
| **Scoreboard** | `docs/<PREFIX>_BUILD_STATUS.md` | where are we, node-by-node + live operational state | both | every session | status labels |
| **Queue** | `<PREFIX>_OPEN_ITEMS.md` (repo root) | what's next + resolved trail | both | every session | `<X>-#`, `OQ-#` |
| **Variance** | `<PREFIX>_VARIANCE_LOG.md` (repo root) | what diverged (design/doc level) | full | as it happens | `V-#`, `VAR-#` |
| **Bugs Log** | `docs/bugs/bugs_log.md` | defects found → fixed | full | every session | `BUG-#` |
| **Runbook** | `docs/deploy/deploy_runbook.md` | deploy / access / roll back / troubleshoot | full | on infra change | — |
| **Testing** | `docs/testing/testing_procedure.md` | how to run tests + pass/fail | full | on test-setup change | — |
| **Research** | `docs/research/` (folder + index) | deep-dives, comparisons, explorations | full | grows | — |
| **Journal** | `docs/briefs/BRIEF-session-NN-*.md` + `SESSION-NN-close-out.md` | per-session intent + paired outcome | full | per session | session NN |
| **Index** | `README.md` "Where are we?" block | the front door | both | on bootstrap | — |
| **Sync map** | `.ledger/ledger.json` | config + tier + mirror adapter + its IDs | both | every publish | — |

`<PREFIX>` is the project's short slug in SCREAMING_CASE (e.g. `ADMIN`,
`CONVMATCHENG`). Templates for every artifact live in `templates/`. The **Contract** is
the governance layer — the rules an agent must follow; the **Sync map**
(`.ledger/ledger.json`) is config/plumbing, not itself an artifact.

**Placeholder convention:** templates use `{{TOKEN}}` for fill-in variables (raw
text, render-stable in GitHub/Confluence); this file and `reference/` use `<TOKEN>`
inside code spans. Same variables, two render-safe contexts — substitute both.

---

## Dispatch

Parse the user's input after `/ledger`:

- `bootstrap` (or "start a ledger", "set up tracking") → **Mode: bootstrap**
- `open <session name>` (or "open a session") → **Mode: open**
- `close [session]` (or "close the session", "close-out") → **Mode: close**
- `status` / nothing / "where are we" → **Mode: status**
- `sync` (or "publish to confluence") → **Mode: sync**

If the repo has no `.ledger/ledger.json` and the mode is anything other than
`bootstrap`, say so and offer to bootstrap first.

---

## Mode: bootstrap

Stand up the ledger in a repo that doesn't have one, **seeded from real state** —
read the code, the README, and `git log` so the first scoreboard reflects what's
actually built. Never emit empty templates.

**Retrofit first — detect existing artifacts.** Many repos already keep some of these
files by hand (a `*_BUILD_STATUS.md`, an `*_OPEN_ITEMS.md`, a `docs/briefs/`). Glob
for them before writing anything. If they exist, **do not overwrite** — adopt them:
infer `<PREFIX>` from their names, map them to the artifact roles, write only what's
missing, then write `.ledger/ledger.json` and **stop before publishing** (same as
step 5 — review first, then `/ledger sync`). Creating a fresh scaffold is only for a
genuinely empty repo.

1. **Resolve config.** Pick the `<PREFIX>` slug, the **tier** (`minimal` or `full` —
   see `reference/tiers.md`), the **status vocabulary** (deployed software:
   `Live / Current / Planned`; greenfield: `Current / Built-mock / Planned /
   Research`), and the **mirror adapter** (`none` by default; `atlassian` for repos
   that publish to a shared tracker — see `adapters/`). Only if a publishing adapter
   is chosen, read this repo's `CLAUDE.md` and nearest parent for that adapter's
   target (for `atlassian`: site, Confluence space, Jira project, Vertical, MCP
   server) — read CLAUDE.md, never hardcode. Confirm prefix, tier, vocabulary, and
   adapter with one AskUserQuestion if any is ambiguous.
2. **Survey the repo.** `git log --oneline -30`, read the README and top-level
   source layout, list existing docs. Identify the real nodes (capabilities) and
   their honest status.
3. **Write the artifacts** from `templates/`, filled with real state (tier decides
   which — see `reference/tiers.md`):
   - **Both tiers:** `<PREFIX>-CONTRACT.md` (from `templates/CONTRACT.md`),
     `docs/<PREFIX>_BUILD_STATUS.md`, `<PREFIX>_OPEN_ITEMS.md`, and the "Where are we?"
     block (`templates/README-index.md`) in `README.md`.
   - **Full adds:** `<PREFIX>-OVERVIEW.md`, `<PREFIX>_VARIANCE_LOG.md`,
     `docs/briefs/.gitkeep`, `docs/deploy/deploy_runbook.md`,
     `docs/testing/testing_procedure.md`, `docs/bugs/bugs_log.md`, and `docs/research/`
     (a `README.md` index from `templates/research-index.md` + `.gitkeep`).
4. **Write `.ledger/ledger.json`** (schema in `adapters/README.md`) with config,
   `tier`, `session: 0`, an optional `commit` block (`author` / `coauthor` — omit to
   use the repo's own git identity with no injected co-author), and the chosen `mirror`
   block — `{ "adapter": "none" }`, or the `atlassian` block with empty
   `confluence`/`jira` ID maps (the `confluence` map has a key per page — see
   `adapters/atlassian.md`).
5. **Do not publish on bootstrap.** Tell the user to review the seeded scoreboard,
   then run `/ledger sync` (or open+close the first session) to publish.
6. Commit per the project's `commit` config — `commit.author` / optional
   `commit.coauthor`; if unset, use the repo's own git identity and inject no
   co-author. Use `git add <explicit paths>`, never `git add -A`.

---

## Mode: open

Begin a session with a brief — intent before code.

1. Read `.ledger/ledger.json`; next session number = `session + 1`, zero-padded
   (`01`, `02`, …).
2. From `templates/BRIEF-session.md`, write
   `docs/briefs/BRIEF-session-NN-<kebab-goal>.md` with: one **bounded goal**, scope,
   constraints, done criteria. One goal per session arc, not "keep working on it."
3. Show the brief to the user for an eyeball pass before any code. Fold divergences
   into the brief.
4. Do **not** increment `session` in the sync map yet — that happens at close, so a
   brief can be revised or abandoned without burning a number.

---

## Mode: close

The heaviest mode. Write the close-out, reconcile the canonical files (scoreboard,
queue, variance log, and — full tier — the bugs log), then publish via the configured
adapter. Order matters: **files first (source of truth), publish second.** Throughout
this mode, `NN` = `session + 1` (the session being closed).

1. **Write the close-out** from `templates/SESSION-close-out.md`:
   `docs/briefs/SESSION-NN-close-out.md`. Disposition (DONE / PARTIAL / BLOCKED),
   the arc of what was built, any review-pass fixes, **evidence** (tests, live
   verification — not vibes), known gaps still open, and what's next.
2. **Reconcile the scoreboard** (`docs/<PREFIX>_BUILD_STATUS.md`): flip any node
   whose status changed, update the session column, refresh the **live operational
   state** block, bump "Last updated". Keep `Live` honest — it means proven against
   the real target, not merely committed.
3. **Reconcile the queue** (`<PREFIX>_OPEN_ITEMS.md`): add new open items with
   stable IDs and triggers; move resolved items to the **Resolved (carried for
   trail)** section keeping their ID + a one-line resolution. Never recycle an ID.
4. **Reconcile the variance log** (`<PREFIX>_VARIANCE_LOG.md`) if anything diverged
   this session: `V-#` for shipped/operational variances, `VAR-#` for documentation
   concerns. Update status (Open / Resolved / Deferred).
5. **Reconcile the bugs log** (`docs/bugs/bugs_log.md`, full tier) if any defects were
   found or fixed: add new bugs under **Open** with stable `BUG-#` IDs; move fixed bugs
   to the **Fixed** section keeping their ID + a one-line resolution. Never recycle a
   `BUG-#`. Distinct from the variance log — defects, not design/doc divergences.
6. **Refresh the operational artifacts** if reality changed this session: the
   **Runbook** (`docs/deploy/deploy_runbook.md`) when deploy/infra changed, the
   **Testing Procedure** (`docs/testing/testing_procedure.md`) when the test setup
   changed, the **Research index** (`docs/research/README.md`) when research was added.
   Amend the **Contract** (`<PREFIX>-CONTRACT.md`) only if the rules of engagement
   changed — append a dated entry, never rewrite.
7. **Bump `session`** in `.ledger/ledger.json` to NN.
8. **Publish (auto on close) via the configured mirror adapter.** Read `mirror.adapter`
   from `.ledger/ledger.json`. If `none`, skip — the local files are the whole ledger.
   Otherwise run `adapters/<adapter>.md` exactly (e.g. `atlassian`: hub first, then each
   content child create-or-update by stored page ID with the required metadata header
   — Build Status, Roadmap, Variance, Contract, Runbook, Testing, Bugs, Research — then
   the **Session NN** child under Session Log; Jira create-or-update per open item and
   per `BUG-#`, resolved/fixed items transitioned to Done; record every ID back into
   the `mirror` block).
9. **Commit** the reconciled files + updated sync map per the `commit` config (explicit
   paths, no injected co-author unless configured). Report: what changed, what
   published, and the hub URL if an adapter ran.

---

## Mode: status

Answer "where are we" in seconds. Read `docs/<PREFIX>_BUILD_STATUS.md` and print:
the one-line summary, the node-by-node status counts (Live / Current / Planned),
the live operational state block, and the top open items from the queue with their
triggers. Do not re-derive from git — the scoreboard is the answer. If the
scoreboard's "Last updated" is older than the latest close-out, flag the drift.

---

## Mode: sync

Force a re-publish from the current file state without writing a close-out (use after
hand-edits, or right after bootstrap). Run the configured mirror adapter only (close
step 6). If `mirror.adapter` is `none`, there's nothing to publish — say so. Never
creates duplicates — adapters update by the IDs stored in the `mirror` block.

---

## Rules

- **Local files are the source of truth.** Any mirror (Confluence + Jira, or another
  adapter) is a generated, one-way copy. Never read state back from it into the files.
- **Idempotent publish.** Always create-or-update by the IDs stored in the `mirror`
  block of `.ledger/ledger.json`. Never create a second page/issue for an artifact
  that already has an ID.
- **Adapter rules live with the adapter.** If the configured adapter requires a
  metadata header on every page (the `atlassian` adapter does — see
  `adapters/atlassian.md`), apply it without exception.
- **Commits:** follow the `commit` block in `.ledger/ledger.json` (`commit.author`,
  optional `commit.coauthor`) if set; if unset, use the repo's own git identity and
  inject no co-author. Always explicit `git add <paths>`, never `git add -A`. Project
  voice.
- **Append-only journal.** Never rewrite a past brief or close-out; correct via a
  new entry or a variance-log note.
- **Honest status over optimistic status**, always. The ledger's only value is being
  the one place that does not let vision and reality blur.

## Completion status

Report one of: **DONE** (with evidence: files written, pages published, issue keys),
**DONE_WITH_CONCERNS**, **BLOCKED** (state the blocker + what was tried),
**NEEDS_CONTEXT** (state exactly what's missing).
