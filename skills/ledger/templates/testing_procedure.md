# {{PROJECT}} — Testing Procedure

> How to run the tests, what harnesses exist, and what pass/fail means. Fairly static;
> refresh when the test setup changes.

**Last updated:** {{DATE}}.

## How to run

```bash
{{the command(s) to run the full suite}}
```

## Harnesses

| Harness | Scope | How to run |
|---|---|---|
| {{unit}} | {{what it covers}} | {{cmd}} |
| {{integration / e2e}} | {{...}} | {{cmd}} |
| {{live verification}} | {{what it proves against the real target}} | {{cmd}} |

## Pass / fail criteria

- {{what "green" means — required suites, coverage bar, live-verify result}}
- {{what blocks a node from reaching `Live` on the scoreboard}}

## Adding a test

{{Conventions: where tests live, naming, how to add one, what must be covered before a
change is considered done.}}
