# {{PROJECT}} — Digest: {{SINCE}} to {{TO}}

> A per-window summary of what shipped and what's pending, synthesized from the session
> close-outs and the resolved queue. Generated on demand by `/ledger digest` — not on
> every close. Local file is the source of truth; the Confluence page (if a publishing
> adapter is configured) is a one-way mirror of this file.

**Window:** {{SINCE}} → {{TO}} · **Generated:** {{DATE}} · **Sessions covered:** {{SESSION_RANGE}}

---

## Summary

From {{SINCE}} to {{TO}}, the project moved {{ADVANCED_COUNT}} capabilities forward —
{{LIVE_COUNT}} reached **Live** (proven against the real target), {{CURRENT_COUNT}}
advanced to **Current** (built, not yet proven) — and resolved {{RESOLVED_COUNT}} open
items. (Honest framing: don't call Current work "shipped"; name what's actually proven.)

**Key accomplishments:**
- {{accomplishment 1 — drawn from a close-out, with evidence}}
- {{accomplishment 2}}
- {{accomplishment 3 (3–5 bullets)}}

**Remaining priorities:**
- {{priority 1 — an open item, with its trigger}}
- {{priority 2}}
- {{priority 3 (3–5 bullets)}}

{{Optional: one or two sentences of narrative — what the arc of the window was, what was
learned, why work was deferred. Keep it honest; uncertain stays uncertain.}}

---

## Advanced this window

| Item | Type | Status | Session | Evidence / Notes |
|---|---|---|---|---|
| {{item or node}} | {{feature / bugfix / hardening}} | {{Live | Current}} | {{NN}} | {{what proves it (Live), or why it's still unproven (Current)}} |

---

## Pending

| Item | Type | Trigger | Priority |
|---|---|---|---|
| {{open item / question}} | {{work / question}} | {{the condition that reactivates it}} | {{high / medium / low}} |

---

## Open questions & variance logged this window

- {{OQ-# / VAR-# / V-# — one-liner, with its ID}}
- {{... or "None this window."}}
