# {{PROJECT}} — Variance Log

Running record of where the project diverged from intent. Two kinds of entries, kept
distinct:

- **Shipped/operational variances (`V-#`)** — something behaved differently than the
  plan assumed: a wrong default, a gotcha hit on first run, an infra reality the docs
  didn't expect. Records the gap and what closed it, so it isn't repeated.
- **Documentation concerns (`VAR-#`)** — a claim that is stale, best-match, or
  unverified and could be mistaken for confirmed fact. Records what was written, what
  is actually true, and the action needed.

**Related:** `{{PREFIX}}_OPEN_ITEMS.md` (forward work) ·
`docs/{{PREFIX}}_BUILD_STATUS.md` (scoreboard).
**Last updated:** {{DATE}}.

---

## Shipped / operational variances (`V-#`)

### V-1 — {{short title}} (session NN)
**What happened:** {{the surprise, the symptom}}.
**Evidence gap:** {{what the plan/template assumed that turned out false}}.
**Closed by:** {{the fix + where it's documented}}. **Severity:** {{...}}.

---

## Documentation concerns (`VAR-#`)

### VAR-1 — {{short title}}  · status: open
**What's written:** {{the claim as it currently reads}}.
**What's actually true:** {{the real state}}.
**Action needed:** {{what to do}}. Tracked as {{X-#}}. **Severity:** {{...}}.

---

## Status values

| Status | Meaning |
|---|---|
| Open | Logged, not yet corrected |
| Resolved | Corrected / closed (kept for the trail) |
| Deferred | Known, intentionally not addressed now |
