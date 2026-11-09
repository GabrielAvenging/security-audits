# Raydium CP Swap — Bounded Review

**Date:** November 9, 2026  
**Target:** Raydium constant-product swap  
**Public repository:** [raydium-io/raydium-cp-swap](https://github.com/raydium-io/raydium-cp-swap)  
**Commit reviewed:** [`3d69122790e97cd6aff24e8ba95940793773f58c`](https://github.com/raydium-io/raydium-cp-swap/commit/3d69122790e97cd6aff24e8ba95940793773f58c)

Bounded review of swap math, liquidity, fees, and account validation at the pinned commit. Local library tests only.

---

## Coverage

Swap math and rounding, deposit and withdraw LP, fee, protocol-fee, and creator-fee accounting, and account validation.

---

## Hypotheses

Tested at that level only. "Refuted" means these checks did not validate a reportable finding.

| Hypothesis | Result |
|---|---|
| A reportable defect in swap math or rounding | Tested and refuted in `cargo test --lib` |
| A reportable defect in deposit or withdraw LP | Tested and refuted in `cargo test --lib` |
| A reportable defect in fee, protocol-fee, or creator-fee accounting | Tested and refuted in `cargo test --lib` |
| A reportable defect in account validation | Tested and refuted in `cargo test --lib` |

---

## Recorded checks

Command: `cargo test --lib`

| Group | Result |
|---|---|
| Existing | 21 passed |
| Added | 9 passed |
| **Total** | **30 passed, 0 failed** |

The integration TypeScript suite was not run.

---

## Hardening note (out of scope)

`create_amm_config` does not apply the rate-sum bound that `update_amm_config` enforces. The note is admin-only and out of scope for this pass. It is not recorded as a validated reportable finding.

---

## Result

**No validated reportable finding in this bounded review; source-only unless noted; nothing submitted.**

No live-network or fork exception is recorded for this pass.

---

## Limitations

- Evidence is `cargo test --lib` at one commit: 30 passed, 0 failed. The integration TypeScript suite was not run.
- The admin-only create-config bound is a hardening note, not a submitted finding.
- This pass does not verify deployed bytecode and does not record a bounty acceptance.
