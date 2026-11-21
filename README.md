# Karkain Accounting

A local-first, double-entry ledger CLI/TUI with append-only journals and explicit, versioned jurisdiction rule packs.

## Status

| Item | Status |
|---|---|
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain Accounting records financial transactions as a double-entry ledger on the embedded `db2` database. Entries are posted from an append-only journal of balanced debits and credits; the application computes deterministic trial balances and financial summaries from that journal. Jurisdiction-specific tax and reporting rules are supplied as explicit, versioned rule packs that are always labelled "not legal advice / not certified" — the software never claims legal compliance or files returns. It consumes stock valuations from the standalone Karkain Inventory Management application through a defined contract rather than tracking stock itself, and publishes summaries to Karkain ERP. Interaction is entirely through a command-line interface and text UI; there is no graphical client.

## Who it serves

Bookkeepers, small-business owners and developers who want offline bookkeeping, local data ownership and scriptable, journal-centric workflows. It suits deterministic, auditable environments where predictable flat licensing and explicit jurisdiction rules are preferred over implied "compliance" from a SaaS platform. Users must be able to apply their own professional judgment; this tool does not replace an accountant.

## Planned features

- **Double-entry ledger** — chart of accounts, balanced journal entries, period close (planned); append-only journal with immutable posted entries (planned).
- **Reporting** — deterministic trial balance, P&L and balance-sheet summaries (planned); exportable snapshots (planned).
- **Jurisdiction rule packs** — versioned, explicitly labelled tax/reporting rules, never "compliant" (planned); per-jurisdiction configuration selection (planned).
- **Integration** — consume inventory valuations from standalone Karkain Inventory Management (planned); publish summaries to Karkain ERP (planned).
- **Import/export** — partial-JSON journal and account import/export (planned); flat deterministic format (planned).
- **Administration** — roles for posting vs. review (planned); audit log (planned); backup/restore (planned).

## Explicit non-goals

- Claiming legal, statutory or tax compliance, or filing returns; all rule packs are informational and labelled "not legal advice / not certified."
- Tracking SKUs, warehouses, stock counts or performing valuation itself — that is Karkain Inventory Management's role.
- Re-implementing ERP's coordination workflows or CRM's front-office records.
- Native GUI in v1.2.0, external SQL engines with JOIN reporting, bank feeds or TLS-dependent services, or any alternate implementation language.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

Windows x86-64 toolchain VERIFIED (compile to native executable and execute). Linux cross-target output PRESENT but execution NOT VERIFIED. macOS NOT VERIFIED. These describe the toolchain, not this application, whose compilation is NOT STARTED.

## Build and run status

No application binary exists yet. The intended generic commands are `karkain build src/main.kark -o karkain-accounting` and `karkain run src/main.kark`. These become active only when implementation starts; until then they are informational and have not been executed for this application.

## Known blockers

- Native GUI toolkit BLOCKED — CLI/TUI-first design; GUI is a planned, blocked layer.
- OS atomic-write/file locking PARTIAL — `db_begin`/`db_commit`/`db_rollback` exist, but OS-level atomic write is unverified; ledger durability needs explicit validation before release.
- Full JSON DOM/schema PARTIAL — import/export limited to simple field extraction.
- `system`/`env`/`time` PARTIAL shell wrappers — must be probed before posting-date claims.
- Embedded `db2` SQL subset with no JOINs; reporting uses precomputed balances.
- TLS BLOCKED — no bank-feed or external tax-service integration.

## Repository

GitHub: https://github.com/ajit-ai/Karkain-Accounting — issues: https://github.com/ajit-ai/Karkain-Accounting/issues — License: Apache-2.0.
