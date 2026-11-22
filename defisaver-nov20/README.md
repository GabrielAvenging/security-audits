# DeFi Saver v3 strategy partial-execution follow-up (bounded review)

**Status: No validated reportable finding in this bounded review; source-only; nothing submitted.**

- **Target:** https://github.com/defisaver/defisaver-v3-contracts
- **Commit range:** `cae44d54` → `c14d8e35`
- **Date:** 2026-11-20
- **Program:** [Immunefi DeFi Saver](https://immunefi.com/bug-bounty/defisaver/information/) (assets in scope: "Defi Saver V3 (excluding the 'mocks' and 'views' folders)"; privileged-access attacks out of scope)

This is a later range than the CurveUsd/LlamaLend swapper review in [pull request #4](https://github.com/GabrielAvenging/security-audits/pull/4) (`26146bf1` → `cae44d54`). It does not replace that write-up.

## Scope

Reviewed the strategy partial-execution changes in this range: ToggleSub, UpdateSub, RecipeExecutor (`executeRecipeFromStrategy`, calldata-length validation, partial-execution helpers), StrategyExecutorCommon, StrategyPartialExecutionStorage, related interfaces and IDs, and the unchanged `StrategyExecutor.executeStrategy`. Instant-close automation in this range is strategies-spec JavaScript only, with no Solidity. TriggerView changes in `views/` are out of scope.

Repo metadata in the range leaves RecipeExecutor, ToggleSub, and UpdateSub addresses unchanged; the new logic was treated as likely not yet deployed. That was not verified on-chain.

## Methodology

1. Scope check against the Immunefi program rules fetched on 2026-11-20.
2. Line-by-line review of the new partial-execution start/end flow, who can enable or clear it, and whether a third party can force or hijack another user's strategy execution.
3. The local watcher reported 0 High / 0 Medium hits (scan dated 2026-11-19).

## Tests

All relevant Foundry tests call `forkFromEnv` (Tenderly mainnet fork). They were **not run**, under the no-RPC rule. No temporary tests were added.

## Result

**No payable finding.** Starting partial execution requires bot authorization and the subscription wallet, and the approval is cleared before actions run. Ending it is limited to the owner wallet or the admin vault owner. Remaining gaps either depend on DeFi Saver's own bot (privileged, out of scope) or only let an owner affect their own subscription.

## Limits

- Bounded diff review of this later range only, not a full audit and not a substitute for the earlier swapper review.
- Program tests that need a mainnet fork were not run.
- Deployment was not verified on-chain.
