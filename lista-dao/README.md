# Lista DAO Moolah bounded review

**Status: No validated reportable finding in this bounded review; source-only plus local Foundry tests; nothing submitted.**

- **Target:** https://github.com/lista-dao/moolah
- **Commit range:** `abf18896` → `284fb57`
- **Date:** 2026-11-20
- **Program:** [Immunefi Lista DAO](https://immunefi.com/bug-bounty/listadao/information/) (page last updated 20 November 2026; Primacy of Impact is off)

## Scope

In-scope assets include Moolah and PublicLiquidator (plus providers, distributors, and the CDP stack as listed on the program page). LendingBroker, BrokerInterestRelayer, CreditBrokerInterestRelayer, BrokerInterestLockBuffer, and the non-Public liquidator contracts are not listed as in-scope assets. Privileged-address attacks are out of scope. Audit-folder reports are treated as known issues.

## Methodology

1. Scope and known-issue check against the Immunefi program page fetched for this review.
2. Diff review of the relayer queue-guard and related broker / liquidator pause changes in this range.
3. Duplicate check against the public HashDit RelayerVaultQueueGuard report (2026-11-10), which already documents the relayer issues in this area.
4. Local Foundry tests, no fork or RPC.

## Tests

Ran at `284fb57` with a temporary Foundry test directory so unrelated unfetched submodules were not required.

| Suite | Result |
|---|---|
| LiquidatorPause | 10 passed |
| BrokerInterestRelayerQueueGuard | 6 passed |
| LendingBroker.t.sol | 136 passed |
| **Total** | **152 passed, 0 failed** |

No temporary tests were added to the project. The checkout used for the test run was deleted afterwards.

## Result

**No payable finding.** Changes in the in-scope PublicLiquidator pause path are privileged-role operations. Relayer issues in this range are already covered by the published HashDit report and were treated as duplicates. No new fund-loss path for an unprivileged caller was identified.

## Limits

- Bounded diff review, not a full audit of Lista DAO or Moolah.
- Contracts not listed as in-scope assets were reviewed only far enough to confirm scope and duplicates.
- No on-chain or fork verification.
