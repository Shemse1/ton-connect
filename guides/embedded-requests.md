# Wallet guide — Embedded requests

Embedded requests embed an RPC request — `sendTransaction`, `signMessage` or `signData` — directly into the connect URL via the `e` query parameter. The wallet handles connection and action in a single step.

## URL and wire format

See [`spec/deeplinks.md` § Embedded requests](../spec/deeplinks.md#embedded-requests-e) for the URL placement of `e` and the `EmbeddedWireRequest` field mappings.

## Error codes

The embedded action's `response.error.code` follows the underlying method's error table in [`spec/rpc.md`](../spec/rpc.md#central-error-catalogue). When the user rejects the connect itself, return a `connect_error` event with a code from [`spec/connect.md` § Connect event error codes](../spec/connect.md#connect-event-error-codes).

## Implementation

### Step 1 — declare the feature

Advertise `{ name: 'EmbeddedRequest' }` in `DeviceInfo.features`. Also advertise the underlying methods the embedded request can carry — typically `SendTransaction`, `SignMessage` and `SignData` — using the same feature entries as in the standard bridge flow.

### Step 2 — parse the request

Read `e` from the connect URL's query string and decode it as `base64url` (no padding). `JSON.parse` the resulting UTF-8 string to obtain the `EmbeddedWireRequest`. Parse `r` to obtain the standard `ConnectRequest`.

### Step 3 — validate the payload

Branch on the `m` discriminator (`st`, `sm`, `sd`) and apply the field mappings in [`spec/deeplinks.md` § Embedded requests](../spec/deeplinks.md#embedded-requests-e). The expanded payload is the same JSON the wallet would receive over the standard bridge for `sendTransaction`, `signMessage` or `signData`. If `e` is absent, malformed, or carries an unknown `m`, proceed with the connect flow as if `e` were not present.

### Step 4 — process and respond

Present both the connect request and the embedded action. The UX is up to the wallet — a combined screen, two sequential screens, or another flow the wallet prefers. The wallet sends **exactly one bridge message** for each connect attempt:

| Outcome                                            | What to send                                      |
|----------------------------------------------------|---------------------------------------------------|
| User rejects the connection                        | `connect_error` event — no `response` field       |
| User approves connect and embedded request         | `connect` event with `response: { result: ... }`  |
| User approves connect but rejects embedded request | `connect` event with `response: { error: ... }`   |

The `response` field follows the standard `WalletResponse` format defined in [`spec/rpc.md`](../spec/rpc.md#walletresponse). The embedded-request response does **not** carry an `id` — the request was not assigned one by the dApp.

### Step 5 — handle unsupported `e`

If the wallet does not support embedded requests, or `e` is malformed, silently ignore it and process the connection normally. The `ConnectEventSuccess` then has no `response` field, and the SDK falls back to sending the request over the bridge.

## Backward compatibility

| Scenario                            | Behaviour                                |
|-------------------------------------|------------------------------------------|
| Old wallet receives URL with `e`    | Ignores unknown param, connects normally |
| New wallet receives URL without `e` | Standard connect flow                    |
| `e` carries unsupported method      | Wallet ignores `e`, connects normally    |

## See also

- [`spec/deeplinks.md` § Embedded requests](../spec/deeplinks.md#embedded-requests-e) — canonical wire format
- [`spec/connect.md`](../spec/connect.md) — `ConnectEventSuccess` and the `response` field
