# Karkain Accounting — Product Specification

**Document status:** Design COMPLETE. Research = COMPLETE. Repository creation = COMPLETE. Application implementation = NOT STARTED. Application compile/execute verification = NOT STARTED (toolchain path itself VERIFIED on Karkain v1.2.0).


## A. Product definition

**Product name:** Karkain Accounting.

**Purpose/problem:** Small businesses need dependable offline bookkeeping with deterministic balances and local data ownership. Cloud tools are browser-bound, tier-gated and market implicit "compliance," while a local ledger must make jurisdiction-specific tax behaviour explicit and auditable. Karkain Accounting is a double-entry ledger whose rules are data, not claims.

**Vision:** The journal-centric, scriptable financial record of the Karkain portfolio — deterministic, local-first and honest about what its jurisdiction rule packs do and do not guarantee.

**Users/segments:** Bookkeepers, small-business owners and developer-operators in offline or air-gapped settings wanting reproducible journal workflows and explicit rule selection. Users must apply professional judgment; this is not legal advice.

**Usage scenarios:** Post a balanced entry and compute a trial balance; apply a selected rule pack to categorize a transaction with its "not certified" label visible; close a period and export snapshots; consume an inventory valuation from Karkain Inventory Management as an asset adjustment.

**Positioning/value:** A CLI/TUI double-entry ledger on embedded `db2`, with append-only journal, deterministic trial balance, partial-JSON import/export and versioned rule packs labelled "not legal advice / not certified." Value: offline, auditable, flat-cost bookkeeping with no implied compliance.

**Non-goals/boundaries:** The product must **never claim legal compliance** and files no returns. It does not track SKUs/warehouses or compute valuation — it consumes valuations from standalone Karkain Inventory Management. It does not duplicate ERP coordination or CRM records. No GUI, external SQL/JOIN reporting, TLS bank feeds or alternate language.

## B. Functional architecture

| Module | Responsibility |
|---|---|
| Chart of accounts | Codes, types (asset/liability/equity/income/expense), hierarchy |
| Journal & posting | Append-only entries, balanced debit/credit validation, immutable posted records |
| Period close | Open/close periods, lock posted periods, adjustments |
| Ledger queries | Balances, transaction listings, deterministic trial balance |
| Reporting | Precomputed P&L and balance-sheet summaries (no JOINs) |
| Jurisdiction rule packs | Versioned, labelled tax/reporting data; categorization helper, never certification |
| Integration | Consume Inventory valuations; publish summaries to ERP |
| Import/export | Partial-JSON and flat journal/account formats |
| Administration | Poster/reviewer roles, audit log, config, backup/restore |

**Core workflows:** Define accounts → post balanced entry → query trial balance → apply rule pack → close period → export snapshots → publish summary to ERP.

**Admin workflows:** Select rule-pack version; define roles; review audit trail; lock/unlock periods with logged override; back up/restore the ledger pair.

**Error/recovery:** Reject unbalanced entries with exact imbalance; quarantine duplicate journal ids; reconcile after abrupt termination by journal replay; require logged reviewer override for closed-period posting.

**Roles/permissions:** Administrator (config, rule-pack selection), Bookkeeper (post/query), Reviewer (approve overrides), Auditor (read-only), enforced at CLI/TUI layer; not a security certification.

**Configuration/customization:** Local profile selects rule-pack version and reporting currency. Rule packs are data files with a mandatory visible "not legal advice / not certified" label; no pack may suppress it.

**Import/export:** Deterministic flat format plus simple `json` field extraction; re-importable with duplicate detection; no full JSON DOM assumed.

**Integration:** Consume valuation summaries from Inventory and publish summaries to ERP via defined contracts over local CLI/file drop; no TLS or external tax services.

## C. Competitor analysis

Competitors: **Zoho Books**, **QuickBooks Online**, **Xero**. Evidence and sources are in `docs/competitor-analysis/COMPETITORS.md`; labels match that document.

**Positioning:** [VF] Zoho Books is compliance-driven cloud accounting with size/region editions. [VF] QuickBooks Online targets small business from Free to Advanced, with Intuit Enterprise Suite and Desktop Enterprise separate. [VF] Xero targets self-employed through medium businesses ("trusted by 5.1 million customers" — vendor claim), no per-user fees. [RI] The sharpest SMB battle is QuickBooks vs Xero; Zoho wins on bundle value.

| Capability | Karkain Accounting [P] | Zoho Books | QuickBooks | Xero |
|---|---|---|---|---|
| CLI/TUI-first | Yes | No | No | No |
| Offline/local ledger (db2) | Yes | No | No | No |
| Explicit jurisdiction rule packs | [P] (never "compliant") | GST/local editions | Tax features | Sales tax |
| Inventory valuation | Via standalone Inventory app | Built-in (upper tiers) | Built-in (Plus+) | Add-on |
| Published volume limits | [P] to publish | Yes (1k–100k invoices/yr) | Tier-based | Tier-based |
| Per-user fees | [P] flat | Users/add-ons | Per tier | None |

