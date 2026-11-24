---
name: ship
description: Pre-deploy checklist CLI for Rust, Node, Python, and Go projects. Use when the user is about to deploy, push, release, or says ship it, ready to ship, pre-deploy, pre-commit gate, or asks to run ship / ship security. Covers install, default checks, deep security scan, git hooks, report export, and interpreting results for agents.
---

# Ship — Pre-Deploy Checklist

Ship is a fast Rust CLI that runs a local pre-flight gate before `git push` or deploy. It is **not** a full SAST/SCA platform. Default path stays sub-second; heavier checks live under `ship security`.

Site: https://ship-v.vercel.app/  
Repo: https://github.com/dominicOT/ship

## When to run

- User is about to deploy, push, tag, or open a release PR
- User says "ship it", "ready to ship", "pre-deploy", "run the checklist"
- Before merging or after finishing a feature that touches secrets, auth, deps, or migrations
- Setting up a new repo's pre-commit gate (`ship init`)

Do **not** treat ship as a substitute for `npm audit` / `cargo audit` / CI security jobs. Use it as the last local gate.

## Install (if missing)

```bash
curl -sSL https://raw.githubusercontent.com/dominicOT/ship/master/scripts/install.sh | bash
# or
cargo install --path .   # from a clone
```

Verify: `ship -V` (or `ship --version`).

## Default run (fast path)

```bash
ship                    # cwd
ship -p /path/to/proj   # explicit root
ship -v                 # verbose snippets
ship -n                 # dry-run (never non-zero)
ship --skip todos,logs
ship --only secrets,tests
ship --json --md        # write ship-report.json + ship-report.md
```

### Default checks

| Check | Critical | What it does |
|-------|----------|--------------|
| tests | yes | Runs detected test runner (`cargo test`, `npm test`, `pytest`, `go test`, …). Failure → non-zero exit. |
| secrets | yes | Fast scan for private keys, cloud tokens, high-entropy assignments, connection strings. Failure → non-zero exit. |
| TODOs | no | `TODO` / `FIXME` / `XXX` / `HACK` |
| console.logs | no | Leftover `console.log`, `dbg!`, `print(`, `fmt.Println`, … |
| feature flags | no | Temporary / suspicious feature-flag usage |
| version | no | Manifest version vs git tags |
| migrations | no | Detected migration tooling / uncommitted schema hints |
| changelog | no | `CHANGELOG.md` presence and freshness vs last tag |

**Exit behavior:** any critical fail (tests, secrets) → exit 1 (unless `-n`). Use this to gate `ship && ./deploy.sh`.

## Deep security (`ship security`)

Slower, broader pass. Run after the fast path is green, or when the change touches auth, deps, env, or infra.

```bash
ship security
ship security -v
ship security --only secrets,env-files
ship security --json    # → ship-security-report.json
ship security --md
```

| Check | Focus |
|-------|--------|
| secrets | Expanded patterns (JWT secrets, more cloud/vendor keys). Also scans Terraform, Dockerfiles, compose, Jenkinsfiles. |
| env-files | **Git-tracked** `.env` / `.env.*` (templates like `.env.example` excluded). Critical if committed. |
| deps | Lockfile parse + small curated advisory seed; missing lockfile (apps); unbounded ranges on security-sensitive deps; unpinned `requirements.txt`. Not a full audit DB. |
| auth | Heuristic JWT/auth smells in JS/TS + Python (Express/Flask): `"none"` alg, literal signing secrets, missing algorithms allowlist, sensitive routes without nearby middleware/decorators. |

## Git hook (`ship init`)

```bash
ship init
ship init --skip tests,logs
ship init --only secrets,todos
ship init --all
```

Creates `.githooks/pre-commit`, sets `core.hooksPath`, and writes `.ship.toml`. Commit both so the team shares the gate. Default hook skips `tests` (too slow for every commit).

## Agent workflow

1. Ensure `ship` is on PATH (`ship -V`).
2. Run `ship --json` (or `ship` + read terminal). Prefer JSON when you need structured findings.
3. If critical fails → fix those first (tests / leaked secrets). Re-run until green.
4. For release or security-sensitive changes → also run `ship security --json`.
5. Fix or explicitly accept remaining warnings; do not ignore critical env-files / secrets hits.
6. Only then suggest push / deploy / tag.

When reporting to the user, summarize blockers first, then warnings. Quote file:line from verbose or report output.

## Config

`.ship.toml` in project root can set default `skip` / `only` lists. CLI flags override.

## Limits (do not overclaim)

- No live DAST, no cloud resource scanning, no full CVE database.
- Auth checks are regex heuristics, not AST analysis — can miss middleware higher in the stack.
- `deps` advisories are a small seed; point users to `cargo audit` / `npm audit` / `pip-audit` for complete coverage.
- Respects `.gitignore` for most walks; env-files check intentionally looks at **tracked** files only.

## Quick reference

```bash
ship && ship security          # full local gate
ship --json --md               # artifacts for CI / agents
ship init                      # team pre-commit hook
ship update                    # self-update binary installs
```

More detail: https://ship-v.vercel.app/ and https://ship-v.vercel.app/security.html
