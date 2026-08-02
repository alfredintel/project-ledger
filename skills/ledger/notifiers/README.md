# Notifiers

A **notifier** pushes a short, human-facing update to where a team already pays attention
(Slack and Google Chat today; Discord/email are future siblings). It is **not a mirror** — it doesn't keep
durable, idempotent pages in sync. It **composes** with whatever `mirror.adapter` is set:
run `atlassian` (or `none`) for the canonical record *and* a notifier for the ping.
Configured per project under `notify` in `.ledger/ledger.json`.

## Available notifiers

| `notify.<name>` | File | What it does |
|---|---|---|
| `slack` | `slack.md` | Posts digest / close summaries to a Slack channel via incoming webhook. |
| `google-chat` | `google-chat.md` | Posts digest / close summaries to a Google Chat space via incoming webhook. |

## The notifier contract

Any notifier — the one above, or a future sibling — must honor these:

1. **Composes, never replaces.** Independent of `mirror.adapter`. A notification is *in
   addition to* (not instead of) the local files and any mirror.
2. **Secret from env, never committed.** Any token / webhook / URL is read from an env var
   named in config; the secret itself never lives in `.ledger/ledger.json` or the repo.
3. **Event-gated.** Fire only on the events listed in config (`digest`, `close`). An empty
   or absent list means the notifier is off.
4. **Fail soft.** If the secret is missing or the send fails, do **not** fail the digest or
   close — finish the local work, report `DONE_WITH_CONCERNS` (notification skipped), and
   say what to set. A ping is never worth losing the source-of-truth write.
5. **Not idempotent — it's a stream.** Re-running a digest/close re-posts. That's expected
   (a notifier is a ping, not a mirror). The operator controls cadence by choosing when to
   run.

## The `notify` block

```json
"notify": {
  "slack": {
    "enabled": true,
    "webhookEnvVar": "LEDGER_SLACK_WEBHOOK",
    "events": ["digest", "close"],
    "channel": "#project-updates"
  },
  "google-chat": {
    "enabled": false,
    "webhookEnvVar": "LEDGER_GOOGLE_CHAT_WEBHOOK",
    "events": ["digest", "close"],
    "space": "Project Updates"
  }
}
```

Each notifier is independent — enable **whichever a project uses** (both, either, or
neither). Optional and **off by default** (omit a block, or set `enabled: false`).
`webhookEnvVar` names the env var holding the secret (the URL itself is never stored here).
`events` is a subset of `["digest", "close"]`. The **informational label key differs per
notifier** — Slack uses `channel`, Google Chat uses `space`; both are human labels only
(the webhook URL routes). Full procedure per notifier in its own file.

## Adding a notifier

Write `notifiers/<name>.md` against the contract above, add a row here, and add a
`notify.<name>` block to a project's `.ledger/ledger.json`. The core skill calls every
configured, enabled notifier after the source-of-truth write on each gated event.
