# Security Audit: MoneroOcean/nodejs-pool

**Target:** [MoneroOcean/nodejs-pool](https://github.com/MoneroOcean/nodejs-pool)  
**Commit reviewed:** `d56fdbc873fd9d82abce52c6f78fe5cef80dd86c` (2026-10-26)  
**Review date:** 2026-10-27  
**Method:** Static source-code review only

---

## Summary

This defensive audit reviewed MoneroOcean's Node.js mining-pool codebase at the commit above. **No hot-wallet drain path was found.** Payout addresses are checksum-validated at login, payment-ID logins are rejected, wallet `transfer` parameters are built as structured JSON from database balances, and batches are claimed and reconciled before and after submit. SQL reachable from user input is parameterized, and the HTTP API has no admin surface.

What remains:

- Two **Medium** issues that allow unauthenticated changes to account settings in the default configuration (payout threshold and notification email).
- Share-credit integrity issues that are configuration-dependent or intentional trust trade-offs.
- A few informational hardening items.

Nothing rates Critical or High under default configuration.

### Key conclusions

| Risk | Result |
|---|---|
| Hot-wallet drain / wallet-RPC destination injection | **Not found** |
| Steal or skew miner rewards (default config) | Indirect only — F-04 ~2% PPLNS dilution with `trustedMiners` (on by default; model estimate) |
| Remote unauthenticated crash / DoS | None found in this pass |
| Account / settings griefing (default) | **F-01** threshold freeze, **F-02** email claim |

---

## Scope

### In scope

| Area | Files |
|---|---|
| Stratum TCP / framing / rate limits | `lib/pool/servers.js`, `lib/pool/protocol.js` |
| Login parsing / payout split / address validation | `lib/pool/miners.js`, `lib/coins/index.js` |
| Share verification / trust / block submit | `lib/pool/shares.js`, `lib/pool/share_blocks.js`, `lib/pool/state.js` |
| Leaf → master share ingestion | `lib/remote_share.js`, `lib/pool/remote_uplink.js`, `lib/remote_share/*` |
| HTTP API | `lib/api.js`, `lib/api/public.js`, `lib/api/account.js` |
| Payments / wallet RPC | `lib/payments.js`, `lib/payments/{planner,processor,common}.js` |
| Long runner / block manager | `lib/long_runner.js`, `lib/block_manager.js` |
| DB layer | MySQL (`?` placeholders) + LMDB (`lib/common/database.js`) |
| Shell / exec | `lib/coins/index.js`, `lib/block_manager.js` (`execFile` only) |
| Email tokens / templates / misc | `lib/common/support.js` |

### Out of scope / limitations

- Static review of a single commit only.
- No live pool, wallet, daemon, or host was contacted, scanned, or tested.
- Dependencies (`node_modules`, native hashing/address addons, `monero-wallet-rpc`) were not audited.
- Non-XMR coin profiles received lighter coverage than the XMR path.
- The F-04 ~2% figure is an offline model estimate, not a live measurement.
- No ready-to-run exploit scripts are published with this report.

### Exploitability classes

| Class | Meaning |
|---|---|
| **(a)** | Remote and unauthenticated in default config |
| **(b)** | Remote but configuration-dependent |
| **(c)** | Not remotely exploitable / local or config-only |

---

## Findings overview

| ID | Severity | Location | Title |
|---|---|---|---|
| F-01 | Medium | `lib/api/account.js:148-168`, `:255-259` | Unauthenticated change of any wallet's payout threshold |
| F-02 | Medium | `lib/api/account.js:187-215`; `lib/pool/miners.js:371-382` | Unauthenticated email binding / notification toggle |
| F-03 | Low (High if misconfigured) | `lib/remote_share.js:430-433`, `:266-275` | `/leafApi` trusts any frame carrying the shared key |
| F-04 | Low | `lib/pool/share_blocks.js:94-120` | Trusted-share sampling allows ~2% PPLNS credit inflation |
| F-05 | Low | `lib/pool/shares.js:152-160`, `:756-764`; `lib/pool/protocol.js:270-274` | Loopback peers trusted (test bypass + rate limits) |
| F-06 | Low | `lib/api/account.js:18-20`, `:74-83` | Per-username throttle map can be flushed |
| F-07 | Info | `lib/common/support.js:319-330` | Unsubscribe key falls back to empty `secKey` |
| F-08 | Info | `lib/common/support.js:859`, `:876` | SQL built by string interpolation of internal `coin` |

---

## F-01 (Medium): Anyone can change any wallet's payout threshold

**Class:** (a) Remote, unauthenticated, default config  
**Location:** `lib/api/account.js:148-168`, route at `:255-259` (`POST /user/updateThreshold`)

### Vulnerable code

```js
async function updateThreshold(username, thresholdValue) {
    // ... validation omitted ...
    const rows = await query(
        "SELECT id FROM users WHERE username = ? AND payout_threshold_lock = '1'",
        [username]
    );
    if (rows.length !== 0) {
        return { status: 401, payload: { success: false, msg: "Can't update locked payment threshold" } };
    }
    await query(
        "INSERT INTO users (username, payout_threshold) VALUES (?, ?) ON DUPLICATE KEY UPDATE payout_threshold = ?",
        [username, thresholdCoin, thresholdCoin]
    );
}

registerJsonRoute(app, "post", "/user/updateThreshold", "user update threshold", async function handler(req, res) {
    const body = getBody(req);
    const result = await updateThreshold(body.username, body.threshold);
    return sendJson(res, result.status, result.payload);
});
```

### Details

The endpoint reads `username` (a public wallet address) and `threshold` from the request body and writes `users.payout_threshold`. It asks for no proof of ownership (no signature, session token, password, or email confirmation) and has no rate limit. The only guard is the operator-set `payout_threshold_lock` column.

### Impact

- A third party can set a victim's threshold to 1000 XMR and keep resetting it, which in practice withholds that miner's payouts indefinitely.
- They can also set it to the minimum and force frequent small payouts.
- **No funds are stolen**; balances remain owed in the pool wallet.

### Conceptual PoC (local testnet only)

On a private local API with test data: `POST /user/updateThreshold` with `{username: <other testnet wallet>, threshold: 1000}` and observe `users.payout_threshold` change so the planner stops paying that wallet.

### Fix recommendations

1. Require a wallet-signed message over `{address, threshold, nonce, expiry}`.
2. Or send a confirmation link to an already-verified email.
3. Add a rate limit. Treat `payout_threshold_lock` as only a partial mitigation.

```js
// Sketch: require wallet signature before applying the change
if (!(await verifyWalletSignature(body.username, body.nonce, body.signature, "updateThreshold"))) {
    return sendJson(res, 401, { success: false, msg: "Signature required" });
}
```

---

## F-02 (Medium): First-come email binding and unauthenticated notification toggle

**Class:** (a) Remote, unauthenticated, default config  
**Location:** `lib/api/account.js:187-215`; `lib/pool/miners.js:371-382`

### Vulnerable code

```js
if (body.from === "" && body.to === "") {
    const result = await query(
        "UPDATE users SET enable_email = ? WHERE username = ?",
        [body.enabled, body.username]
    );
    // ...
}
if (body.from === "") {
    const result = await query(
        "UPDATE users SET enable_email = ?, email = ? WHERE username = ? AND (email IS NULL OR email = '')",
        [body.enabled, body.to, body.username]
    );
    // ...
}
```

```js
// stratum path: first share to an address can INSERT users.email from the password field
global.mysql.query(
    "INSERT INTO users (username, email) VALUES (?, ?)",
    [payoutAddress, email]
);
```

### Details

Email “ownership” is first-come:

- If a wallet has no email, anyone can bind one (`from === ""`).
- Toggling `enable_email` needs only the wallet address (`from === "" && to === ""`).
- Changing an existing email requires knowing the current one.
- Mining one valid share to a victim's address with `worker:<email>` as the password can create the `users` row with that email if none exists yet.

### Impact

- Attacker receives payment emails (amount, `tx_hash`, `tx_key`) and worker notices.
- Real owner is locked out of binding their own email.
- Anyone can silently disable notifications for any wallet.
- **No funds move.**

### Conceptual PoC (local testnet only)

On a local testnet wallet with no `users.email`: `POST /user/subscribeEmail` with `{username: W, from: "", to: "x@example.test", enabled: 1}`, then trigger a local payout and observe the notification destination.

### Fix recommendations

- Double opt-in **and** proof of wallet control before binding.
- Store stratum-supplied email only as *pending* until confirmed.
- Require the same proof to toggle `enable_email`.

---

## F-03 (Low; High impact if misconfigured): `/leafApi` trusts frames that carry the shared key

**Class:** (b) Remote, config-dependent  
**Location:** `lib/remote_share.js:430-433`, `:266-275`

### Vulnerable code

```js
if (msgData.key !== global.config.api["authKey"]) {
    state.requestStats.authRejected += 1;
    return respond(res, 403);
}
// ...
queueShare(share);
```

### Details

The master accepts share/block frames from leaf nodes over HTTP. Auth is a shared-key string compare (`!==`, not constant-time) with **no guard against an empty `authKey`**. A protobuf frame without a key decodes to `""`. `/leafApi` is published through nginx in `deploy.bash`, so the key alone protects share accounting. Default installs generate a random 32-character key (`deploy.bash:629`).

### Impact

If `authKey` is empty, weak, or leaked, forged shares/blocks can redirect PPLNS rewards. This is **not** a direct wallet-RPC call.

### Conceptual PoC (local testnet only)

On a local master with `authKey=""`, POST a SHARE frame with an empty key to local `/leafApi` and observe the share queued.

### Fix recommendations

```js
const expected = global.config.api["authKey"];
if (typeof expected !== "string" || expected.length < 32) {
    throw new Error("api.authKey must be >= 32 chars");
}
const got = Buffer.from(typeof msgData.key === "string" ? msgData.key : "");
const exp = Buffer.from(expected);
if (got.length !== exp.length || !crypto.timingSafeEqual(got, exp)) {
    state.requestStats.authRejected += 1;
    return respond(res, 403);
}
```

Also restrict `/leafApi` to leaf IPs or mTLS, and bounds-check share fields on receipt.

---

## F-04 (Low, config-dependent): Trusted-share sampling allows ~2% credit inflation

**Class:** (b) Remote; on by default (`trustedMiners=true`)  
**Location:** `lib/pool/share_blocks.js:94-120`  
**Defaults:** `trustedMiners=true`, `trustMin=20`, `trustThreshold=30` (`deployment/base.sql`)

### Vulnerable code

```js
const gate = walletOk
    ? trustMin
    : Math.max(256 - minerTrust / rewardDiff / (2 * trustChange), trustMin);
return crypto.randomBytes(1).readUIntBE(0, 1) > gate;
```

### Details

Once a miner is trusted, only about 8% of shares are fully hashed. A caught fake resets trust and is far below the ban threshold. An offline model of the gate formula estimates a best-case attacker gain of about **2.4%** extra credited difficulty, taken from other PPLNS miners' rewards. It does not touch the hot wallet. An earlier double-draw issue in this area appears already fixed.

### Conceptual PoC (local testnet only)

Build trust with valid shares on a local pool, then interleave fabricated results; compare credited vs rejected counts in local storage. (Descriptive only — no attack tooling is provided.)

### Fix recommendations

Penalize failed trusted-path verification heavily (temporary ban and/or revoke credit back to the last verified share).

---

## F-05 (Low, local/config-only): Loopback peers are trusted

**Class:** (c) Local/config-only  
**Location:** `lib/pool/shares.js:152-160`, `:756-764`; `lib/pool/protocol.js:270-274`

### Vulnerable code

```js
function isBlockSubmitTestBypassEnabled(miner) {
    return isBlockSubmitTestModeEnabled() === true
        && isLoopbackAddress(miner && miner.ipAddress);
}
```

### Details

When stratum sits behind a local proxy so every miner appears as `127.x`, all miners share the loopback login rate-limit bucket. If the `.pool-live-block-submit-test` marker file also exists, block-difficulty shares with self-reported hashes can be credited without verification. Unreachable in normal operation; the marker is cleared at startup.

### Fix recommendations

Also require an explicit non-mainnet config flag and an allow-listed test wallet. Document that proxies must forward the real client IP (PROXY protocol).

---

## F-06 (Low): The throttle map can be flushed

**Class:** (a) Remote, unauthenticated  
**Location:** `lib/api/account.js:18-20`, `:74-83`

### Vulnerable code

```js
const MAX_SENSITIVE_FAILURE_KEYS = 5000;
while (sensitiveFailures.size > MAX_SENSITIVE_FAILURE_KEYS) {
    const first = sensitiveFailures.keys().next();
    if (first.done) break;
    sensitiveFailures.delete(first.value);
}
```

### Details

About 5000 requests with distinct usernames can evict the 2-second throttle that slows guessing a target's current email (`from`) for F-02.

### Fix recommendations

Fail closed when the map is full, and add a per-IP limiter.

```js
if (sensitiveFailures.size >= MAX_SENSITIVE_FAILURE_KEYS && !sensitiveFailures.has(key)) {
    return throttlePayload(SENSITIVE_FAILURE_DELAY_MS);
}
```

---

## F-07 (Info, config-only): Unsubscribe key falls back to empty `secKey`

**Location:** `lib/common/support.js:320-322`

```js
const secKey = global.config && global.config.api && typeof global.config.api["secKey"] === "string"
    ? global.config.api["secKey"]
    : "";
```

**Impact:** Missing `secKey` yields tokens under a key derived from `""`, allowing forged unsubscribe links (email disable only; F-02 already covers that path). Default deploy sets a random key.

**Fix:** Fail at startup when `secKey` is missing or too short.

---

## F-08 (Info): SQL built by string interpolation

**Location:** `lib/common/support.js:859`, `:876`

```js
global.mysql.query(
    `SELECT item_value FROM config WHERE module = 'daemon' and item = 'coinHashFactor${coin}'`
)
```

**Impact:** `coin` comes from the pool's internal coin list, so this is not exploitable today. It is a latent injection sink.

**Fix:** Use a placeholder: `... AND item = ?` with `[`coinHashFactor${coin}`]`.

---

## Classification

| ID | Severity | Exploitability | Real money? |
|---|---|---|---|
| F-01 | Medium | (a) | Delays/freezes owed payouts; no theft |
| F-02 | Medium | (a) | No funds; leaks payment details |
| F-03 | Low (High if misconfigured) | (b) | Yes if `authKey` empty/weak/leaked — PPLNS redirect |
| F-04 | Low | (b, on by default) | ~2% PPLNS dilution (model) |
| F-05 | Low | (c) | Only if proxy + test marker |
| F-06 | Low | (a) | No |
| F-07 | Info | (c) | No |
| F-08 | Info | (c) | No |

**No finding reaches the hot wallet or enables a drain.**

---

## Areas reviewed with no finding

- **Hot-wallet / wallet-RPC injection:** `transfer` params built as structured JSON; addresses checksum-validated; payment-ID logins rejected; amounts integer-normalized from DB balances; batches claimed and reconciled.
- **SQL injection:** User-reachable queries use `?` placeholders; non-primitive values rejected where relevant.
- **Stratum DoS:** Packet/frame size caps, JSON structure-token caps, per-IP connection caps, RPC rate limits. No unauthenticated crash found in this static pass.
- **exec/shell:** Only `execFile` with internally derived arguments.
- **Deserialization:** `JSON.parse` and protobuf only; no `eval` / `vm` / unsafe serializers.
- **Admin endpoints / SSRF:** None found.
- **XSS:** Unsubscribe HTML escapes values; tokens authenticated with AES-256-GCM.

---

## Methodology

1. Mapped the current tree (stratum, API, payments, remote share, DB, exec paths).
2. Manual static review of authentication, authorization, share accounting, payout construction, and config defaults.
3. Cross-checked defaults in `deployment/base.sql` and `deploy.bash`.
4. For F-04 only, ran a small **offline** numeric model of the trust-gate formula (no network).
5. Documented conceptual local-testnet sketches only — no live probing and no ready-to-run attack code.

---

## Disclosure posture

This report is published for operators and maintainers. Reproduction notes are intentionally short and local-testnet-oriented. Please treat any residual risk in production deployments according to your own threat model and patch timeline.
