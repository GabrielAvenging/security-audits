# Security Audits

Public collection of defensive secure source-code reviews.

| Audit | Target | Commit / range | Date |
|---|---|---|---|
| [MoneroOcean/nodejs-pool](./moneroocean-nodejs-pool/) | https://github.com/MoneroOcean/nodejs-pool | `d56fdbc873fd9d82abce52c6f78fe5cef80dd86c` | 2026-10-27 |
| [GMX V2 (gmx-synthetics)](./gmx-v2/) | https://github.com/gmx-io/gmx-synthetics | `a85ea3491c19c93bb4b5a002d9b358fb769b7849` | 2026-10-27–28 |
| [Twyne](./twyne/) | https://github.com/0xTwyne/twyne-contracts-v1 | `beb4bf15a70b1a8abcf7e17e8a08f556ee5f84ab` | 2026-10-28–29 |
| [Enzyme Finance Protocol Onyx](./enzyme-onyx/) | https://github.com/enzymefinance/protocol-onyx | `b98576f` → in-scope pins `61ef53d` / `7b48d24` | 2026-11-05 |
| [OnRe Solana program](./onre/) | https://github.com/onre-finance/onre-sol | `1dbc6ec` → `c049083` | 2026-11-05 |

## Notes

- Reviews are **static source-code analysis only**, except where a write-up explicitly records read-only deployment checks or local-fork simulation. No live hosts were scanned, probed, or written to.
- Where a review is a post-audit diff pass, only code between the last audited (or inferred-audited) commit and the current in-scope commit was examined in depth.
- Findings and recommended fixes, when present, are intended for operators and maintainers hardening their deployments.
