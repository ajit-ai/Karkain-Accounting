# Karkain Accounting — First Implementation Backlog

## First vertical slice

**Slice goal:** Prove a complete, honest posting workflow: define two accounts, post a balanced journal entry, recompute a deterministic trial balance, and confirm the active jurisdiction rule pack always renders its mandatory "not legal advice / not certified" label — real persisted work with durability discipline.

**Karkain modules/files expected:** `src/main.kark` (dispatcher), `src/accounts.kark`, `src/journal.kark`, `src/ledger.kark`, `src/rulepacks.kark`, `src/store.kark` (using `db` with `db_begin`/`db_commit`/`db_rollback`, `io`, `json` PARTIAL, `crypto`, `collections`).

**Exact input:** `karkain-accounting account create --code 1000 --name Cash --type asset` and `... --code 4000 --name Revenue --type income`, then `karkain-accounting journal post --id je-1 --date 2026-11-21 --line 1000:debit:250.00 --line 4000:credit:250.00 --rulepack generic.v1`, then `karkain-accounting report trial-balance`.

**Exact expected observable output:** `Created 1000`, `Created 4000`; `Posted je-1 balanced=250.00 rulepack=generic.v1 label="not legal advice / not certified"`; trial balance prints `Cash|debit|250.00`, `Revenue|credit|250.00`, `TOTAL|250.00|250.00`. An unbalanced entry prints `Rejected je-2 imbalance=50.00` and writes no journal row. There is no output path in which the rule-pack label is omitted.

**Verification command pattern (Karkain v1.2.0):** With the Go front-end engine selected, build/run with the verified C-free native target —
```
karkain build src/main.kark --target native-x86_64-windows -o karkain-accounting
.\karkain-accounting account create --code 1000 --name Cash --type asset
.\karkain-accounting account create --code 4000 --name Revenue --type income
.\karkain-accounting journal post --id je-1 --date 2026-11-21 --line 1000:debit:250.00 --line 4000:credit:250.00 --rulepack generic.v1
.\karkain-accounting report trial-balance
```
Build must exit 0 and the executable must print the exact lines above. No test has been run yet; this is the planned verification pattern.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
|---|---|---|---|---|---|---|---|---|
| ACC-001 | Command dispatcher & config profile | `main.kark`, `config_profile.kark` | None | All subcommands dispatch; no hard-coded paths | Unit, CLI smoke | S | `system`/`env` PARTIAL | Builds, help lists commands |
| ACC-002 | Chart of accounts CRUD | `accounts.kark`, `store.kark` | ACC-001 | Unique codes, typed accounts, persisted in `db2` | Unit, integration | M | `db` PRESENT/PARTIAL | CRUD survives restart |
| ACC-003 | Append-only journal & balanced posting | `journal.kark`, `store.kark` | ACC-002 | Unbalanced rejected; posted entries immutable | Unit, integration | L | `db` transactions; atomic write PARTIAL | Replay preserves balance |
| ACC-004 | Deterministic trial balance & ledger queries | `ledger.kark` | ACC-003 | Totals match fixtures; output reproducible | Unit, regression | M | None | Trial balance deterministic |
| ACC-005 | Rule packs with mandatory non-certified label | `rulepacks.kark` | ACC-001 | Versioned data; label always rendered; never "compliant" | Unit, security | M | None | No label-free output path |
| ACC-006 | Period close & logged override | `periods.kark`, `audit.kark` | ACC-003, ACC-005 | Closed periods locked; override logged | Integration, security | M | `time` PARTIAL wrapper | Override auditable |
| ACC-007 | P&L / balance-sheet summaries (no JOINs) | `reporting.kark` | ACC-004 | Precomputed reports match journal | Integration, regression | M | `db2` no JOINs | Snapshots reproducible |
| ACC-008 | Partial-JSON / flat import-export | `importexport.kark` | ACC-002, ACC-003 | Deterministic round-trip; duplicates detected | Integration | L | `json` PARTIAL | Export re-imports identically |
| ACC-009 | Inventory valuation & ERP summary contracts | `integration.kark` | ACC-004, ACC-007 | Valuation consumed; summary published via versioned contract | Integration | L | `json` PARTIAL; contracts undefined | Round-trip with stub apps |
| ACC-010 | Roles, audit trail & backup/restore | `auth.kark`, `audit.kark`, `backup.kark` | ACC-001, ACC-003 | Poster/reviewer separated; consistent pair backup | Security, failure | L | atomic write PARTIAL | Restore recomputes identical balance |

## Dependency order and gates

1. ACC-001 → 2. ACC-002 + ACC-005 (parallel) → 3. ACC-003 (posting) → 4. ACC-004 + ACC-006 + ACC-007 (ledger/period/reports) → 5. ACC-008 + ACC-009 (IO/integration) → 6. ACC-010 (admin/durability). Gate between phases: the actual Karkain v1.2.0 compiled executable must be run and expected output verified before advancing; durability of the journal must be explicitly validated before any production-readiness claim, and the non-certified rule-pack label must be present in every release. Blockers (GUI, JSON, atomic write, TLS) are recorded, never bypassed by changing Karkain.

## Status

All items: **NOT STARTED**. Implementation must not begin until all designs and repository initialization are complete. No tests have been run and no binary exists for this application.
