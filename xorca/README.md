# xORCA — Bounded Review

**Date:** November 9, 2026  
**Target:** xORCA (`xorca`)  
**Public repository:** [orca-so/xorca](https://github.com/orca-so/xorca)  
**Reviewed tree:** [`05fe66b67c95271cb07aa42c83b9fdfd8a39b936`](https://github.com/orca-so/xorca/tree/05fe66b67c95271cb07aa42c83b9fdfd8a39b936)

Bounded review of stake, exchange-rate, and account-validation behavior at the pinned tree.

The local toolchain was pinned to Rust 1.86.0 because a dependency is incompatible with newer stable Rust. The dependency name is not recorded here.

---

## Coverage

Stake and unstake, exchange rate and virtual offset, inflation and donation, escrow, cooldown and pending withdraw, and account validation.

---

## Hypotheses

Tested at that level only. "Refuted" means these checks did not validate a reportable finding.

| Hypothesis | Result |
|---|---|
| A reportable defect in stake or unstake | Tested and refuted in this pass |
| A reportable defect in the exchange rate or virtual offset | Tested and refuted in this pass |
| A reportable defect in inflation or donation handling | Tested and refuted in this pass |
| A reportable defect in escrow | Tested and refuted in this pass |
| A reportable defect in cooldown or pending withdraw | Tested and refuted in this pass |
| A reportable defect in account validation | Tested and refuted in this pass |

---

## Recorded checks

| Outcome | Count |
|---|---|
| Passed | 190 |
| Failed | 0 |

---

## Result

**No validated reportable finding in this bounded review; source-only unless noted; nothing submitted.**

No live-network or fork exception is recorded for this pass.

---

## Limitations

- Bounded to the surfaces named above at one tree.
- Recorded result is 190 passed and 0 failed. This note does not split that total into existing and added tests.
- Rust was pinned to 1.86.0 for the dependency incompatibility above. A newer stable compiler was not used.
- This pass does not verify deployed programs and does not record a bounty acceptance.
