# Notifier: google-chat

Posts a short update to Google Chat on `digest` and/or `close`. Selected by
`notify.google-chat` in `.ledger/ledger.json`. Honors the notifier contract in
`notifiers/README.md`.

## Config

```json
"notify": { "google-chat": {
  "enabled": true,
  "webhookEnvVar": "LEDGER_GOOGLE_CHAT_WEBHOOK",
  "events": ["digest", "close"],
  "space": "Project Updates"
}}
```

## When it fires

On a successful `digest` or `close` whose name is in `notify.google-chat.events`,
**after** the local files are written and the mirror (if any) has published. Never
before the source-of-truth write. If `enabled` is false or `events` doesn't list the
event, do nothing.

## Secret (never committed)

Read the webhook URL from the env var named by `notify.google-chat.webhookEnvVar`
(default `LEDGER_GOOGLE_CHAT_WEBHOOK`). If it's unset, **skip and fail soft**: report
`DONE_WITH_CONCERNS` with "Google Chat notify skipped: set
`LEDGER_GOOGLE_CHAT_WEBHOOK`", and continue. Never read the URL from `ledger.json`,
never echo it. (The whole webhook URL is the secret — it embeds the `key` and `token`
query params.)

## Message

Plain-`text` payload (Google Chat incoming webhook). Keep it short — the canonical
detail lives in the digest / close-out (and the mirror page, if published). End with
the same 🟢/🟡/🔴 status signal the mode reports.

**Google Chat is not Slack — two format differences that matter:**

- **Emoji must be literal Unicode.** Google Chat does **not** render Slack colon
  shortcodes (`:scroll:`, `:white_check_mark:`) — they post as literal text. Use the
  Unicode glyph directly (📜, ✅). The 🟢/🟡/🔴 status signal is already Unicode, so it
  carries over unchanged.
- **Bold is `*text*`** (single asterisk), like Slack's mrkdwn here — so `*<Project>*`
  renders bold in both. `\n` newlines work.

**`digest` event:**

```
📜 *<Project>* digest <START> → <END>
<L> Live · <C> Current (unproven) · <M> resolved
Top pending: <id> <title> — <trigger>
<link: the Confluence digest page URL if the mirror published one, else the local path>
🟢/🟡/🔴 <one-line status>
```

**`close` event:**

```
✅ *<Project>* session <NN> — <DONE | PARTIAL | BLOCKED>
<one-line outcome>
Next: <what's next>
🟢/🟡/🔴 <one-line status>
```

## Post — incoming webhook (portable default)

```bash
WEBHOOK="${LEDGER_GOOGLE_CHAT_WEBHOOK:-}"    # or the configured webhookEnvVar
if [ -z "$WEBHOOK" ]; then
  echo "Google Chat notify skipped: webhook env var unset"; exit 0   # fail soft, not an error
fi
# Build JSON with jq so the message is escaped correctly — never string-concat into JSON.
payload="$(jq -n --arg t "$MESSAGE" '{text: $t}')"
code="$(curl -sS -o /dev/null -w '%{http_code}' -X POST \
  -H 'Content-Type: application/json' --data "$payload" "$WEBHOOK")"
[ "$code" = "200" ] || echo "Google Chat notify: non-200 ($code) — soft failure, continuing"
```

`notify.google-chat.space` is **informational** (the Google Chat analog of Slack's
`channel`): an incoming webhook posts to the space it was created for. The space name
here is only a human label — the webhook URL is what routes.

## Multiple projects & spaces

One incoming webhook posts to exactly **one space**, fixed when it's created — so a
new project space needs a **new webhook**:

1. In the Google Chat space → space name → **Apps & integrations** → **Webhooks** →
   **Add webhook** → name it → copy the URL.
2. Give each project a **distinct** `webhookEnvVar` so several ledgers on one machine
   don't collide (e.g. `LEDGER_GOOGLE_CHAT_WEBHOOK_<PROJECT>`), and export the matching
   URL. `notify.google-chat.space` is only a human label.

**No lightweight dynamic-routing path (unlike Slack).** Slack scales past a handful of
projects with one `chat:write` bot token routed by channel. Google Chat's equivalent —
posting to arbitrary spaces from one credential — needs a **Chat app backed by a Google
Cloud service account** (create a GCP project, enable the Chat API, publish the app, add
it to each space). That's much heavier than a per-space webhook and is **out of scope**
for this notifier. For many projects, prefer one webhook + one env var per space; reach
for a Chat app only if that per-space overhead becomes real pain.

## Idempotency

None — re-running a `digest` or `close` re-posts. Expected for a notifier (a stream, not
a mirror). The operator controls cadence by choosing when to run the mode.
