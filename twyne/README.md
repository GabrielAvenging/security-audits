# Twyne — Source Review

**Date:** October 28–29, 2026  
**Target:** Twyne collateral vaults / credit protocol (v1)  
**Public repository:** [0xTwyne/twyne-contracts-v1](https://github.com/0xTwyne/twyne-contracts-v1)  
**Commit reviewed:** [`beb4bf15a70b1a8abcf7e17e8a08f556ee5f84ab`](https://github.com/0xTwyne/twyne-contracts-v1/commit/beb4bf15a70b1a8abcf7e17e8a08f556ee5f84ab) (2026-09-09)  
**Public bug bounty scope:** reviewed against publicly listed Ethereum Twyne surfaces

Related public source also consulted for the Aave aToken wrapper path: [0xTwyne/aave-v3-aToken-wrapper](https://github.com/0xTwyne/aave-v3-aToken-wrapper).

Prior public audits named for context (not re-quoted): **yAudit**, **Enigma Dark**.

---

## Approach

Two-pass review:

1. **Static source review** of liquidation math, credit accounting, factory / access control, share pricing, interest timing, price feeds, reentrancy, and the leverage operators added around 2026-10-06.
2. **Read-only deployment check** that live Ethereum contracts match the GitHub source under review, plus **local-fork** stress and fuzz testing (no mainnet or public-testnet interaction; nothing broadcast).

Previously published audit findings and documented known issues were used as a duplicate filter.

---

## Areas checked

### Pass 1 — Core protocol surfaces

- **Liquidation math:** health / incentive regimes (`collateralForBorrower`), rounding and boundary ordering, split math after external liquidation; no underflow or inverted-regime paths identified in the reviewed code.
- **Credit accounting:** reservation and release of intermediate-vault credit (`_handleExcessCredit` / `maxRelease`); withdraw capped by reserved credit; no novel loss of reserved credit identified.
- **Vault setup and access control:** factory deploys and initializes BeaconProxies atomically; implementations disable initializers; borrower gates via EVC; intermediate-vault borrow hook limited to registered collateral vaults; admin allowlists and LTV ramps on `VaultManager`.
- **Share pricing:** collateral vaults are 1:1 non-transferable; intermediate vaults use EVK-style internal cash accounting; Aave wrapper prices via index rather than raw `balanceOf`; donations are not an inflation vector against other users.
- **Interest timing:** debt views include accrued interest; `maxRelease` is min-capped against totals; batch snapshots cache liquidation params, not balances.
- **Price feeds:** Twyne router unit-of-account wiring; wrapper / Morpho oracle paths checked for obvious mis-wiring (third-party oracle data itself out of scope).
- **Reentrancy:** nonReentrant + call-through-EVC on state-changing vault functions; view guards on balance / liquidatability; operators use transient reentrancy guards; flash-loan callbacks only from the operator itself.
- **Leverage operators (2026-10-06):** Euler / Aave leverage (and related deleverage / teleport) flash-loan callback sender checks, borrower authentication via EVC, and swap / min-out ordering — no novel fund-loss path identified.

### Pass 2 — Deployed match, fuzz, expired PT vault

- **Deployed-versus-source (read-only):** verified explorer sources and runtime bytecode comparison (immutables / metadata masked) against local builds of the reviewed commit — live Ethereum Twyne contracts under review **match** the GitHub source.
- **Local-fork stress and fuzz (15 tests, 512 runs each):** tiny positions, positions at the liquidation edge (binary-searched price drops), and mid-test parameter / LTV-ramp changes. Assertions checked borrower collateral-return correctness, clean post-liquidation unwind, no stuck funds after full repay + withdraw, non-decreasing intermediate-vault share price, and bounded collateral loss without liquidation — **all passed**.
- **Expired PT vault (aPT22Oct2026, past maturity):** post-maturity pricing remained sane on the fork; existing positions could fully exit; internal and external liquidation unwind paths cleared intermediate-vault debt without LP loss in the exercised scenarios.

---

## Result

**No exploitable issues identified in the reviewed scope.**

---

## Limitations

- Source review plus read-only explorer / bytecode checks and local-fork simulation only; no live-network testing or transaction broadcast.
- Fork tests exercise selected markets and parameter ranges; they are not an exhaustive proof over all possible vault configurations.
- Third-party lending-market and oracle behaviour (Euler / Aave / Morpho / Pendle) was treated as external assumption, consistent with the public program’s known-issue posture.
