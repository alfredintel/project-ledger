# {{PROJECT}} — Deploy Runbook

> Operational procedures: how to deploy, reach live state, roll back, and troubleshoot.
> Reconciled on `close` whenever deploy or infra reality changes. Keep it true — a stale
> runbook is worse than none.

**Last updated:** {{DATE}}.

## Environments

| Env | Where | Notes |
|---|---|---|
| {{prod / staging}} | {{host / URL / region}} | {{...}} |

## Deploy

1. {{the exact steps / command to ship}}
2. {{...}}

**Pre-deploy checks:** {{tests green, migration plan, feature flags, backups}}.

## Access live state

- **Host / box:** {{id, region, the exact command to reach it}}
- **App / image:** {{what runs, where, how}}
- **DB:** {{name, host, TLS mode, where creds live}}
- **Logs / dashboards:** {{where to look}}

## Roll back

{{The exact rollback procedure — how to get to the last known-good state, and how long
it takes. The rollback is itself a destructive-ish action: confirm before running it on
prod (see the Contract).}}

## Troubleshoot

- **{{symptom}}** → {{first thing to check, likely cause, fix}}.
- **{{symptom}}** → {{...}}.
