# Mirror adapter: `none` (local-only)

The default. No external mirror. The files in the repo are the entire ledger.

- **`close` / `sync` publish step:** no-op. Reconcile the local files (the Contract,
  scoreboard, queue, and — full tier — variance log, bugs log, runbook, testing
  procedure, research index), commit, done. Report what changed and where the
  scoreboard lives. All these files live in the repo; `none` just doesn't push them
  anywhere.
- **`status`:** fully local — read `docs/<PREFIX>_BUILD_STATUS.md` and the queue.
- **`digest`:** writes the local digest at `docs/digests/digest-<START>_to_<END>.md`
  and stops there — no Confluence page, no page tree. The local file is the digest.
- **No metadata header, no page tree, no Jira issues.**
- **Notifiers still fire.** `none` only means *no mirror*. If `notify.slack` is enabled,
  `digest` / `close` still ping Slack (with the local path as the link) — notifiers are
  independent of the adapter. See `notifiers/`.

Use this for solo projects, greenfield work, anything not published to a shared
workspace. Upgrade later by setting `mirror.adapter` to a publishing adapter and
running `/ledger sync`; the current local files become the first publish with no
rework.
