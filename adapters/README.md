# Mirror adapters

The ledger discipline is local and universal. **Publishing is an adapter** — a
per-project choice recorded in `.ledger/ledger.json` under `mirror.adapter`. The core
modes (`bootstrap`, `open`, `close`, `status`, `sync`) never name a specific
Confluence space, Jira project, or MCP server; they call whatever adapter the project
configured.

## Available adapters

| `mirror.adapter` | File | What it does |
|---|---|---|
| `none` | `none.md` | Local-only. Publish is a no-op; the files in the repo are the whole ledger. **The default.** |
| `atlassian` | `atlassian.md` | Mirrors to Confluence (hub + children + session log) and Jira (open items become issues). For repos in an Atlassian workspace. |

Default is `none`, so a fresh `/ledger bootstrap` works on any repo with zero external
setup. Opt into a publishing adapter only where stakeholders actually read it.

## The adapter contract

Any adapter — the two above, or a new one (Linear, GitHub Projects, Notion) — must
honor these:

1. **One-way.** Local files are the source of truth. Never read mirror state back into
   the files.
2. **Idempotent by stored ID.** Create-or-update by IDs persisted in `mirror`. First
   publish creates and records the ID; every publish after updates in place. No
   duplicates.
3. **Write IDs back after each create**, not just at the end, so a mid-run failure is
   resumable and never orphans a page/issue.
4. **Fail soft.** If the backend is unreachable, finish the local file reconciliation
   and report `DONE_WITH_CONCERNS` (publish deferred — rerun `/ledger sync`). Never
   leave files half-reconciled because publishing failed.
5. **Self-contained config.** Everything the adapter needs (server name, target IDs, ID
   maps) lives under `mirror` in `.ledger/ledger.json`. Nothing workspace-specific
   leaks into `SKILL.md`.

## Top-level `.ledger/ledger.json` keys

The `mirror` block lives inside the project's sync map. The other top-level keys:

```json
{
  "project": "<name>",
  "prefix": "<PREFIX>",
  "slug": "<slug>",
  "tier": "minimal | full",
  "session": 0,
  "statusVocab": ["Live", "Current", "Planned"],
  "commit": { "author": "<Name <email>>", "coauthor": "" },
  "digest": { "defaultWindow": "last 7 days" },
  "mirror": { "adapter": "none" }
}
```

- `tier` decides which artifacts `bootstrap` scaffolds (see `reference/tiers.md`).
- `commit` is **optional**. If present, commits use `commit.author` and append
  `commit.coauthor` only when it's non-empty. If the whole block is absent, the skill
  uses the repo's own configured git identity and injects no co-author. Nothing
  author-specific is baked into the skill.
- `digest` is **optional**. `defaultWindow` is the `--since` used when `/ledger digest`
  is run with no window (falls back to `"last 7 days"` if absent). Digest is a
  `full`-tier, on-demand mode — see `SKILL.md` → Mode: digest. (A future Slack notifier
  will add its own `notify` block, kept separate from `digest`.)
- `mirror` selects the publish adapter (below).

## The `mirror` block

`none`:

```json
"mirror": { "adapter": "none" }
```

`atlassian` (shape only; full schema in `atlassian.md`):

```json
"mirror": {
  "adapter": "atlassian",
  "mcpServer": "atlassian-reflex",
  "site": "reflexmedia.atlassian.net",
  "cloudId": "<resolved-once-and-cached>",
  "confluenceSpace": "ARDM",
  "jiraProject": "ARDM",
  "vertical": "Platform",
  "featureStatus": "In Development",
  "metadataHeader": true,
  "publishContract": true,
  "confluence": {
    "hub": "", "build_status": "", "roadmap": "", "variance": "",
    "contract": "", "runbook": "", "testing": "", "bugs": "",
    "research": "", "research_docs": {},
    "session_log": "", "sessions": {},
    "digest_hub": "", "digests": {}
  },
  "jira": {}
}
```

The example values are the Reflex/ARDM target — they are config, not part of the skill.
`confluence` has one key per published page; `research` / `research_docs` and
`digest_hub` / `digests` are parent-page + per-child sub-map pairs (both mirror the
`session_log` + `sessions` pattern — `digests` is keyed by `"<START>_to_<END>"` window).
Only keys whose source artifact exists get published, so a `minimal`-tier project leaves
most empty. The `jira` map covers both open-item IDs and `BUG-#` defect IDs. Full
per-page schema and semantics in `atlassian.md`.

## Adding an adapter

Write `adapters/<name>.md` describing its publish procedure against the contract above,
add a row to the table here, and set `mirror.adapter` to `<name>` in a project's
`.ledger/ledger.json`. The core skill needs no changes.
