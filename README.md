# Security Audits

Public collection of defensive secure source-code reviews, including independent limited reviews, recorded baseline tests, and bounded reviews.

| Audit | Target | Commit / range | Date |
|---|---|---|---|
| [MoneroOcean/nodejs-pool](./moneroocean-nodejs-pool/) | https://github.com/MoneroOcean/nodejs-pool | `d56fdbc873fd9d82abce52c6f78fe5cef80dd86c` | 2026-10-27 |
| [GMX V2 (gmx-synthetics)](./gmx-v2/) | https://github.com/gmx-io/gmx-synthetics | `a85ea3491c19c93bb4b5a002d9b358fb769b7849` | 2026-10-27–28 |
| [Twyne](./twyne/) | https://github.com/0xTwyne/twyne-contracts-v1 | `beb4bf15a70b1a8abcf7e17e8a08f556ee5f84ab` | 2026-10-28–29 |
| [Enzyme Finance Protocol Onyx](./enzyme-onyx/) | https://github.com/enzymefinance/protocol-onyx | `b98576f` → in-scope pins `61ef53d` / `7b48d24` | 2026-11-05 |
| [OnRe Solana program](./onre/) | https://github.com/onre-finance/onre-sol | `1dbc6ec` → `c049083` | 2026-11-05 |
| [Origin Protocol ARM (limited review)](./origin-arm/) | https://github.com/OriginProtocol/arm-oeth | `3e36db1` | 2026-11-05 |
| [dHEDGE V2 (limited review)](./dhedge/) | https://github.com/dhedge/V2-Public | `ec35e7e` | 2026-11-05 |
| [GammaSwap DeltaSwap V2 (limited review)](./gammaswap/) | https://github.com/gammaswap/v2-deltaswap | `b20ce4d` | 2026-11-05 |
| [Nexus Mutual (limited review)](./nexus-mutual/) | https://github.com/NexusMutual/smart-contracts | `9e88562` on `release-candidate` | 2026-11-05 |
| [Flux Finance (limited review)](./flux-finance/) | https://github.com/flux-finance/contracts | docs review unpinned; setup inspection `05bba79` (tests not executed) | 2026-11-06 |
| [deBridge contracts-v1 (baseline test)](./debridge/) | https://github.com/debridge-finance/debridge-contracts-v1 | `2e9f9c7` | 2026-11-06 |
| [deBridge contracts-v1 (Nov 9 follow-up)](./debridge/#nov-9-2026-follow-up-passes) | https://github.com/debridge-finance/debridge-contracts-v1 | `2e9f9c7` | 2026-11-09 |
| [Raydium CP swap (bounded review)](./raydium-cp-swap/) | https://github.com/raydium-io/raydium-cp-swap | `3d69122` | 2026-11-09 |
| [Enzyme Onyx (Nov 9 bounded review)](./enzyme-onyx/#november-9-2026-bounded-review) | https://github.com/enzymefinance/protocol-onyx | `d8a522d` | 2026-11-09 |
| [Orca Whirlpools (bounded review)](./orca-whirlpools/) | https://github.com/orca-so/whirlpools | `cc683ec` | 2026-11-09 |
| [xORCA (bounded review)](./xorca/) | https://github.com/orca-so/xorca | `05fe66b` | 2026-11-09 |
| [tBTC v2 (bounded review)](./tbtc-v2/) | https://github.com/threshold-network/tbtc-v2 | `5d2af6c` | 2026-11-09 |
| [Twyne (Nov 9 follow-up)](./twyne/#november-9-2026-follow-up) | https://github.com/0xTwyne/twyne-contracts-v1 | `beb4bf1` | 2026-11-09 |
| [Lista DAO Moolah (bounded review)](./lista-dao/) | https://github.com/lista-dao/moolah | `abf18896` → `284fb57` | 2026-11-20 |
| [DeFi Saver v3 strategy partial-execution (bounded follow-up)](./defisaver-nov20/) | https://github.com/defisaver/defisaver-v3-contracts | `cae44d54` → `c14d8e35` | 2026-11-20 |
| [GMTrade / gmx-solana (bounded review)](./gmtrade/) | https://github.com/gmsol-labs/gmx-solana | `d81a3e69` → `3e0a4115` | 2026-11-22 |
| [Sei Chain (bounded review)](./sei/) | https://github.com/sei-protocol/sei-chain | `f70ab582` → `1d3b8021` | 2026-11-22 |
| [Hermetica hBTC trading v2 (bounded review)](./hermetica/) | https://github.com/hermetica-fi/hermetica-contracts | `5e13e43` → `4e473be` | 2026-11-22 |

## Notes

- Reviews are **static source-code analysis only**, except where a write-up explicitly records read-only deployment checks, local-fork simulation, offline tests, or a read-only fork over a public RPC. No transactions were sent to live networks.
- Where a review is a post-audit diff pass, only code between the last audited (or inferred-audited) commit and the current in-scope commit was examined in depth.
- Findings and recommended fixes, when present, are intended for operators and maintainers hardening their deployments.
- Origin ARM, dHEDGE, GammaSwap, and Nexus Mutual (2026-11-05) and Flux Finance (2026-11-06) are independent limited reviews. The status line for each is: "No validated reportable finding in this bounded review; production applicability not established." Read the linked write-up for the exact slice, recorded checks, and limits. They are not commissioned audits, coverage certificates, project endorsements, or accepted bounty reports.
- The Flux Finance row splits two passes. The documented review pinned no commit. The later setup inspection is pinned at `05bba79` and is labeled "test setup attempted, not executed (fork-dependent fixtures)." That pin does not cover the unpinned documented review, and no Flux test is recorded as passed or failed.
- deBridge contracts-v1 (2026-11-06) is a baseline run of the existing `test/00_SignatureVerifier.test.js` suite: 16 passing, 0 failing. It is not a security review and does not record a vulnerability finding. The November 9 section on that page is a separate bounded review.
- November 9, 2026 rows are bounded reviews. Each states: "No validated reportable finding in this bounded review; source-only unless noted; nothing submitted." Twyne's November 9 pass records a read-only mainnet fork over a public RPC and no transactions. Skipped fork tests, suites that did not execute, and the Raydium admin-only hardening note are described in the linked write-ups.
- November 20 and 22, 2026 rows are bounded reviews. Each states: "No validated reportable finding in this bounded review; source-only, or source-only plus local tests as recorded; nothing submitted." Lista DAO recorded 152 local Foundry tests passing. Sei recorded 2,014 local Go tests passing (7 skipped). DeFi Saver (Nov 20) and Hermetica tests were not run because they need a live-network fork. GMTrade program tests were not run because they need a live test-network connection.
- The DeFi Saver November 20 write-up is a later range (`cae44d54` → `c14d8e35`) than the CurveUsd/LlamaLend swapper review in [pull request #4](https://github.com/GabrielAvenging/security-audits/pull/4) (`26146bf1` → `cae44d54`, still open). This index does not duplicate that earlier write-up; it will be listed when that pull request is merged.

## Services

Independent, source-focused security research:

- **Pay-per-bug research:** $150 per valid bug, after the project confirms the issue is in scope and is not a duplicate. Work stays on code the client provides or already publishes. Nothing is tested on live production systems unless the client explicitly authorizes a written scope.
- **Fixed-price security reviews:** quoted per project. A typical engagement is a scoped source review (and local tests where they can run offline), with a written report covering methodology, findings, and recommended fixes. Timeline and price depend on language, size, and whether a prior audit already covers the same code.

To inquire, open a GitHub issue on this repository or email the address on the [GitHub profile](https://github.com/GabrielAvenging).

## Tooling

Reviews use a local, user-level toolchain. Tools are pointed at checkouts on disk only, not at live hosts, RPC endpoints, or networks.

Typical inventory:

- **Smart contracts:** Slither, Aderyn, Semgrep (including Decurity's smart-contract rules), Mythril, Halmos, Echidna, Medusa, Wake, Foundry, Surya, and heimdall-rs (EVM decompiler, used only on local bytecode).
- **Rust, Go, C/C++, Python, and JavaScript:** rustc/cargo with clippy, cargo-audit, cargo-fuzz, and cargo-geiger; gosec, govulncheck, and staticcheck; clang with ASan/UBSan/libFuzzer, cppcheck, and flawfinder; bandit, pip-audit, and Atheris; njsscan.
- **Secrets and dependencies:** gitleaks, trufflehog (filesystem, no live verification), osv-scanner, syft, grype, and trivy (filesystem or local-repo mode only).
- **Deeper analysis:** CodeQL (OSI-approved open-source code or academic research only), Certora Prover (local formal specs), Ghidra and rizin (local binaries only), and Jazzer (Java fuzzing).
- **Authorized web testing helpers:** OWASP ZAP and ffuf are installed for manual, in-scope testing that a client has explicitly authorized. They are not used against any host by default.
- **Harness helpers:** selected [OSS-Fuzz](https://github.com/google/oss-fuzz) infra scripts for building local libFuzzer targets.

A machine-specific install list, versions, and skip reasons live with the scanner used for these reviews and are not part of this repository.
