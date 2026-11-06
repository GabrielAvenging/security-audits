# dHEDGE V2 — Independent Limited Review

**Date:** November 5, 2026  
**Target:** dHEDGE V2 public contracts  
**Public repository:** [dhedge/V2-Public](https://github.com/dhedge/V2-Public)  
**Reviewed tree:** [`ec35e7e8749b551c49855af40c81daa766a15852`](https://github.com/dhedge/V2-Public/tree/ec35e7e8749b551c49855af40c81daa766a15852)  
**Public program page:** [dHEDGE on Immunefi](https://immunefi.com/bug-bounty/dhedge/information/)

Independent limited review of selected pool entry, exit, fee, and nested-vault guard logic at the pinned tree above.

---

## Approach

Source review of the surfaces below, plus existing offline unit tests that completed. Deployment-address notes came from source and build artifacts only. Live bytecode was not verified. No live system was contacted.

---

## Areas checked

- **PoolLogic:** deposits, withdrawals, and entry/exit fees.
- **NestedVaultAssetGuard.**

---

## Recorded checks

Existing offline unit tests that completed:

| Suite | Result |
|---|---|
| Withdrawals | 2/2 passed |
| Entry-exit fee | 15/15 passed |
| Deposits | 17/17 passed |
| Nested vault guard | 4/4 passed |
| **Completed total** | **38 passed** |

An additional custom probe is incomplete. Setup stopped on an allowance precondition in the test setup. That incomplete probe is not a protocol finding and is not counted among the 38 passes.

The completed suite results are assertions in those existing tests. They are not validation of a bounty bug.

The fresh clone, dependencies, and test directory were deleted, and their absence was verified.

---

## Result

**No validated reportable finding in this bounded review; production applicability not established.**

---

## Limitations

- Bounded to the PoolLogic and NestedVaultAssetGuard surfaces named above, at a single public tree.
- Deployment-address correspondence is from source and build artifacts only. Live bytecode was not verified.
- The additional custom probe did not complete. It is excluded from the pass count.
- The fresh clone, dependencies, and test directory were removed, and their absence was verified.
- This page is an independent limited review of that slice. It records no commissioned engagement, no comprehensive-audit claim, no project endorsement, no production-safety claim, and no accepted bounty finding.
