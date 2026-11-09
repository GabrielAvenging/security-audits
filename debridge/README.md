# deBridge contracts-v1 — Baseline Test

**Date:** November 6, 2026  
**Target:** deBridge contracts-v1  
**Public repository:** [debridge-finance/debridge-contracts-v1](https://github.com/debridge-finance/debridge-contracts-v1)  
**Commit tested:** [`2e9f9c73438738310b5293cec4cffe25741e9b1f`](https://github.com/debridge-finance/debridge-contracts-v1/commit/2e9f9c73438738310b5293cec4cffe25741e9b1f)  
**Public program page:** [deBridge on Immunefi](https://immunefi.com/bug-bounty/debridge/information/)

Baseline correctness run of one existing Hardhat suite at the pinned commit. This page records that test run only. No vulnerability assessment was performed in this pass.

---

## Approach

An earlier attempt stopped because Yarn was missing and executed zero tests.

The run recorded here installed Yarn 1.22.22 locally inside a disposable test directory, with caches isolated to that directory. A frozen-lockfile install failed because the lockfile needed an update under the runtime in use, so the run continued with a local `yarn install`. Solc 0.8.17 compilation succeeded. The suite then ran in-process under Hardhat.

No chain RPC or fork was used. No externally deployed contract was tested. No real wallets, funds, or live-network transactions were used.

---

## Suite

Existing file: `test/00_SignatureVerifier.test.js`

Recorded command: `./node_modules/.bin/hardhat test test/00_SignatureVerifier.test.js`

The existing cases cover initial values, admin, oracle configuration, and access control, using local `upgrades.deployProxy` fixtures.

An optional `sqlite3` native-build warning occurred. It did not stop the suite.

---

## Result

| Outcome | Count |
|---|---|
| Passing | 16 |
| Failing | 0 |

Test execution took about 2 seconds. That figure is the test run itself, not the full setup time.

**16 passing, 0 failing.** This is baseline correctness testing of the existing suite above.

No submission or disclosure was made, and nothing was written to the target repository. The temporary checkout, local package manager, dependencies, and caches were deleted, and their absence was independently verified. No dependency or lockfile change was pushed upstream.

---

## Limitations

- One existing suite only. Node and Hardhat versions were not recorded for this run, so they are not stated here.
- In-process Hardhat only: no chain RPC, no fork, no external deployed-contract testing, and no real wallets, funds, or live-network transactions.
- Dependency resolution and the local lockfile changed for this run. It is not an exact frozen-lockfile reproduction, even though the target source commit was pinned.
- The earlier missing-Yarn attempt executed zero tests and is separate from the 16 passing result.
- This page is not an independent security audit, not comprehensive coverage, not a vulnerability search, not proof of production security, and not a validated bounty finding. No vulnerability assessment was performed in this pass.

---

## Nov 9, 2026 follow-up passes

**Date:** November 9, 2026  
**Same pin:** [`2e9f9c73438738310b5293cec4cffe25741e9b1f`](https://github.com/debridge-finance/debridge-contracts-v1/commit/2e9f9c73438738310b5293cec4cffe25741e9b1f)

Bounded follow-up on the pinned source above. It is separate from the November 6 baseline correctness run. The checks below are local tests on that source.

### (a) Signature confirmation checks

`SignatureVerifier.submit` and DeBridgeGate claim confirmation checks.

| Group | Result |
|---|---|
| Existing | 16 passing |
| Added local tests | 4 passing |
| **Total** | **20 passing** |

| Hypothesis | Result |
|---|---|
| A duplicate signature is accepted | Tested and refuted. The local tests revert the duplicate. |
| A claim below the minimum confirmation count is accepted | Tested and refuted. The local tests reject it. |

### (b) Fee and withdraw accounting

| Group | Result |
|---|---|
| Added local tests | 6 passing |
| LightDebridge | 18 passing |
| **Total** | **24 passing** |

| Hypothesis | Result |
|---|---|
| `withdrawFee` is available to a caller other than the fee proxy | Tested and refuted. The local tests keep it restricted to the fee proxy. |
| Fee math does not conserve the amount | Tested and refuted. The local tests keep the amount conserved. |

### Result

**No validated reportable finding in this bounded review; source-only unless noted; nothing submitted.**

No live-network or fork exception is recorded for these two passes.

### Limitations

- The totals are 20 and 24 for the two passes above. They are not a full-protocol run, and they do not replace the November 6 baseline result of 16 passing and 0 failing.
- These checks do not verify deployed bytecode and do not record a bounty acceptance.