**Platform/deployment:** [VF] All three are cloud-first with browser and iOS/Android apps; Zoho Books and QuickBooks also have desktop paths. [RI] None offers a CLI/TUI ledger — a differentiator. Karkain [P] runs as a local CLI/TUI service on `db2`.

**Licensing (verified only):** [VF] Zoho Books (India, INR, billed annually): Free ₹0 (≤25 lakh revenue), Standard ₹899→₹749, Professional ₹1,799→₹1,499, Premium ₹3,599→₹2,999, Elite ₹5,999→₹4,999, Ultimate ₹9,599→₹7,999 per org/month. [VF] QuickBooks (US, 70%-off-3-months promo): Free $0, Simple Start $38→$11.40, Essentials $85→$25.50, Plus $140→$42, Advanced $340→$102 per month (users 1/3/5/25). [VF] Xero (US, promo): Early $27→$2.70, Growing $59→$5.90, Established $9.70 then $97/month; no per-user fees. Karkain licensing: [P] flat Apache-2.0. Promotions are time-bound as of 2026-11-21; no prices invented.

**Strengths/limitations:** [VF/R] Zoho: strong local compliance and transparent tier limits, but feature-gating and INR-region pricing. QuickBooks: brand/trust and AI, but promo-dependent pricing and seat caps. Xero: unlimited users and 1000+ apps, but inventory add-on and promo ending.

**Market gaps:** [RI] Users needing offline bookkeeping, local data ownership, scriptable/journal-centric workflows and jurisdiction rules that are explicit rather than implied "compliance."

**Differentiation:** [P] Double-entry ledger on `db2` with append-only journal, deterministic trial balance, partial-JSON IO and versioned rule packs always labelled non-certified. Integrates with — not replaces — Inventory.

**Deliberately not copied:** Implicit "compliant" marketing, GST/e-invoicing filing, TLS bank feeds, app marketplaces, browser/mobile clients, per-seat fees, built-in stock valuation.

## D. Technical architecture

**Application/boundaries:** A dispatcher fronts accounts, posting, period close, queries, reporting, rule packs, integration, IO and administration. Posting is the sole journal writer; all balances derive from the immutable journal, never mutable cached totals. Single executable as one-shot CLI, TUI loop or minimal local service; Inventory valuations and ERP summaries exchanged via contracts. No non-Karkain in-process extensions.

**UI architecture:** The UI is **CLI/TUI/service-first because Karkain v1.2.0 has no native GUI toolkit (BLOCKED)**. A GUI is a planned layer contingent on ecosystem capability, not part of v1.2.0. Every TUI action has a CLI equivalent and rule-pack labels render in plain text.

**Domain model:** Account, JournalEntry, JournalLine, Period, RulePack, RulePackVersion, ValuationAdjustment, SummarySnapshot, Role, AuditEntry. Entries are append-only; corrections are new reversing entries, never edits.

**Storage strategy:** Embedded `db2` SQL subset for accounts/posted lines plus append-only journal; no external SQL engine; balances computed sequentially, no JOINs.

**API/events:** Stable CLI subcommands plus minimal `http` building-block request/response; no routing/concurrent-server framework. Each posting emits a journal-sequence event for replay; idempotent by journal id; failed imports quarantined.

**Logging/config/extensions:** Structured text logs with run id, posting counters and an audit trail with hash chaining planned via `crypto` sha256 (designed, not implemented). Local profile selects rule-pack version and currency via `config`/`io` once probed, no machine-specific paths. Rule packs are versioned data, never code changing Karkain. Any future FFI must be a justified Karkain-compatible native interface, never a replacement language; none planned.

## E. Data design

**Entities/ownership:** Account, JournalEntry, JournalLine, Period, RulePack, RulePackVersion, ValuationAdjustment, SummarySnapshot, AuditEntry, Role. Lines belong to entries and reference accounts; summaries derive from posted lines. Accounting owns ledger data only.

**Validation/lifecycle:** Entries must balance before posting; account codes must exist; posted entries immutable; rule-pack version declared. Entry: draft→posted→reversed by a new entry; Period: open→closed (reopen with logged override); RulePack: draft→active→retired.

**Persistence/transactions:** `db2` tables plus append-only journal. `db_begin`/`db_commit`/`db_rollback` exist in v1.2.0, but **OS-level atomic write is unverified**. Because durability is critical for a ledger, atomicity must be explicitly validated before production readiness; checkpoints conservative and replay reconciliation mandatory. No unsupported durability claim.

**Backup/migration/IO/audit:** Stop service, copy store and journal as pair, restore by replacing both and verify by recomputing trial balance; no online snapshot. Explicit version integers with adjacent-version Karkain migration routines; no pre-v1.2.0 schema. Deterministic flat format and simple `json` extraction. Every posting, override and rule-pack change appends an AuditEntry with actor, command and timestamp (once `time` is verified); hash chaining planned.

