# {{PROJECT}} — Build Status Map

**Last updated:** {{DATE}} ({{one-line: post session NN — what changed}})
**Purpose:** the *scoreboard*. One page that answers "where are we" without
re-deriving it from commits and briefs. Every capability pinned to an honest status,
the session that touched it, and what's next.

**How this differs from the other tracking files:**
- `{{PREFIX}}-OVERVIEW.md` = the **frame** (what we're building and why).
- `{{PREFIX}}_OPEN_ITEMS.md` = the **work queue** (what's open/next, with triggers).
- `{{PREFIX}}_VARIANCE_LOG.md` = **what diverged** (shipped gotchas + doc concerns).
- **This file = the scoreboard** (what's *proven*, what's *built*, what's *planned*).

---

## Status legend

| Status | Meaning |
|---|---|
| **Live** | Deployed and verified against the real target |
| **Current** | Built and in the repo — not yet proven against the real target |
| **Planned** | On the roadmap, designed or partially designed, not built |
| **Superseded** | Replaced by another node; kept briefly for the trail, points to its replacement |
{{add Built-mock / Research rows if the project uses them}}

---

## One-line summary

{{The honest one-liner. What's proven, in one or two sentences. Then what's left —
framed as hardening vs new capability where that's the truth.}}

---

## Node-by-node status

### {{Group, e.g. Data source}}
| Node | Status | Session | Notes |
|---|---|---|---|
| {{node}} | **Live** | {{NN}} | {{evidence / caveat}} |

### {{Group, e.g. App}}
| Node | Status | Session | Notes |
|---|---|---|---|
| {{node}} | **Current** | {{NN}} | {{...}} |

### Planned / not built
| Node | Status | Tracked as |
|---|---|---|
| {{node}} | **Planned** | {{X-#}} |

---

## Live operational state (carry between sittings)

- **{{Box / host}}:** {{id, region, specs}}
- **App / image:** {{what runs, where, how}}
- **DB:** {{name, host, TLS mode, creds source}}
- **Access:** {{the exact command to reach it}}
- **Last verify:** {{harness result + date}}

---

## Maintenance rule

Update this file when a node changes status, when something is deployed/verified, or
when a new capability lands. Keep `Live` honest — it means *proven against the real
target*, not merely committed. If a node's status is uncertain, mark it uncertain
rather than optimistic. Granularity: a node is a capability a stakeholder would ask
about — split a node when one honest label no longer covers it; merge nodes nobody asks
about separately. When a node is **replaced**, mark the old one `Superseded` with a
pointer to its replacement instead of deleting it silently; drop it once the replacement
is `Live`.
