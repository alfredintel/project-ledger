# Mirror adapter: `atlassian` (Confluence + Jira)

One-way mirror: local files → Confluence + Jira. Runs on `close` (auto) and `sync`
(manual). **Git is the source of truth; never read state back from the mirror.** Honor
the adapter contract in `adapters/README.md`.

Selected when `.ledger/ledger.json` has `mirror.adapter = "atlassian"`. All Atlassian
calls use the MCP server named in `mirror.mcpServer` — and only that one. If a workspace
denies a different Atlassian MCP at the project layer (a per-workspace `CLAUDE.md`
concern, not a skill rule), respect that; the skill just calls the configured server.
*Example:* the Reflex workspace sets `mcpServer` to `atlassian-reflex`
(`reflexmedia.atlassian.net`, defined in `ReflexAir/.mcp.json`) and its `CLAUDE.md`
denies the `claude_ai_Atlassian` / `infomaticsai` variant. That's config + a workspace
rule, not part of this adapter.

**Prerequisite.** The `mirror.mcpServer` server must be in scope in the repo you run
from. It is not available at the PersonalCode root or sibling repos unless their
`.mcp.json` defines it. If it is missing or unreachable, finish the local
reconciliation and report `DONE_WITH_CONCERNS` (§5).

---

## 0. The `mirror` block — `.ledger/ledger.json`

This adapter's config + idempotency record. Committed to the repo.

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
    "hub": "<pageId>",
    "build_status": "<pageId>",
    "roadmap": "<pageId>",
    "variance": "<pageId>",
    "contract": "<pageId>",
    "runbook": "<pageId>",
    "testing": "<pageId>",
    "bugs": "<pageId>",
    "research": "<pageId>",
    "research_docs": { "<doc-slug>": "<pageId>" },
    "session_log": "<pageId>",
    "sessions": { "01": "<pageId>", "02": "<pageId>" }
  },
  "jira": {
    "A-1": "ARDM-123",
    "A-2": "ARDM-124",
    "OQ-1": "ARDM-130",
    "BUG-1": "ARDM-140"
  }
}
```

`research` is the parent page (a string), and `research_docs` is its `{doc-slug:
pageId}` sub-map of children — one child per doc in `docs/research/`, mirroring the
`session_log` + `sessions` pattern. The Contract page is included only when the project
publishes it (it does by default; see §2).

The example values are the Reflex/ARDM target. Another Atlassian workspace fills its own
`mcpServer` / `site` / `confluenceSpace` / `jiraProject` / `vertical` — nothing here is
baked into the skill.

- `cloudId`: resolve once with `getAccessibleAtlassianResources`, cache it. Every other
  call needs it.
- An empty string for a `confluence` key means "not created yet" → create it. Only keys
  whose source artifact exists are published (tier-dependent); the rest stay empty.
- A missing `jira[ID]` means "no issue yet" → create it. This covers `BUG-#` keys too.
- `publishContract` (default true) gates the Contract page. `metadataHeader` (default
  true) gates the per-page header table.
- **Write the block back after every create**, so a mid-run failure never orphans a
  page/issue. Commit it at the end of `close`.

---

## 1. Required Confluence metadata header (when `metadataHeader` is true)

If `mirror.metadataHeader` is true, **every** page this adapter creates or updates must
begin with this table as the first block, before the body. In the Reflex workspace it's
a `CLAUDE.md` rule — non-negotiable there.

```markdown
| Status | In Progress |
| --- | --- |
| Last reviewed | YYYY-MM-DD |
| Page type | prd |
| Agent access | agents-readable |
| Vertical | {{VERTICAL}} |
| Feature status | {{FEATURE_STATUS}} |
| Related Jira |  |
```

Fill per page:

| Page | Page type | Status | Feature status | Related Jira |
|---|---|---|---|---|
| Hub | `prd` | from frame | from config | (blank or epic) |
| Build Status | `spec` | `In Progress` | from config | (blank) |
| Roadmap & Open Items | `notes` | `In Progress` | from config | (blank) |
| Variance Log | `notes` | `In Progress` | from config | (blank) |
| Contract | `notes` | `In Progress` | from config | (blank) |
| Runbook | `notes` | `In Progress` | from config | (blank) |
| Testing Procedure | `notes` | `In Progress` | from config | (blank) |
| Bugs Log | `notes` | `In Progress` | from config | `BUG-#` issues |
| Research (parent + children) | `notes` | `In Progress` | from config | (blank) |
| Session NN | `notes` | `Approved` once closed | from config | issues touched |

