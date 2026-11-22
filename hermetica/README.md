# Hermetica hBTC trading v2 bounded review

**Status: No validated reportable finding in this bounded review; source-only; nothing submitted.**

- **Target:** https://github.com/hermetica-fi/hermetica-contracts
- **Commit range:** `5e13e43` → `4e473be` (3 commits on `master`; HEAD `4e473be781025ba43a13766fe8803a836a702713`)
- **Date:** 2026-11-22
- **Program:** [Immunefi Hermetica](https://immunefi.com/bug-bounty/hermetica/information/) (no KYC; PoC required; Primacy of Impact for Critical and High; no Medium payout)
- **Prior pass:** a 2026-11-05 review of this repository at `5e13e43` is not published in this collection.

## Scope

This pass covered the November 16 updates to the hBTC Trading and Zest Interface contracts (v2), after the January 2026 / November 2025 audits and after the earlier review pin. Named in-scope assets include Trading and Zest Interface. A later commit in the same range added USDh-product sources; those were only spot-checked as a dependency of the new trading path and are not a named hBTC asset.

Privileged-address / strategist attacks, governance, centralization, and previously audited issues are out of scope. The trader role used by the new public functions is a privileged strategist role under the program rules. This review did not query a chain.

## Methodology

1. Scope check against the Immunefi information and scope pages fetched on 2026-11-22.
2. Diff review of trading v2 and zest-interface v2 against the audited v1 contracts, focused on who can call new entry points and whether an unprivileged caller can move funds.
3. Check of prior public audit notes that the v2 changes are documented as addressing.
4. The local watcher reported 0 High / 0 Medium hits (Clarity; no Semgrep hits of note).

## Tests

The repository has 22 `trading-v2` tests and 32 `zest-interface-v2` tests. They run against a Stacks mainnet fork and require a Hiro API key. That is live-network RPC, so **no tests were run**. No temporary tests were added.

## Result

**No payable finding.** Every new public function reviewed in the v2 trading and Zest interface contracts is gated on the protocol's trusted trader role, which the bounty treats as out of scope. No novel Critical or High path for an unprivileged caller was identified.

The temporary checkout and install were deleted after the review.

## Limits

- Bounded review of the v2 trading / Zest interface diff, not a full audit of Hermetica.
- Program tests that need a mainnet fork were not run.
- On-chain deployment was not verified with RPC.
