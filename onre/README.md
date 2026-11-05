# OnRe Solana Program — Post-Audit Diff Review

**Date:** November 5, 2026  
**Target:** OnRe Solana program (`onre-sol`)  
**Public repository:** [onre-finance/onre-sol](https://github.com/onre-finance/onre-sol)  
**Public bug bounty scope:** reviewed against the publicly listed program source

---

## Commit ranges reviewed

| Role | Commit | Notes |
|---|---|---|
| Inferred last-audited baseline | [`1dbc6ec0ed18a8fc81516f930d85e669b689d852`](https://github.com/onre-finance/onre-sol/commit/1dbc6ec0ed18a8fc81516f930d85e669b689d852) | Merge closing OtterSec remediation commits (OTR-1 … OTR-15); treated as audited baseline by inference |
| Current in-scope HEAD reviewed | [`c0490831f438074562ebf51e437ca5f8e027596b`](https://github.com/onre-finance/onre-sol/commit/c0490831f438074562ebf51e437ca5f8e027596b) | Post-remediation delta reviewed end-to-end |

Reviewed range: **`1dbc6ec` → `c049083`** (new code only). Material changes in that window include client-generated redemption request IDs, variable-length excluded-owner list input, account-size preservation for redemption requests, and embedded security disclosure metadata.

Prior public audits named for context (not re-quoted): **Quantstamp**, **Ackee**, **OtterSec**.

---

## Approach

Post-audit **diff review**. The last audited commit was **inferred** from the OtterSec remediation series (OTR-1 … OTR-15) landing in `1dbc6ec`, then compared to the current in-scope commit `c049083`. Only code introduced after that inferred baseline was reviewed in depth; pre-existing / previously audited program logic was not re-audited.

---

## Areas checked

Security properties verified as holding on the new code:

- **`create_redemption_request`**: request ID constrained to exactly 32 bytes; PDA seeds include offer, redeemer, and id (redeemer in seeds blocks third-party squat of another user’s PDA); `init` prevents account reuse; amount must be positive; checked arithmetic on balances.
- **`cancel_redemption_request` / `fulfill_redemption_request`**: PDA seeds re-derived from stored offer / redeemer / id with stored bump (or `find_program_address`); redeemer / worker / boss account constraints unchanged; closed accounts cannot be revived under the same id.
- **Account layout**: `RedemptionRequest` account size preserved across the ID-format change; `RedemptionOffer` retains its reserved gap slot for layout compatibility.
- **`set_circulating_supply_excluded_accounts`**: boss-gated; length capped; uniqueness and non-default pubkey validation; zero-padding of unused slots.

---

## Result

**No exploitable issues identified in the reviewed scope.**

---

## Limitations

- Source review only; no live-network testing, local program-test attack development, or mainnet interaction.
- **Audited-commit inference:** the OtterSec remediation merge was treated as the last audited baseline; if additional unaudited commits sit before that merge, they were outside this pass.
- **Deployed-versus-source not verified:** on-chain programdata was not matched to a reproducible build of `c049083`; findings are about the GitHub source at the reviewed commits only.
- Pre-existing / previously audited code paths outside the post-remediation delta were not re-reviewed.