`Last reviewed` is always today (`date +%F`). `Vertical` and `Feature status` come from
`mirror.vertical` / `mirror.featureStatus`. `Agent access` defaults `agents-readable`.

---

## 2. Confluence page tree

```
<Project> — Project Hub                  (prd)    parent: space root (or configured)
├── <Project> — Build Status             (spec)   parent: hub
├── <Project> — Roadmap & Open Items     (notes)  parent: hub   + Jira filter macro (§4)
├── <Project> — Variance Log             (notes)  parent: hub
├── <Project> — Contract                 (notes)  parent: hub   [if mirror.publishContract]
├── <Project> — Runbook                  (notes)  parent: hub
├── <Project> — Testing Procedure        (notes)  parent: hub
├── <Project> — Bugs Log                 (notes)  parent: hub
├── <Project> — Research                 (notes)  parent: hub   (parent; one child per doc)
│   ├── <Project> — Research: <doc-slug>          parent: research
│   └── …
└── <Project> — Session Log              (notes)  parent: hub
    ├── <Project> — Session 05: <slug>            parent: session_log
    └── <Project> — Session 04: <slug>            parent: session_log
```

Only pages whose source artifact exists are published — a `minimal`-tier project has no
Variance/Runbook/Testing/Bugs/Research/Frame, so those pages are skipped. The Contract
page is published when `mirror.publishContract` is true (default true).

Page bodies (source file → page):

| Page | Source | Notes |
|---|---|---|
| Hub | `<PREFIX>-OVERVIEW.md` | + a links section to the children |
| Build Status | `docs/<PREFIX>_BUILD_STATUS.md` | the scoreboard verbatim |
| Roadmap & Open Items | `<PREFIX>_OPEN_ITEMS.md` | + the Jira filter macro (§4) |
| Variance Log | `<PREFIX>_VARIANCE_LOG.md` | verbatim |
| Contract | `<PREFIX>-CONTRACT.md` | rules of engagement, verbatim |
| Runbook | `docs/deploy/deploy_runbook.md` | verbatim |
| Testing Procedure | `docs/testing/testing_procedure.md` | verbatim |
| Bugs Log | `docs/bugs/bugs_log.md` | verbatim (Open + Fixed sections) |
| Research (parent) | `docs/research/README.md` | the index doc |
| Research: <doc-slug> | `docs/research/<doc-slug>.md` | one child per doc, verbatim |
| Session NN | `docs/briefs/SESSION-NN-close-out.md` | + a link back to its brief |

Strip the source file's top-level `# H1` (the Confluence page has a title). Keep all
tables — they render in storage format. The MCP create/update tools accept Markdown.

### Create-or-update logic (per page)

```
key = "build_status"            # etc.
pageId = mirror.confluence[key]
body = metadataHeader + render(sourceFile)
if pageId is empty:
    res = createConfluencePage(cloudId, mirror.confluenceSpace, title, parentId, body)
    mirror.confluence[key] = res.id            # write back immediately
else:
    updateConfluencePage(cloudId, pageId, title, body)   # update in place
```

Order: **hub first** (its ID is the parent for all the rest), then each content child
that has a source artifact — `build_status`, `roadmap`, `variance`, `contract`,
`runbook`, `testing`, `bugs` — then the `research` parent and its per-doc children, then
`session_log`, then the Session NN page under `session_log`. Skip any page whose source
file doesn't exist (tier-dependent). Write each ID back to the `mirror` block
immediately after creating it.

### Research tree (parent + one child per doc)

Create-or-update the `research` parent page from `docs/research/README.md`
(`mirror.confluence.research`). Then for each `docs/research/<doc-slug>.md` (excluding
the README index), create-or-update a child under it, keyed by `<doc-slug>` in
`mirror.confluence.research_docs` — exactly the `session_log` + `sessions` pattern.

### Session page (new each close)

`mirror.confluence.sessions[NN]` — create once, update on re-publish. Parent =
`mirror.confluence.session_log`. Title `<Project> — Session NN: <kebab-goal>`.

---

## 3. Jira mirror — open items and bugs become issues

