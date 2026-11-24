# Ship command cheatsheet

## Core

```bash
ship
ship -p <dir>
ship -v
ship -n
ship --skip tests,todos
ship --only secrets,tests
ship --json [path]
ship --md [path]
```

## Security

```bash
ship security
ship security -v
ship security -n
ship security --skip deps,auth
ship security --only secrets,env-files
ship security --json
ship security --md
```

## Lifecycle

```bash
ship init
ship init --skip tests
ship init --only secrets
ship init --all
ship update
ship update --check
ship update --force
ship -V
```

## Reports

- Default run → `ship-report.json` / `ship-report.md`
- Security run → `ship-security-report.json` / `ship-security-report.md`

Critical failures (tests, secrets, tracked env-files) exit non-zero unless `-n`.
