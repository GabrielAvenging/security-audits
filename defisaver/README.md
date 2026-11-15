# DeFi Saver v3 contracts: CurveUsd/LlamaLend swapper and Fluid diff (bounded review)

**Status: No validated reportable finding in this bounded review; source-only plus offline unit tests; nothing submitted.**

- **Target:** https://github.com/defisaver/defisaver-v3-contracts
- **Commit range:** `26146bf1` → `cae44d54` (9 commits; PR #654 `improvement/advanced-llamalend-curveusd-actions`, PR #655 `fix/fluid-oracle`)
- **Date:** 2026-11-14
- **Program:** Immunefi DeFi Saver (assets in scope: "Defi Saver V3 (excluding the 'mocks' and 'views' folders)")

## Scope check

| Changed code | Repo deployment metadata (`addresses/*.json` at `cae44d54`) | In Immunefi scope? |
|---|---|---|
| `contracts/actions/curveusd/advanced/CurveUsdSwapperTransient.sol` | New mainnet address `0xfb667B694aeA7c1ECF3bd581d06FefdcA769099c`, v1.0.2, `inRegistry: true`, `changeTime: 86400` | Yes (V3 contracts) |
| `contracts/actions/llamalend/advanced/LlamaLendSwapper.sol` | New mainnet `0x1946d286…b547` and Arbitrum `0xce3701F7…9C6D`, v1.0.2, `inRegistry: true` | Yes |
| `CurveUsdRepayTransient.sol`, `CurveUsdSelfLiquidateWithCollTransient.sol`, LlamaLend Repay / SelfLiquidateWithColl | Not redeployed; only a `payable(...)` cast changed | Yes, but no logic change |
| `contracts/actions/fluid/helpers/addresses/*FluidAddresses.sol` | Resolver constants updated; no Fluid action redeployment is recorded in this range | Yes, but no deployed bytecode changes yet |
| `contracts/views/FluidView.sol` (the oracle V2 / lending-resolver fix) | New addresses on 4 chains, `inRegistry: false` | **No.** The `views` folder is excluded |

The registry status comes from the repo's own metadata. It was not checked on-chain, because this review does not probe live networks. The 86400-second change time means a registry swap could still have been pending when the commits landed.

## What was reviewed

- **`CurveUsdSwapperTransient`** (+148/−...), the core change:
  - Controller callbacks (`callback_repay`, `callback_deposit`, `callback_liquidate`) are now `nonReentrant`.
  - Collateral and debt tokens now come from the calling controller, not from `exData`, and the swap direction is validated with `InvalidExchangeTokens`.
  - The callback user is recorded in transient storage. `withdrawAll` now pays out only when called by that user.
  - Partial-fill debt refunds go to the user in `callback_deposit`.
  - A `receive()` function was added for ETH dust.
- **`LlamaLendSwapper`**: the same refactor, with the controller validated through `(controller, controllerId)` against the LlamaLend factory.
- **Repay and self-liquidate actions**: the flow from transient `exData` through the extended controller call, then `withdrawAll`, then the snapshot-based leftover sweep.
- **Shared code**: `BytesTransientStorage` (permissionless global transient bytes), `DFSExchangeWithTxSaver`, the slippage check in `DFSExchangeCore._sell`, `TokenUtils.withdrawTokens`, and the vendored `SafeERC20.safeApprove`.
- **Fluid**: the resolver address swap and a skim of the FluidView oracle V2 handling. FluidView is out of scope, so it was not reviewed in depth.

## Methodology

1. Range diff and line-by-line reading of the changed action and swapper code, cross-checked against the crvUSD/LlamaLend controller callback semantics. Covered: who triggers each callback, the `_user` it receives, and where proceeds go.
2. Threat model:
   - Transient-storage misuse: global writable exData, stale `callbackUser` within one transaction, callback skipped on liquidation.
   - Callback access control.
   - Third-party `liquidate_extended` using the DFS swapper as callbacker against a DFS user.
   - Fee and gas-argument abuse.
   - Approve and leftover handling.
   - Slippage when `sellAllCollateral` overrides `srcAmount`.
3. Offline Foundry unit tests against the real `CurveUsdSwapperTransient` source, with mock ERC20, factory, and controller contracts `vm.etch`-ed at the hard-coded addresses. No RPC, fork, or wallet was used.
4. The automated watcher (slither, aderyn, semgrep) reported 0 high/medium hits on the changed files.

## Results

- Callbacks require `msg.sender` to be a factory-registered controller. A third party can make a valid controller call the swapper only for its *own* position (repay, deposit) or as a liquidator, where controller proceeds go to the liquidator. No path was found that spends another user's funds.
- Global `BytesTransientStorage` is writable by anyone, but `exData` is written and consumed within the same user transaction. The only external code between the write and the read is the trusted Curve controller. The new token-direction check also stops attacker-set `exData` from naming foreign tokens.
- `withdrawAll` gating: the old version let any caller sweep whatever sat on the swapper mid-transaction. The new version pays out only to the recorded callback user. A stale `callbackUser` from an earlier callback in the same transaction can only be the same wallet, since another wallet's position actions cannot be injected into someone's transaction.
- Slippage stays price-based (`minPrice × srcAmount` after fees), so overriding `srcAmount` with `sellAllCollateral` does not weaken it.
- Approvals are set to exactly what the controller pulls. The vendored `safeApprove` falls back to resetting the allowance to 0 first, so leftover allowances do not block later calls.
- **Informational:** ETH received through `receive()` is not refunded to users and can only be recovered by the owner. The code documents this as intended dust handling.
- **Informational:** the Fluid resolver constants affect Fluid actions and triggers only once they are redeployed. Whether the new resolver addresses are correct was not checked on-chain.

**Payable finding:** none. Most of this range hardens the swappers, and no Medium-or-higher issue was identified.

## Test results (offline, Foundry, solc 0.8.24, cancun)

| Test | Result |
|---|---|
| `withdrawAll` from a non-callback-user leaves swapper balances untouched | PASS |
| Callback from an unregistered controller reverts `CurveUsdInvalidController` | PASS |
| Callback with reversed swap direction in transient exData reverts `InvalidExchangeTokens` and leaves `callbackUser` unset | PASS |
| Liquidate callback with a foreign src token reverts `InvalidExchangeTokens` | PASS |

4 passed, 0 failed. The temporary test harness and checkout were deleted after the run. Full swap paths (exchange wrappers, TxSaver) were not exercised, because they need deployed registry state.

## Limits

- Bounded diff review, not a full audit.
- No mainnet fork and no on-chain verification of the deployed bytecode or registry entries.
- The Curve controller behavior was reasoned from its public Vyper semantics, not executed.
