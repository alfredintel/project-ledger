# Mirror adapter: `none` (local-only)

The default. No external mirror. The files in the repo are the entire ledger.

- **`close` / `sync` publish step:** no-op. Reconcile the local files (the Contract,
  scoreboard, queue, and — full tier — variance log, bugs log, runbook, testing
  procedure, research index), commit, done. Report what changed and where the
  scoreboard lives. All these files live in the repo; `none` just doesn't push them
  anywhere.
- **`status`:** fully local — read `docs/<PREFIX>_BUILD_STATUS.md` and the queue.
- **No metadata header, no page tree, no Jira issues.**

Use this for solo projects, greenfield work, anything not published to a shared
workspace. Upgrade later by setting `mirror.adapter` to a publishing adapter and
running `/ledger sync`; the current local files become the first publish with no
rework.
