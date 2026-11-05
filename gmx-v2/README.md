# GMX V2 (gmx-synthetics) — Source Review

**Date:** October 27–28, 2026  
**Target:** GMX V2 synthetics protocol  
**Public repository:** [gmx-io/gmx-synthetics](https://github.com/gmx-io/gmx-synthetics)  
**Commit reviewed:** [`a85ea3491c19c93bb4b5a002d9b358fb769b7849`](https://github.com/gmx-io/gmx-synthetics/commit/a85ea3491c19c93bb4b5a002d9b358fb769b7849) (2026-07-31)  
**Public bug bounty scope:** reviewed against publicly listed GMX V2 / synthetics surfaces

Prior public audits named for context (not re-quoted): **Guardian**, **Quantstamp**, **ABDK**.

---

## Approach

Multi-pass **static source review** of the public `gmx-synthetics` tree at the commit above. Work was organized into three focused passes over handlers, routers, accounting, and oracle paths. Previously published audit findings and documented design choices were used as a duplicate filter; only novel, in-scope issues would have been recorded. No live network was contacted.

---

## Areas checked

### Pass 1 — ExternalHandler, order creation, funding claims

- **ExternalHandler / callback surface:** callback and external-call entry points relative to handler/controller gates; no mid-state window comparable to the historical V1 OrderBook → Vault reentrancy class.
- **Order creation paths:** execution-fee and transfer-in accounting on create-order / related vault recording; keeper / controller gating on execute paths.
- **Funding-claim paths:** claim funding fees follow checks-effects-interactions with post-balance validation; no novel drain of claimable funding identified in the reviewed code.

### Pass 2 — Multicall value, subaccounts, GLV, multichain, reentrancy, funding shortfall

- **PayableMulticall `msg.value` reuse:** documented reuse across delegatecall steps; routers do not spend via raw `msg.value` — wrap / send paths use the contract’s actual ETH balance and explicit amounts; order/deposit creation takes execution fees via recorded transfer-in, not `msg.value`.
- **Subaccount `expiresAt` / `maxAllowedCount`:** expiry and cumulative count enforced on every subaccount entry (including relay and multichain variants); signed approvals bind chain id, deadline, and sequential nonce; no bypass to act after expiry or over the allowed count.
- **GLV deposit / withdraw / shift:** GLV valuation uses internal token balances (donation-resistant); first-deposit guards; withdrawal burns and syncs before inner market withdrawal; shift is keeper-only.
- **Multichain routers:** LayerZero compose validates enabled provider and endpoint; follow-up actions require the account’s signature; relay digests are single-use and domain-separated by source chain and router — **no cross-chain or cross-router replay**; vault credit/record sites are atomic so pending bridged funds cannot be stolen via front-running bridge-in; bridge-out enforces account == receiver where required.
- **GlobalReentrancyGuard:** DataStore-backed global non-reentrancy on exchange handlers **closes the July 2025 V1 reentrancy class** (local guard that did not cover cross-contract reentry). Router-local guards were checked for residual callback reentry; claims and withdrawals finalize state before callbacks, with no V1-style mid-state window.
- **Funding-fee shortfall:** insufficient collateral to cover funding is deliberate (emits and continues under documented rules); claimable-funding vs market-token balance edge after late liquidations is documented design and covered by a **2023** Guardian audit — not treated as a novel issue.

### Pass 3 — JIT, fees, price feeds, instant swaps

- **JIT orders:** execute path is keeper-gated with global non-reentrancy and oracle prices; simulation path always terminates in the expected simulation revert; order/shift validation constraints hold for the reviewed surface.
- **Fee swap / distribution:** fee-swap helper has no live callers in-tree; fee distributor state changes are keeper / reader / timelock gated; buyback uses oracle prices with max-age and impact caps.
- **Price-feed validation:** oracle core enforces provider enablement, atomic vs expected provider rules, max age, and reference-price checks; Chainlink feed / data-stream, GM, and Edge providers validate feed identity, bid/ask, multipliers, and signer constraints as applicable.
- **Instant-swap / atomic pricing:** atomic withdrawal and relay fee-swap paths use atomic price setters with feature flags and fee factors; residual pricing-choice risks map to already-published audit items rather than novel bugs.

---

## Result

**No exploitable issues identified in the reviewed scope.**

---

## Limitations

- Source review only; no live-network testing, forking of production state, or transaction broadcast.
- Review covered the three passes above at a single public commit; not every contract in the repository received equal depth.
- Deployed bytecode on Arbitrum / Avalanche was not independently verified against the GitHub commit reviewed here.
