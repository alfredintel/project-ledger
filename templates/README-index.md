<!-- Paste this block into the project README. It is the front door of the ledger.
     Keep only the rows whose artifact exists in this project's tier: minimal has
     Contract + Scoreboard + Work queue + this index; full has all rows below. Drop
     any row whose file you didn't scaffold so the front door has no dead links. -->

## Where are we? (project tracking)

This project runs on the **Project Ledger** discipline:

- **Contract** — [`{{PREFIX}}-CONTRACT.md`](./{{PREFIX}}-CONTRACT.md): the rules of engagement for any agent working here. Read first.
- **Scoreboard** — [`docs/{{PREFIX}}_BUILD_STATUS.md`](./docs/{{PREFIX}}_BUILD_STATUS.md): every capability, its honest status ({{STATUS_VOCAB}}), and live operational state. **Start here.**
- **Work queue** — [`{{PREFIX}}_OPEN_ITEMS.md`](./{{PREFIX}}_OPEN_ITEMS.md): open items ({{X}}-#) + resolved trail.
- **Variance log** — [`{{PREFIX}}_VARIANCE_LOG.md`](./{{PREFIX}}_VARIANCE_LOG.md): what diverged (V-# / VAR-#).
- **Bugs log** — [`docs/bugs/bugs_log.md`](./docs/bugs/bugs_log.md): defects found → fixed (BUG-#).
- **Runbook** — [`docs/deploy/deploy_runbook.md`](./docs/deploy/deploy_runbook.md): deploy, access, roll back, troubleshoot.
- **Testing** — [`docs/testing/testing_procedure.md`](./docs/testing/testing_procedure.md): how to run the tests + pass/fail.
- **Research** — [`docs/research/`](./docs/research/): deep-dives, comparisons, explorations.
- **Briefs** — [`docs/briefs/`](./docs/briefs/): per-session intent (`BRIEF-session-NN`) + outcome (`SESSION-NN-close-out`).
- **Frame** — [`{{PREFIX}}-OVERVIEW.md`](./{{PREFIX}}-OVERVIEW.md): what we're building and why.

Maintained with `/ledger` — `open` a session, `close` to reconcile (and, if a mirror
adapter is configured, publish). Published mirror (when configured): Confluence space
**{{SPACE}}** (hub + Contract + Build Status + Roadmap + Variance + Runbook + Testing +
Bugs + Research + Session Log) · open items and bugs tracked in Jira project **{{JIRA}}**.