On every `close`/`sync`, walk the open items parsed from `<PREFIX>_OPEN_ITEMS.md` (both
the **Open** section and the **Resolved** section), then the bugs parsed from
`docs/bugs/bugs_log.md` (both **Open** and **Fixed**, full tier only).

Resolve issue-type IDs once per project with
`getJiraProjectIssueTypesMetadata(cloudId, mirror.jiraProject)` — `Task` is the safe
default (Story if the project uses it), and `Bug` for the bugs walk (fall back to `Task`
if the project has no Bug type).

For each open item `ID` (e.g. `A-1`, `OQ-1`):

```
key = mirror.jira[ID]
summary     = "[<ID>] <one-line title>"
description = the item's body + its Trigger line
labels      = ["ledger", "ledger-<slug>", "ledger-id-<ID>"]
priority    = severityToPriority(item.severity)   # high→High, medium→Medium, low→Low
                                                  # OQ-#/open-question → Low + label "open-question"

if key is empty:
    res = createJiraIssue(cloudId, mirror.jiraProject, issueTypeId, summary, description, labels, priority)
    mirror.jira[ID] = res.key                      # write back immediately
else:
    editJiraIssue(cloudId, key, {summary, description, priority, labels})

if item is in the Resolved section:
    transitions = getTransitionsForJiraIssue(cloudId, key)
    pick the transition to a Done/Closed status; transitionJiraIssue(cloudId, key, transitionId)
    # include the one-line resolution as a comment if useful
```

### Bugs walk (full tier, type Bug)

For each `BUG-#` parsed from `docs/bugs/bugs_log.md`:

```
key = mirror.jira[BUG-#]
summary     = "[BUG-#] <one-line title>"
description = the bug's Symptom / Where / Repro / Suspected cause
labels      = ["ledger", "ledger-<slug>", "ledger-id-BUG-#"]
priority    = severityToPriority(bug.severity)

if key is empty:
    res = createJiraIssue(cloudId, mirror.jiraProject, bugIssueTypeId, summary, description, labels, priority)
    mirror.jira[BUG-#] = res.key                   # write back immediately
else:
    editJiraIssue(cloudId, key, {summary, description, priority, labels})

if the bug is in the Fixed section:
    transitions = getTransitionsForJiraIssue(cloudId, key)
    pick the transition to Done/Closed; transitionJiraIssue(cloudId, key, transitionId)
    # include the one-line resolution as a comment if useful
```

Notes:
- **Local file wins.** Jira reflects the file. A manual Jira edit to a ledger-managed
  issue may be overwritten on the next publish — that's the cost of the mirror model.
- Never delete Jira issues. Resolved/fixed items transition to Done and keep their key.
- `OQ-#` open questions become Tasks labeled `open-question` (owner-decision items), not
  bugs. `BUG-#` defects become issues of type **Bug**.

---

## 4. Roadmap page — embedded Jira filter

The *Roadmap & Open Items* page embeds a live view of the project's ledger issues so the
forward queue is visible in Confluence without leaving the page. Append the Jira issues
macro (Confluence storage format) with this JQL:

```
project = <mirror.jiraProject> AND labels = ledger-<slug> ORDER BY priority DESC, status ASC
```

Storage-format snippet to include in the page body (example uses the Reflex/ARDM
target):

```xml
<ac:structured-macro ac:name="jira">
  <ac:parameter ac:name="server">System JIRA</ac:parameter>
  <ac:parameter ac:name="jqlQuery">project = ARDM AND labels = ledger-convmatcheng-admin ORDER BY priority DESC</ac:parameter>
  <ac:parameter ac:name="columns">key,summary,priority,status</ac:parameter>
</ac:structured-macro>
```

If the create/update tool only accepts Markdown and won't pass the macro through, fall
back to a static table of `ID → Jira key → status` rendered from the mirror block, and
note that the live macro can be added in the Confluence UI once.

---

## 5. Failure handling

- Resolve `cloudId` and the space/project IDs **before** the first write; if Atlassian
  is unreachable, finish the local file reconciliation and report `DONE_WITH_CONCERNS`
  (files updated, publish deferred — run `/ledger sync` later). Never leave files
  half-reconciled because publishing failed.
- Write the mirror block back after **each** successful create (not just at the end) so
  a partial run is resumable and never creates a duplicate on retry.
- Commit `.ledger/ledger.json` with the reconciled files at the end of `close`.
