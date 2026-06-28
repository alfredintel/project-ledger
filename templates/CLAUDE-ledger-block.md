<!-- Paste this block into the project's agent-context file (CLAUDE.md / .claude/CLAUDE.md
     / AGENTS.md). It points Claude Code at the Contract before any work. `/ledger
     bootstrap` installs and updates it automatically — keep the section heading exactly
     "## Project Ledger — rules of engagement" so re-runs update it in place instead of
     duplicating. Do not paste the full Contract here; this is a pointer + the
     load-bearing rules in brief. -->

## Project Ledger — rules of engagement

This project is tracked with **Project Ledger** (`/ledger`). Before doing any work,
read the Contract at `{{PREFIX}}-CONTRACT.md` — the binding rules of engagement for this
repo. The load-bearing ones:

- **Stop on uncertainty.** If you lose context, are missing information, or are unsure
  of intent, STOP and ask the operator. Never proceed on an optimistic assumption.
- **No irreversible action without a human "yes."** Never delete or destroy servers,
  databases, or files, never force-push or rewrite shared history, and never take any
  irreversible action without an explicit, in-the-moment human confirmation.
- **Senior-engineer bar.** Hold all work to a senior engineer's standard: the
  architecture stays coherent, and status labels stay honest — uncertain is marked
  uncertain, never optimistic.
- **Ground in current docs.** For third-party APIs/SDKs or high-stakes flows (auth,
  billing, migrations, deploys), verify against current primary docs before acting —
  don't code from memory; name the source.

Autonomous work is opt-in (`/ledger open --autonomous`, "plow ahead"): proceed through
ordinary ambiguity only by logging stated assumptions in the ledger, and never waive the
no-irreversible-action rule. Full detail in the Contract.

Work happens in sessions: `/ledger open` before, `/ledger close` after. The scoreboard
(`docs/{{PREFIX}}_BUILD_STATUS.md`) is the source of truth for "where are we." See
`README.md` for the full ledger layout.
