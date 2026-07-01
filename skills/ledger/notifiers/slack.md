# Notifier: slack

Posts a short update to Slack on `digest` and/or `close`. Selected by `notify.slack` in
`.ledger/ledger.json`. Honors the notifier contract in `notifiers/README.md`.

## Config

```json
"notify": { "slack": {
  "enabled": true,
  "webhookEnvVar": "LEDGER_SLACK_WEBHOOK",
  "events": ["digest", "close"],
  "channel": "#project-updates"
}}
```

## When it fires

On a successful `digest` or `close` whose name is in `notify.slack.events`, **after** the
local files are written and the mirror (if any) has published. Never before the
source-of-truth write. If `enabled` is false or `events` doesn't list the event, do nothing.

## Secret (never committed)

Read the webhook URL from the env var named by `notify.slack.webhookEnvVar` (default
`LEDGER_SLACK_WEBHOOK`). If it's unset, **skip and fail soft**: report
`DONE_WITH_CONCERNS` with "Slack notify skipped: set `LEDGER_SLACK_WEBHOOK`", and continue.
Never read the URL from `ledger.json`, never echo it.

## Message

Plain-`text` payload (Slack incoming webhook). Keep it short — the canonical detail lives in
the digest / close-out (and the mirror page, if published). End with the same 🟢/🟡/🔴 status
signal the mode reports.

**`digest` event:**

```
:scroll: *<Project>* digest <START> → <END>
<L> Live · <C> Current (unproven) · <M> resolved
Top pending: <id> <title> — <trigger>
<link: the Confluence digest page URL if the mirror published one, else the local path>
🟢/🟡/🔴 <one-line status>
```

**`close` event:**

```
:white_check_mark: *<Project>* session <NN> — <DONE | PARTIAL | BLOCKED>
<one-line outcome>
Next: <what's next>
🟢/🟡/🔴 <one-line status>
```

## Post — incoming webhook (portable default)

```bash
WEBHOOK="${LEDGER_SLACK_WEBHOOK:-}"        # or the configured webhookEnvVar
if [ -z "$WEBHOOK" ]; then
  echo "Slack notify skipped: webhook env var unset"; exit 0   # fail soft, not an error
fi
# Build JSON with jq so the message is escaped correctly — never string-concat into JSON.
payload="$(jq -n --arg t "$MESSAGE" '{text: $t}')"
code="$(curl -sS -o /dev/null -w '%{http_code}' -X POST \
  -H 'Content-type: application/json' --data "$payload" "$WEBHOOK")"
[ "$code" = "200" ] || echo "Slack notify: non-200 ($code) — soft failure, continuing"
```

`notify.slack.channel` is **informational**: an incoming webhook posts to the channel it
was created for. To target a channel dynamically, post via a connected Slack MCP server's
send-message tool to `notify.slack.channel` instead of the webhook — same message, same
fail-soft rule.

## Multiple projects & channels

One incoming webhook posts to exactly **one channel**, fixed when it's created — so a new
project channel needs a **new webhook, not a new Slack app**. A single app holds many
webhooks:

1. Create the channel in Slack.
2. api.slack.com/apps → your app → **Incoming Webhooks** → **Add New Webhook to Workspace**
   → pick the channel → copy the URL.
3. Give each project a **distinct** `webhookEnvVar` so several ledgers on one machine don't
   collide (e.g. `LEDGER_SLACK_WEBHOOK_<PROJECT>`), and export the matching URL.
   `notify.slack.channel` is only a human label — the webhook URL is what routes.

**Scaling past a handful of projects — switch to a bot token.** Instead of one webhook per
channel, use a Slack app with a `chat:write` bot token (or a connected Slack MCP server):
one token, unlimited channels, and `notify.slack.channel` becomes the real target (routed
dynamically, not fixed at creation). The bot must be invited to each channel
(`/invite @<app>`). Same message, same fail-soft rule.

| | Incoming webhook (default) | Bot token / MCP |
|---|---|---|
| Per new project | add a webhook + set its env var | `/invite` the bot to the channel |
| Secrets to manage | one URL per project | one token total |
| Channel routing | fixed at creation | dynamic via `notify.slack.channel` |
| Best for | 1–5 projects | many projects |

## Idempotency

None — re-running a `digest` or `close` re-posts. Expected for a notifier (a stream, not a
mirror). The operator controls cadence by choosing when to run the mode.
