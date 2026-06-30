# Project Ledger

A reusable **Claude Code skill** (`/ledger`) and the methodology behind it: a
lightweight project operating system that keeps "where are we" honest and, when a
project opts in, mirrors its state one-way to a tracker like Confluence and Jira.

This directory — `/Users/alin/Documents/PersonalCode/Skills/ledger` — is the
**canonical home** and the standalone git repo. It's packaged as a **Claude Code plugin**:
the skill itself lives in `skills/ledger/`, with the plugin + marketplace manifests in
`.claude-plugin/`. Install it via `/plugin` (see [Install](#install)), or symlink
`skills/ledger/` for live local development.

---

## What it is

Project Ledger is a small set of living documents — each with exactly one job —
plus a **session cadence** that keeps them honest and an optional **one-way mirror**
that publishes the state to a project tracker (e.g. Confluence + Jira), so anyone can
read where a project stands without cloning the repo. With no mirror it runs fully
local.

It was extracted from two internal projects that independently arrived at the same
shape.

### The problems it solves

1. **Vision/reality blur.** On a roadmap, a built node and a dreamed node render
   identically. The scoreboard pins every node to an honest status so nobody
   mistakes `Planned` for `Live`.
2. **Cold-start amnesia.** A long build arc holds enormous working context —
   deployed state, gotchas, open questions — expensive to re-derive from commits
   when a conversation resets. The ledger holds it durably.
3. **Invisible status.** Stakeholders shouldn't have to clone the repo to learn the
   roadmap or what shipped. When a mirror adapter is configured it publishes both
   where the team already looks; local-only projects skip this.

### The invariants (the actual product)

The files are just paper. These disciplines are what make it work:

- **Status honesty.** Every capability is pinned to an honest label on a fixed
  spectrum — *proven-live → built-but-unproven → planned → speculative*. The exact
  words flex per project (`Live / Current / Planned` for deployed software;
  `Current / Built-mock / Planned / Research` for greenfield). Uncertain is marked
  uncertain, never optimistic — and every `/ledger` run ends with a one-line 🟢/🟡/🔴
  status signal, the same honesty at a glance.
- **Paired session cadence.** Work happens in sessions. Each **opens** with a brief
  (one bounded goal, intent before code) and **closes** with a close-out
  (disposition, evidence, what's next). The close-out reconciles the canonical files
  and, if a mirror adapter is configured, publishes. The journal is append-only; the
  scoreboard and queue are mutable.
- **The Contract binds the agent.** A standing set of rules of engagement governs how
  any agent works on the project: stop on uncertainty (ask, never guess), no
  destructive action without an in-the-moment human yes, a senior-engineer quality bar,
  grounding in current primary docs (don't assume from memory), and the honest-status
  obligation above. It also defines **autonomy** as an opt-in exception — when the
  operator authorizes it (`/ledger open --autonomous`, "plow ahead"), the agent proceeds
  through ordinary ambiguity by logging stated assumptions in the ledger, while the
  destructive floor and honest status never lift. Amended by appending dated entries,
  never rewritten. The cheap guardrail that ships in both tiers.

---

## The artifacts

The **Tier** column marks what `bootstrap` scaffolds: `both` = minimal and full tiers;
`full` = full tier only. (See `skills/ledger/reference/tiers.md`.)

| Artifact | File | Job | Tier | Cadence |
|---|---|---|---|---|
| **Contract** | `<PREFIX>-CONTRACT.md` | Rules of engagement for any agent — stop on uncertainty, no destructive action without a human yes, senior-engineer quality bar, ground-in-current-docs, honest status, and opt-in autonomy. | both | amend-only |
| **Frame** | `<PREFIX>-OVERVIEW.md` | What we're building and why — thesis, destination. The north star. | full | rare edits |
| **Scoreboard** | `docs/<PREFIX>_BUILD_STATUS.md` | Where are we — every node + status + live operational state. **Start here.** | both | every session |
| **Queue** | `<PREFIX>_OPEN_ITEMS.md` | What's next — open items with stable IDs + a *trigger* each, plus a resolved trail. | both | every session |
| **Variance Log** | `<PREFIX>_VARIANCE_LOG.md` | What diverged — `V-#` operational variances, `VAR-#` documentation concerns. | full | as it happens |
| **Bugs Log** | `docs/bugs/bugs_log.md` | Defects found → fixed (`BUG-#`). Distinct from variances: defects, not design/doc divergences. | full | every session |
| **Runbook** | `docs/deploy/deploy_runbook.md` | Deploy, access live state, roll back, troubleshoot. | full | on infra change |
| **Testing** | `docs/testing/testing_procedure.md` | How to run the tests, harnesses, pass/fail. | full | on test-setup change |
| **Research** | `docs/research/` | Deep-dives, comparisons, explorations, with an index. | full | grows |
| **Journal** | `docs/briefs/BRIEF-session-NN-*.md` + `SESSION-NN-close-out.md` | Per-session intent + paired outcome. The narrative spine. | full | per session |
| **Digest** | `docs/digests/digest-<START>_to_<END>.md` | Per-window stakeholder summary of what shipped + pending. | full, on demand | on `/ledger digest` |
| **Index** | `README.md` "Where are we?" block | The front door pointing at all of the above. | both | on bootstrap |
| **Sync map** | `.ledger/ledger.json` | Config + tier + commit identity + mirror adapter + its IDs. The idempotency record. | both | every publish |

`<PREFIX>` is the project's short slug in SCREAMING_CASE (e.g. `ADMIN`,
`CONVMATCHENG`). Templates for every artifact live in `skills/ledger/templates/`.

---

## The session loop

```
open  →  BRIEF-session-NN (bounded goal, eyeball pass)
          │
          ▼
        do the work
          │
          ▼
close →  SESSION-NN-close-out (disposition + evidence + next)
          │  reconcile: scoreboard · queue · variance · bugs
          ▼
publish →  via the mirror adapter (none = local-only default; atlassian = Confluence + Jira)
          │  + optional Slack ping (notify.slack)
          ▼
        🟢/🟡/🔴 status signal
```

---

## Commands (`/ledger <mode>`)

| Mode | What it does |
|---|---|
| `analyze` | **Read-only.** Survey what the project already has (roadmap, planning docs, issues/milestones, status conventions, in-flight work) and produce an **Adoption Analysis** — proposed config, an ADOPT/CREATE/CONFLICT/LEAVE map, a draft scoreboard, and roadmap risks. Writes nothing. Run it before adopting on a mid-flight project. |
| `bootstrap [--dry-run]` | **Analysis-first and gated.** Runs `analyze`, presents the plan, and **stops for your approval before writing anything** — existing files are never overwritten, only missing ones are created. Then seeds from real state, picks tier + adapter, writes the Contract + agent-context pointer + `.ledger/ledger.json`. Does **not** publish. `--dry-run` stops after the analysis. |
| `open <name> [--autonomous]` | Begin a session with a brief — one bounded goal, intent before code. `--autonomous` lifts stop-on-uncertainty for ordinary ambiguity (assumptions get logged), never for destructive actions. |
| `close [session]` | The heavy mode. Write the close-out, reconcile the canonical files (scoreboard, queue, variance log, and — full tier — bugs log), then publish via the configured adapter. Files first (source of truth), publish second. |
| `status` | Answer "where are we" in seconds from the scoreboard. Flags drift if the scoreboard is staler than the latest close-out. Also reports **capabilities** — mirror / Slack as `on` / `off` / `unavailable`. |
| `sync` | Force a re-publish through the configured mirror adapter from current file state, no close-out (use after hand-edits or right after bootstrap). |
| `digest [--since <when>] [--to <when>]` | Generate a digest of what shipped and what's pending over a time window (default: last 7 days). Pulls from session close-outs and the resolved queue. Writes `docs/digests/` (local-first) and, if a publishing adapter is set, mirrors a page under the hub; can also ping Slack. On demand — never auto-runs on close. |

---

## Mirror adapters

Git is always the source of truth. Publishing is a **per-project adapter**, chosen at
bootstrap and stored in `.ledger/ledger.json` (`mirror.adapter`). The discipline runs
the same everywhere; only the publish target changes.

- **`none`** (the default) — local-only. No external mirror; the files in the repo are
  the whole ledger. Use for solo, greenfield, or short-arc projects.
- **`atlassian`** — a generated, **one-way** mirror to Confluence (a hub page with one
  child per artifact + a session-log subtree) and Jira (each open item `A-#` / `OQ-#`
  and each bug `BUG-#` becomes an issue; resolved/fixed items transition to Done,
  keeping their key). Refreshed on `close` (or manually with `sync`). Avoids two-master
  drift by never reading state back.

Idempotency comes from the committed sync map (`.ledger/ledger.json`), which holds each
page's ID and each open item's key under `mirror`. First publish creates; every publish
after updates by ID. No duplicates. Adapter contract + schema in `skills/ledger/adapters/README.md`.

When the `atlassian` adapter is configured, the Confluence tree under the hub is:
Build Status · Roadmap & Open Items · Variance Log · Contract · Runbook · Testing
Procedure · Bugs Log · Research (parent + one child per doc) · Session Log (one child
per session). Pages publish only when their source artifact exists (tier-dependent).
Jira gets each open item and each `BUG-#` as an issue (`BUG-#` as type Bug).

The `atlassian` adapter's example target is the Reflex workspace (site
`reflexmedia.atlassian.net`, Confluence space `ARDM`, Jira project `ARDM`, Vertical
`Platform`, MCP server `atlassian-reflex`) — read from `CLAUDE.md` / the `mirror` block,
never baked into the skill. Another workspace fills its own. Commit identity is config
too (`commit` block; defaults to the repo's own git identity, no injected co-author).

### Notifications (Slack)

Separate from the mirror, a **notifier** can ping a channel when work lands. Slack is the
first: enable `notify.slack` in `.ledger/ledger.json` and it posts the `digest` summary
(and optionally the `close` disposition) to a channel via an incoming webhook. It
**composes** with any adapter — even `none` gets the ping (with the local path as the
link). The webhook URL is read from an env var (`webhookEnvVar`, default
`LEDGER_SLACK_WEBHOOK`), never committed; a missing webhook fails soft (the digest/close
still completes). Off by default. Contract + procedure in `skills/ledger/notifiers/`.

**Capabilities & graceful degradation.** `mirror` and `notify` are independent on/off
switches for the project's external functions. A project with no Confluence/Jira or Slack
just runs local-only; and a function that's enabled but whose dependency is missing at
runtime (MCP server out of scope, webhook unset) **degrades** — the local ledger is always
written, only the external step is skipped and reported (`DONE_WITH_CONCERNS`).
`/ledger status` shows each as `on` / `off` / `unavailable`.

---

## Repository layout

The repo is a plugin: manifests at the root, the skill under `skills/ledger/`.

```
project-ledger/                    ← repo root · plugin · marketplace
├── README.md                      ← you are here (project overview)
├── BACKLOG.md                     deferred items + known-unproven surfaces
├── .gitignore
├── .claude-plugin/
│   ├── plugin.json                plugin manifest (name: project-ledger; no pinned version)
│   └── marketplace.json           marketplace (alfredintel-skills) → this same repo
└── skills/
    └── ledger/                    ← the skill, invoked /ledger
        ├── SKILL.md               the entrypoint: dispatch + the seven modes
        ├── reference/
        │   ├── concept.md         the methodology, in depth
        │   └── tiers.md           minimal vs full — what each scaffolds
        ├── adapters/              pluggable publish targets (the mirror)
        │   ├── README.md          the adapter contract + the ledger.json schema
        │   ├── none.md            local-only (the default)
        │   └── atlassian.md       Confluence + Jira publish procedure
        ├── notifiers/             pluggable channel pings (compose with the mirror)
        │   ├── README.md          the notifier contract + the notify schema
        │   └── slack.md           Slack incoming-webhook procedure
        └── templates/             one per artifact + the README / CLAUDE blocks
            ├── CONTRACT.md
            ├── CLAUDE-ledger-block.md
            ├── FRAME.md
            ├── BUILD_STATUS.md
            ├── OPEN_ITEMS.md
            ├── VARIANCE_LOG.md
            ├── bugs_log.md
            ├── deploy_runbook.md
            ├── testing_procedure.md
            ├── research-index.md
            ├── DIGEST.md
            ├── BRIEF-session.md
            ├── SESSION-close-out.md
            └── README-index.md
```

---

## Install

**As a plugin (recommended — one command, auto-updating):**

```
/plugin marketplace add alfredintel/project-ledger
/plugin install project-ledger@alfredintel-skills
```

The marketplace is this same repo, and the plugin pins no `version`, so every push to
`main` is picked up as an update (`/plugin update`).

**For local development** (edits live immediately), symlink the skill directory instead:

```bash
ln -sfn /Users/alin/Documents/PersonalCode/Skills/ledger/skills/ledger ~/.claude/skills/ledger
```

Either way, in any repo: `/ledger analyze` to preview safely (read-only), or
`/ledger bootstrap` to start tracking — bootstrap analyzes and asks for approval before it
writes anything.

> Note: Claude Code discovers skills/plugins at session start, so after first installing,
> start a fresh session for `/ledger` to appear.

---

## Status

Honest status, in the skill's own terms — built (Current) vs proven (Live):

- **Skill:** all seven modes implemented — `analyze`, `bootstrap`, `open`, `close`,
  `status`, `sync`, `digest` — with pluggable mirror adapters (`none` / `atlassian`), a
  Slack notifier, minimal/full tiers, a six-rule governance **Contract** (including
  ground-in-current-docs and opt-in autonomy), a 🟢/🟡/🔴 status-signal convention,
  capabilities + graceful degradation, and runbook / testing / bugs / research / digest
  artifacts.
- **Proven (Live):** the local core — `bootstrap → open → close → digest` on the `none`
  adapter — is dogfood-proven end-to-end on a throwaway repo; the five bugs that surfaced
  (B1–B5) are fixed.
- **Built but unproven:** the `atlassian` mirror (never run against a real Confluence /
  Jira), the real Slack POST (needs a webhook — message format + fail-soft are dry-run
  proven), and `analyze` on a real project. All tracked in `BACKLOG.md`.
- **Not yet adopted:** never bootstrapped on a real project — adoption is deliberately
  gated behind `analyze` so it can't disrupt an in-flight roadmap.
- **Distribution:** packaged as a Claude Code **plugin** (skill in `skills/ledger/`,
  manifests in `.claude-plugin/`); installs via `/plugin install project-ledger@alfredintel-skills`,
  or a dev symlink to `skills/ledger/`. No pinned version → pushes auto-update installers.
- **Next:** *prove* the publish path (atlassian mirror, real Slack POST) and *adopt* on a
  real project — both need your environment, not more building.
- **Provenance:** extracted 2026-06-12 from the tracking discipline of two internal
  projects that independently arrived at the same shape.
