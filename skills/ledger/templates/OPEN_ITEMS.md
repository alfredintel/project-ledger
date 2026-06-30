# {{PROJECT}} — Open Items Ledger

The settled work queue: deferred scope, hardening, open questions, and forward
records future sessions must know. IDs are stable (`{{X}}-#` work, `OQ-#` open
questions); items move from **Open** to the **Resolved (carried for trail)** section
but keep their ID and a one-line resolution.

**Related:** `docs/{{PREFIX}}_BUILD_STATUS.md` (scoreboard) ·
`{{PREFIX}}_VARIANCE_LOG.md` (what diverged) · `{{PREFIX}}-OVERVIEW.md` (frame) ·
`docs/briefs/` (per-session intent + close-out).

**Last reconciled:** {{DATE}} (post session NN).

---

## Open — severity-ranked

### {{X}}-1 — {{title}}  · severity: {{high|medium|low}}
{{What it is, why it's deferred, where the design/SQL/spec lives.}}
**Trigger:** {{the condition that should pull this back onto the active path}}.

### {{X}}-2 — {{title}}  · severity: {{...}}
{{...}}
**Trigger:** {{...}}.

### OQ-1 — {{open question}}  · open question
{{The question; what's intentional for now; what changes if the answer flips.}}
**Owner decision:** {{who}}.

---

## Resolved (carried for the resolution trail)

### {{X}}-DONE — {{title}} — RESOLVED (session NN)
{{One or two lines: what the problem was and what closed it.}}
