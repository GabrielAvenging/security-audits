# Sei Chain bounded review

**Status: No validated reportable finding in this bounded review; source-only plus local unit tests; nothing submitted.**

- **Target:** https://github.com/sei-protocol/sei-chain
- **Commit range:** `f70ab582` → `1d3b8021` (13 commits on `main`; about +10.7k / −4.9k lines)
- **Date:** 2026-11-22
- **Program:** [Immunefi Sei](https://immunefi.com/bug-bounty/sei/information/) (KYC required; PoC required at every severity)

## Scope

The review stayed in `sei-chain` (in scope). Nested `sei-db` and `sei-tendermint` trees are covered through that asset. `go-ethereum` and `sei-js` were not in this range.

Immunefi excludes Autobahn multi-proposer consensus, Giga functionality other than the Giga executor and FlatKV, Giga-only FlatKV paths, and StateSync peer/snapshot attacks. Direct-access crashes on non-EVM endpoints are also excluded; a crash of default RPC nodes via direct unauthenticated EVM JSON-RPC remains in scope (Medium).

Latest release at review time was `v6.8.0` (2026-11-17). None of the 13 commits are in that tag or in `CHANGELOG.md`. This range is on `main` only and was treated as not yet released.

## Methodology

1. Scope and impact check against the Immunefi information and scope pages fetched for this review.
2. Commit-by-commit triage of the 13-commit range, with a deeper read of default-node EVM JSON-RPC and receipt-store changes. Giga/Autobahn-gated and out-of-scope packages were identified and not treated as payable paths.
3. Duplicate check against the public tokenfactory known issue and the 2023 Oak Security audits (they predate this evmrpc / FlatKV / Autobahn / data-lake code).
4. Local unit tests only. No scanners against live systems, no RPC to real nodes, and no transactions.

## Tests

Go 1.27.4, in-process only (statesync S3 tests use `httptest`).

| Package | Result |
|---|---|
| `./sei-db/state_db/sc/flatkv/` | 738 passed, 0 failed |
| `./sei-db/receipt_db/...` | 176 passed, 0 failed |
| `./sei-db/statesync/` | 68 passed, 0 failed |
| `./evmrpc/` | 820 passed, 0 failed, 7 skipped |
| `./x/evm/keeper/` | 212 passed, 0 failed |
| **Total** | **2,014 passed, 0 failed, 7 skipped** |

The seven evmrpc skips are pre-existing `t.Skip()` cases plus one already-initialized global pool; they are not network skips. No temporary test files were added. Giga, Autobahn, and `sei-db/bootstrap` suites were not run as primary targets.

## Result

**No payable finding.** The only default-node EVM JSON-RPC behavior change in this range reads data the node already stored for a transaction it executed, and it does not create a new way to crash a default node or move funds. Larger storage, statesync, and validator/consensus changes are off by default and are excluded by the program. The code is not in the latest release.

The temporary checkout and test logs were deleted after the review.

## Limits

- Bounded diff review of one `sei-chain` range, not a full audit of Sei.
- Deployment and mainnet behavior were not checked on-chain.
- Out-of-scope Giga/Autobahn paths were not reviewed as payable surface.
