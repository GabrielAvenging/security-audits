# Enzyme Finance Protocol Onyx — Post-Audit Diff Review

**Date:** November 5, 2026  
**Target:** Enzyme Finance Protocol Onyx  
**Public repository:** [enzymefinance/protocol-onyx](https://github.com/enzymefinance/protocol-onyx)  
**Public bug bounty scope:** reviewed against publicly listed in-scope source pins

---

## Commit ranges reviewed

| Role | Commit | Notes |
|---|---|---|
| Last full-audit baseline | [`b98576fa39213b3c8aecb8b5b104c9acaa24eff0`](https://github.com/enzymefinance/protocol-onyx/commit/b98576fa39213b3c8aecb8b5b104c9acaa24eff0) | ChainSecurity full-audit pin |
| Post-audit `src/` delta end | [`85570db648edb66240564ae844d88d5a3f8fe610`](https://github.com/enzymefinance/protocol-onyx/commit/85570db648edb66240564ae844d88d5a3f8fe610) | Primary post-audit source delta (`b98576f` → `85570db`) |
| Address-list in-scope pin | [`61ef53d2dfd17d83183d832529b148e6e15c26ab`](https://github.com/enzymefinance/protocol-onyx/commit/61ef53d2dfd17d83183d832529b148e6e15c26ab) | `AddressList*` / `SharesOwnedAddressList` |
| Shares + transfer-validator pin | [`7b48d243f505b362cd6f1fcf723c21bc8b047c06`](https://github.com/enzymefinance/protocol-onyx/commit/7b48d243f505b362cd6f1fcf723c21bc8b047c06) | `Shares.sol`, `AddressListsSharesTransferValidator.sol` |

Prior public audits named for context (not re-quoted): **ChainSecurity** (full audit, CCIP wallet, Chainlink ACE integration, and related QA notes).

---

## Approach

Post-audit **diff review**. Compared the last ChainSecurity full-audit commit to the later in-scope pins and reviewed **only new or changed code** in that window — primarily address-list infrastructure and list-based shares transfer validation introduced after `b98576f`. Other publicly in-scope surfaces (ACE / CCIP / CRE / Sync* handlers) were spot-checked against their existing ChainSecurity audit and QA coverage rather than fully re-audited.

---

## Areas checked

Security properties verified as holding on the reviewed surface:

- **Address-list infrastructure** (`AddressListBase`, `OwnableAddressList`, `SharesOwnedAddressList`): owner / admin authorization on add-remove paths; storage layout / initializer expectations for proxy use; no broken auth on list mutation.
- **List-based shares transfer validator** (`AddressListsSharesTransferValidator`): transfer gating for ordinary `transfer` / `transferFrom` paths; list configuration checks; consistency with documented ERC-20-style initiator handling.
- **Shares privileged paths** (`authTransfer` / `authTransferFrom` / `mintFor` / `burnFor`): confirmed they intentionally bypass the transfer validator and rely on separate issuance / redeem hooks and allowlists (trusted-admin / design model, not a novel bypass).
- **Issuance / redeem list gating** (deposit-queue and related list wiring): sequencing and list-pointer behavior checked against existing QA notes; no new exploitable mis-sequencing identified in the reviewed delta.
- **Linear credit/debt tracker** (`LinearCreditDebtTracker` at the `85570db` pin): duration / start-edge valuation semantics match documented NatSpec; no new post-audit vulnerability identified in the reviewed changes.

---

## Result

**No exploitable issues identified in the reviewed scope.**

---

## Limitations

- Source review only; no live-network testing, forking of production state, or transaction broadcast.
- Focused on the post–full-audit address-list / transfer-validator surface and related Shares interactions; ACE / CCIP / CRE / Sync* in-scope pins received spot-checks against prior audits rather than a full re-audit.
- Deployed bytecode was not independently verified against the GitHub pins reviewed here.

---

## November 9, 2026 bounded review

**Date:** November 9, 2026  
**Reviewed tree:** [`d8a522d6ca58af9dbb831dad2b8045a88952e468`](https://github.com/enzymefinance/protocol-onyx/tree/d8a522d6ca58af9dbb831dad2b8045a88952e468)

Separate bounded pass at the tree above. It does not replace the November 5 diff review.

### Coverage

Sync and ERC-7540 deposit and redeem, the fee handler and trackers, valuation, and shares access control.

### Hypotheses

Tested at that level only. "Refuted" means these checks did not validate a reportable finding.

| Hypothesis | Result |
|---|---|
| A reportable defect in sync or ERC-7540 deposit and redeem | Tested and refuted in the tests that ran |
| A reportable defect in the fee handler or trackers | Tested and refuted in the tests that ran |
| A reportable defect in valuation | Tested and refuted in the tests that ran |
| A reportable defect in shares access control | Tested and refuted in the tests that ran |

### Recorded checks

| Group | Result |
|---|---|
| Existing | 510 passed |
| Added | 6 passed |
| **Executed total** | **516 passed, 0 failed** |
| Fork / end-to-end | 43 skipped (RPC required; not run) |

### Result

**No validated reportable finding in this bounded review; source-only unless noted; nothing submitted.**

The 516 results are the tests that executed. Forty-three fork and end-to-end tests were skipped because they need an RPC, so those tests are not evidence in this pass.

### Limitations

- Bounded to the surfaces named above at one tree.
- Skipped fork and end-to-end tests were not executed.
- This pass does not verify deployed bytecode and does not record a bounty acceptance.
