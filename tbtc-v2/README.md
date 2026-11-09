# tBTC v2 — Bounded Review

**Date:** November 9, 2026  
**Target:** Threshold tBTC v2  
**Public repository:** [threshold-network/tbtc-v2](https://github.com/threshold-network/tbtc-v2)  
**Reviewed tree:** [`5d2af6cd99e4cbe7f7062921b514737026f43756`](https://github.com/threshold-network/tbtc-v2/tree/5d2af6cd99e4cbe7f7062921b514737026f43756)

Bounded review of deposit, redemption, vault, bank, relay, and fee-rounding behavior at the pinned tree.

---

## Coverage

Deposit and SPV proofs, redemption, fraud, and moving funds, TBTCVault and optimistic minting, Bank, LightRelay, and fee rounding.

---

## Hypotheses

High level only. "Refuted" means the checks that actually ran did not validate a reportable finding. Bridge and Vault suites did not run, so those hypotheses were not execution-tested.

| Hypothesis | Result |
|---|---|
| A reportable defect in deposit or SPV proofs | Not execution-tested. Bridge suite did not run. Source reading did not validate a reportable finding |
| A reportable defect in redemption, fraud, or moving funds | Not execution-tested. Bridge suite did not run. Source reading did not validate a reportable finding |
| A reportable defect in TBTCVault or optimistic minting | Not execution-tested. Vault suite did not run. Source reading did not validate a reportable finding |
| A reportable defect in Bank | In the named coverage. No separate pass count is assigned here |
| A reportable defect in LightRelay | In the named coverage. No separate pass count is assigned here |
| A reportable defect in fee rounding | In the named coverage. No separate pass count is assigned here |

---

## Recorded checks

| Outcome | Count |
|---|---|
| Passing | 543 |
| Failing | 41 |
| Added tests | 9 passing |

All 41 failures were before-hook setup failures: a WalletRegistry OpenZeppelin upgrade-safety check while local deploy fixtures were starting. Because of that, the Bridge and Vault suites did not execute. Deposit and SPV proofs, redemption, fraud, moving funds, TBTCVault, and optimistic minting were reviewed by source reading.

Local setup adjustments for this run, not protocol changes: a `git://` dependency URL was fetched over `https://`, and a Hardhat chainId file was supplied. No path or file contents are recorded here.

---

## Result

**No validated reportable finding in this bounded review; source-only unless noted; nothing submitted.**

No live-network or fork exception is recorded. The Bridge and Vault suites were source reading only.

---

## Limitations

- The recorded suite totals are 543 passing and 41 failing. All 41 are setup failures, not protocol assertions that ran and failed.
- Bridge and Vault behavior in this pass rests on source reading. Those suites did not execute.
- Nine added tests passed. This note does not claim those nine are part of the Bridge or Vault suites.
- Bank, LightRelay, and fee rounding stay inside the named coverage without a separate pass count.
- The `git://` to `https://` fetch and the Hardhat chainId file were local setup adjustments.
- This pass does not verify deployed bytecode and does not record a bounty acceptance.
