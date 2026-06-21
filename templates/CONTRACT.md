# {{PROJECT}} — Contract (rules of engagement)

> The **Contract**: the standing rules that govern how any agent (Claude Code or
> otherwise) is allowed to engage with this project. Read it before acting; it binds
> every session. Amended only by appending a new dated entry below — never rewritten
> in place (append-only-corrected, like the journal).

**Last amended:** {{DATE}}.

## 1. Stop on uncertainty

If you lose context, are missing information, or are unsure of intent, **stop and ask
the operator.** Never guess, never proceed on an optimistic assumption, never resolve
ambiguity silently. Escalating a question is always acceptable; a confident wrong turn
is not. When two readings of a request are both plausible and the cost of the wrong one
is real, stop and ask.

## 2. No destructive action without an in-the-moment human "yes"

You may **not** take any irreversible or harmful action without an explicit human
confirmation, given in context, for that specific action. This includes (and is not
limited to):

- deleting or destroying servers, hosts, or environments
- dropping, deleting, or truncating databases or tables
- deleting files outside a scratch directory
- force-pushing or rewriting shared git history
- `rm -rf` (or equivalent) on anything outside a scratch dir
- rotating or revoking credentials, tearing down infra, mass external sends

A goal never implies permission for these. "Make it work" is not consent to drop the
database. Name the blast radius, ask, and wait for an explicit yes.

## 3. Senior-engineer quality bar

Every change is held to the standard of a senior engineer who owns this codebase:

- The architecture stays coherent and up to par. Do not degrade it for a quick win.
- Code is the highest level you can produce — clear, correct, tested where it matters.
- Shortcuts that trade architectural integrity for a passing demo are out of bounds.
- Match the surrounding code's conventions; leave the codebase better than you found it.

The Contract's purpose is to keep you on the path that produces the highest-level
engineering, not merely code that runs.

## 4. Honest status

Never overstate build state. Uncertain is marked uncertain; `Live` means proven against
the real target, not merely committed. This binds you to the ledger's status-honesty
invariant — the scoreboard exists to stop vision and reality from blurring, and you do
not blur them.

---

## Amendments

> Append a dated entry to change a rule. Do not edit the rules above in place.

### {{DATE}} — initial Contract
Adopted at bootstrap.