## F. Karkain implementation architecture

**Capabilities (status per register):** Compiler v1.2.0 VERIFIED; lint VERIFIED; C-free native Windows backend VERIFIED with Go engine (default `kcc` BLOCKED — environment); `db` PRESENT/PARTIAL (`db2` subset, transactions present); `json` PARTIAL; `io`/`crypto`/`encoding`/`collections`/`generics` PRESENT; `async`/`net`/`http` PRESENT (building blocks/cooperative); `filesystem` PARTIAL; `system`/`env`/`process`/`time` PARTIAL shell wrappers needing probes. No GUI module.

**Runtime/packaging:** Karkain v1.2.0 only, no compiler modification; Go front-end with C-free native target. Required ecosystem components: Inventory with a valuation-contract version and ERP for summaries, which must exist before integration work. One native executable in a directory package with default config (including rule-pack label text) and empty data directory; no machine-specific paths.

**Blockers:** (1) GUI toolkit BLOCKER — CLI/TUI only, GUI a blocked planned layer; (2) OS atomic write PARTIAL — ledger durability must be validated, never claimed; (3) full JSON DOM PARTIAL — flat formats; (4) `system`/`env`/`time` PARTIAL — probe before posting-date claims; (5) TLS BLOCKER — no bank feeds/external tax services; (6) concurrent HTTP framework BLOCKED — minimal sequential service. Each recorded, never resolved by changing Karkain or using another language.

**Native/FFI:** None for MVP; any future FFI must be a justified Karkain-compatible native interface, never a replacement language.

## G. Non-functional requirements

Targets give workload and measurement method; no arbitrary numbers asserted. Practical machine ~4 GB RAM.

| Area | Target / method |
|---|---|
| Performance | Post 1,000 entries, time with `bench`, expect linear scaling |
| Memory/CPU | Profile posting with `prof`; stay within ~4 GB budget |
| Scalability | 10× entries, record time ratio; document `db2` ceiling |
| Reliability | Inject unbalanced entries and abrupt stop; replay yields identical trial balance |
| Security/privacy | Role separation (post vs override), no secret logging, no telemetry; not a certification |
| Portability | Cross-build Linux; execution UNVERIFIED on this host |
| Maintainability/accessibility | Interfaces documented and unit-tested; text-only keyboard TUI, redirectable output |
| Observability/recovery | Run id, posting counters, audit entries; kill mid-posting, no partial entries |

## H. Compatibility and portability

Windows x86-64 toolchain VERIFIED; Linux cross-target output PRESENT but execution NOT VERIFIED; macOS NOT VERIFIED. All source targets v1.2.0 with Go front-end/C-free backend; default `kcc` unusable here (environment blocker). Native executable plus directory package. Install/uninstall by directory operations after export. Adjacent-version config/data/rule-pack migration; export before upgrade. Backward compatibility not required before v1.2.0. Upgrade/rollback via stop, backup pair, replace, migrate, verify by trial balance; restore on failure. Unsupported: GUI, TLS/bank feeds, external SQL, concurrent HTTP server, non-Windows execution verification, and any legal-compliance claim.

## I. Testing strategy

All tests **PLANNED**; none have been run for this application.

- **Unit:** Balance validation, reversing entries, trial-balance math, rule-pack labelling — deterministic.
- **Integration:** `db2` posting + journal replay + valuation import — balance identity after restart.
- **E2E:** Post-to-close-to-export with stub Inventory valuation — expected snapshots.
- **Platform-specific:** Windows native compile-and-run exit 0; other platforms cross-build only.
- **Security:** Role separation and no secret logging — unauthorized override rejected.
- **Performance:** Posting batch with `bench` — measured, documented scaling.
- **Regression/compatibility:** Replay journals and migrate versions — identical trial balances, lossless.
- **Actual-compiled-executable:** Karkain v1.2.0 build, run native binary — expected stdout.
- **Failure/recovery:** Kill mid-posting — no partial/duplicate entries; audit intact.

## J. Implementation roadmap

**MVP:** Chart of accounts, balanced posting, trial balance, rule-pack structure with mandatory label, CLI queries. **Core:** Period close, P&L/balance-sheet summaries, partial-JSON IO, roles/audit, backup/restore, valuation consumption. **Advanced:** Hash-chained audit, replay tooling, multiple rule packs, ERP publishing, minimal local service. **Production-readiness:** Explicit durability validation, observability, published limits, cross-target builds, full suite.

**Dependencies/gates:** Accounts before posting; posting before balances; balances before reports; valuation contract before integration; durability validation before production readiness. Each phase exits only on passing tests and a verified actual compiled executable; no legal-compliance claim is ever a gate outcome. No dates.

**Risks/mitigations:** Atomic write unverified — checkpoints and replay reconciliation; PARTIAL JSON — flat formats; `time`/`system` PARTIAL — probe before date claims; valuation contract undefined — define first; GUI BLOCKED — CLI/TUI only. No completion claims.
