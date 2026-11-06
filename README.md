# Security Audits

Public collection of defensive secure source-code reviews, including independent limited reviews.

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

## Notes

- Reviews are **static source-code analysis only**, except where a write-up explicitly records read-only deployment checks, local-fork simulation, or offline tests. No live hosts were scanned, probed, or written to.
- Where a review is a post-audit diff pass, only code between the last audited (or inferred-audited) commit and the current in-scope commit was examined in depth.
- Findings and recommended fixes, when present, are intended for operators and maintainers hardening their deployments.
- Origin ARM, dHEDGE, GammaSwap, and Nexus Mutual (2026-11-05) and Flux Finance (2026-11-06) are independent limited reviews. The status line for each is: "No validated reportable finding in this bounded review; production applicability not established." Read the linked write-up for the exact slice, recorded checks, and limits. They are not commissioned audits, coverage certificates, project endorsements, or accepted bounty reports.
- The Flux Finance row splits two passes. The documented review pinned no commit. The later setup inspection is pinned at `05bba79` and is labeled "test setup attempted, not executed (fork-dependent fixtures)." That pin does not cover the unpinned documented review, and no Flux test is recorded as passed or failed.
