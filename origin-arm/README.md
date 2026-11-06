# Origin Protocol ARM — Independent Limited Review

**Date:** November 5, 2026  
**Target:** Origin Protocol ARM (`arm-oeth`)  
**Public repository:** [OriginProtocol/arm-oeth](https://github.com/OriginProtocol/arm-oeth)  
**Reviewed tree:** [`3e36db1d95a14edea6e0dae6980e3c068e360dfd`](https://github.com/OriginProtocol/arm-oeth/tree/3e36db1d95a14edea6e0dae6980e3c068e360dfd)  
**Public program page:** [Origin Protocol on Immunefi](https://immunefi.com/bug-bounty/originprotocol/information/)

Independent limited review of selected ARM liquidity controls at the pinned tree above.

---

## Approach

Static reading of selected accounting controls, plus selected existing offline unit suites. The in-scope proxy was identified from repository metadata. Live bytecode was not verified. No live system was contacted.

EthenaARM was read as a reference for the same control family.

---

## Areas checked

- **AbstractARM:** LP redeem, claimable balances, accounting, and reserved-liquidity controls.
- **EthenaARM:** reference for those controls.
- **Scope proxy:** identity taken from repository metadata only.

---

## Recorded checks

- Several selected existing offline unit suites passed. The exact suite count and test count from that pass were not retained, so no count is stated here.
- A documented liquidity / claim timing behavior was investigated offline. That investigation did not identify a novel reportable vulnerability.
- Those suite results and that investigation record what was run. They are not validation of a bounty bug.

The local review checkout, dependencies, and tests were deleted, and their absence was verified.

---

## Result

**No validated reportable finding in this bounded review; production applicability not established.**

---

## Limitations

- Bounded to the AbstractARM surfaces named above, with EthenaARM used as a reference, at a single public tree.
- Live bytecode was not verified. The in-scope proxy came from repository metadata.
- The exact number of passing existing unit tests was not retained.
- The documented liquidity / claim timing behavior was not a novel reportable vulnerability.
- Local review checkout, dependencies, and tests were removed after the pass, and their absence was verified.
- This page is an independent limited review of that slice. It records no commissioned engagement, no comprehensive-audit claim, no project endorsement, no production-safety claim, and no accepted bounty finding.
