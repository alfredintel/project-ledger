# Project Ledger — Backlog

Deferred items surfaced by the **2026-06-29 dogfood** (a throwaway `linkstash` run:
`bootstrap → autonomous session → close → digest` on the `none` adapter). The five
real bugs it found (B1–B5) were fixed the same day, and B8–B9 (plus seeding the full config schema) in a
2026-06-30 polish pass; B6–B7 remain open, plus the surfaces the `none`-adapter run
could not exercise. B10–B11 were deferred by design in the 2026-07-08 concept/method
review (which also landed the fix batch: allowed-tools removal, ccusage pinning, the
Brief: pointer, concurrency doctrine, status drift checks, and a doc-drift sweep).

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

---

## Open — deferred by design (2026-07-08 concept review)

### B10 — Multi-writer lanes: mint session numbers + item IDs at close · severity: low
The concurrency doctrine is documented (concept.md → *Concurrency — one poster at a
time*): canonical files are single-poster, sessions serialize, an orchestrator's
subagents share its session. The mechanics for true concurrent writers — slug-named
briefs, `NN` and item IDs minted only at the serialized close, close-outs carrying
deltas the posting applies — are deliberately unbuilt: they would churn a dogfood-proven
surface for a writer that doesn't exist yet, and the close-out's **Brief:** pointer
already makes the eventual change non-breaking.
**Trigger:** the first project with a genuine second concurrent writer (two humans, or
two independent agent lanes), or the first team adoption.

### B11 — Hook-enforced destructive floor · severity: low
Contract rule 2 (no destructive action without an in-the-moment human yes) is soft,
prompt-level governance today. A PreToolUse hook could hard-enforce the floor for
autonomous sessions.
**Trigger:** the soft floor demonstrably failing in practice, or unattended autonomous
runs against infrastructure that can actually be damaged.

---

## Resolved (carried for the trail)

### B8 — Status-vocab binary — RESOLVED (2026-06-30, polish pass)
Bootstrap and analyze no longer force a deployed-vs-greenfield binary: vocab defaults to
`Live / Current / Planned` for anything with or intending a deploy target (`Live` is a
valid *future* state pre-deploy); the research vocab is reserved for exploratory work.

### B9 — Superseded-node convention — RESOLVED (2026-06-30, polish pass)
Added a `Superseded` status to the scoreboard legend + a maintenance rule: a replaced node
is marked `Superseded` with a pointer to its replacement, dropped once the replacement is
`Live`. (`templates/BUILD_STATUS.md` + the close reconcile step.)

### Seed the full config schema — RESOLVED (2026-06-30, polish pass)
Bootstrap now writes the complete `.ledger/ledger.json` (including `digest` and `notify`
seeded **off**), so every capability switch is discoverable in the file, not just in docs.

---

## Known unproven (not bugs — untested surfaces)

The dogfood ran on the `none` adapter, so the publishing path is still entirely unproven:

- **`atlassian` adapter** — Confluence page tree, Jira issues, idempotent ID write-back,
  the metadata header rendering, and the roadmap Jira-macro fallback. Never executed.
- **Digest publish via `atlassian`** — the Project Digest parent + per-window children.
  Never executed.
- **Slack notifier** — **POST + fail-soft proven (2026-07-01):** a real incoming webhook
  returned `200 / ok` for the documented `curl`/`jq` path, and the unset-webhook path
  skips clean (exit 0). **Still unproven:** the notifier firing *automatically* from a real
  `/ledger close` / `digest` event (message auto-built from close-out data), which needs a
  bootstrapped ledger to run the mode end-to-end.
- **Autonomous usage guard** — **built** (`autonomy.usageGuard` config + SKILL.md →
  *Autonomous usage guard*), but **unproven**: needs an autonomous session run near a real
  5-hour / weekly cap to confirm it checks usage, stops with a clean **PARTIAL** close-out
  instead of a mid-arc cutoff, and resumes on window-clear — plus the fail-soft path when
  `ccusage` is absent. **Trigger:** the first real `/ledger open --autonomous` arc long
  enough to approach a cap.

**Trigger:** the "prove the mirror" milestone (needs a real Atlassian target + the MCP
server in scope), and a Slack webhook to prove the notifier end-to-end.
