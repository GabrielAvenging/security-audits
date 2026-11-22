# GMTrade (gmx-solana) bounded review

**Status: No validated reportable finding in this bounded review; source-only; nothing submitted.**

- **Target:** https://github.com/gmsol-labs/gmx-solana
- **Commit range:** `d81a3e69` → `3e0a4115` (2 commits on `main`)
- **Date:** 2026-11-22
- **Program:** [Immunefi GMTrade](https://immunefi.com/bug-bounty/gmtrade/information/) (no KYC; PoC required; Primacy of Rules)

## Scope

The range added a Store rent-collector withdrawal instruction and an EVM proxy program scaffold.

In-scope assets on the program page are the Store, Treasury, and Liquidity Provider programs (source and deployed program IDs). The new Store instruction lives under `programs/store`, which is in scope as source. The new `gmsol-evm-proxy` program is not one of the listed assets and was treated as out of scope. Changelog entries for the collectors are under `[Unreleased]`; there are no release tags at the reviewed tip, and this review did not query a chain.

Privileged-role, upgrade-authority, admin, and keeper centralization, known issues, and prior audit findings are out of scope.

## Methodology

1. Scope check against the Immunefi information and scope pages fetched on 2026-11-22.
2. Line-by-line review of the two new commits, focused on who can call the new Store instruction, where funds can move, and whether it can touch other users' or protocol balances.
3. Duplicate check against the repo README known issues, the last Store audit (Zenith, 2026-10-09, commit `8af1e70`, before this range), and the prior Code4rena bounty issue list.
4. The local watcher (static tools only) reported 0 High / 0 Medium hits on the changed files.

## Tests

Anchor tests for the new instruction require a live Solana devnet validator clone in `Anchor.toml`, so they were not run. Offline SDK unit tests that only assert PDA seeds and account metas were not executed in this pass (the review machine did not yet have a Rust toolchain when this review was done). No temporary tests were added. No RPC, fork, wallet, or live-network transaction was used.

## Result

**No payable finding.** The in-scope Store change is a withdrawal of leftover deposit funds from a collector account derived from the signer; it does not move another party's funds or the exchange's balances. The EVM proxy addition is an out-of-scope scaffold and was not treated as attack surface.

The temporary checkout was deleted after the review.

## Limits

- Bounded diff review, not a full audit of Store, Treasury, or Liquidity Provider.
- New code was not verified as deployed.
- Program tests that need a network connection were not run.
