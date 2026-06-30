# Project Ledger — Backlog

Deferred items surfaced by the **2026-06-29 dogfood** (a throwaway `linkstash` run:
`bootstrap → autonomous session → close → digest` on the `none` adapter). The five
real bugs it found (B1–B5) were fixed the same day; these four are the lower-priority
remainder, plus the surfaces the `none`-adapter run could not exercise.

Written in the ledger's own open-items style (`ID · severity · trigger`).

---

## Open — spec precision / verification

### B6 — Verify the CLAUDE.md section-replace on re-run · severity: low
The end-boundary is now specified (the section runs from its `##` heading to the next
`##` heading or EOF — fixed with B1). First-run *append* is proven by the dogfood; the
*replace-in-place* path on a second `bootstrap` over an existing ledger section is still
unproven.
**Trigger:** next time `bootstrap` runs on a repo that already has the ledger section, or
when step 4 is next touched — verify it replaces, doesn't duplicate or clobber neighbors.

### B7 — "last friday" when today IS Friday · severity: low
The window parser rule (a same-weekday request means the *previous* one) exists but is
only exercisable on that weekday; the dogfood ran on a Monday.
**Trigger:** a Friday run, or add a worked example to the digest window-resolve step and
have it echo the resolved dates for the operator to sanity-check.

### B8 — Status-vocab binary doesn't fit "pre-deploy but will deploy" · severity: low
`bootstrap` forces deployed→`Live/Current/Planned` vs greenfield→`Current/Built-mock/
Planned/Research`. A greenfield project that intends to deploy still wants `Live` as a
future state (the dogfood overrode the binary by hand).
**Trigger:** next revision of the bootstrap config step — offer `Live/Current/Planned`
to any deploy-intending project regardless of greenfield status.

### B9 — No convention for superseded nodes · severity: low
When a node is replaced (the dogfood swapped an in-memory store for SQLite), the
scoreboard has no convention for the retired node — drop it, or mark it `superseded`?
**Trigger:** when a real project replaces a capability and the scoreboard reads ambiguous.

---

## Known unproven (not bugs — untested surfaces)

The dogfood ran on the `none` adapter, so the publishing path is still entirely unproven:

- **`atlassian` adapter** — Confluence page tree, Jira issues, idempotent ID write-back,
  the metadata header rendering, and the roadmap Jira-macro fallback. Never executed.
- **Digest publish via `atlassian`** — the Project Digest parent + per-window children.
  Never executed.
- **Slack notifier** — designed (`notify.slack`), not built.

**Trigger:** the "prove the mirror" milestone (needs a real Atlassian target + the MCP
server in scope) and the "build Slack" milestone.
