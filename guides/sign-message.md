# Wallet guide — `signMessage`

`signMessage` is an RPC method alongside `sendTransaction` and `signData`. The wallet **signs** an internal message but **does not broadcast** it. The signed BoC returns to the dApp, which submits it through a relayer (for example in gasless transactions).

Most of the implementation is shared with `sendTransaction`. The two differences: skip the broadcast step and return the signed BoC instead of the broadcast BoC.

## RPC format

See [`spec/rpc.md` § `signMessage`](../spec/rpc.md#signmessage) for the request envelope, payload fields and response shape. The payload structure is identical to `sendTransaction`.

## Error codes

Use the central error catalogue in [`spec/rpc.md`](../spec/rpc.md#central-error-catalogue). For this method, the main path is `1` for pre-signing validation failures, `300` when the user rejects, `0` for unexpected internal errors.

## Implementation

### Step 1 — declare the feature

Advertise `SignMessage` in `DeviceInfo.features`, mirroring the `SendTransaction` capability shape: `maxMessages`, optional `extraCurrencySupported`, optional `itemTypes`. Only advertise fields the wallet supports at runtime.

### Step 2 — parse the request

Parse `request.params[0]` as the transaction payload JSON. The payload is the same shape as `sendTransaction` and accepts `messages` or `items`, `valid_until`, `network`, `from`.

### Step 3 — validate the payload

Apply the same validation as `sendTransaction` — `messages` XOR `items`, network and time bounds, `from` lock, `maxMessages` cap. Reject failures with code `1`. If `valid_until` has expired, return code `1` (`BAD_REQUEST`).

### Step 4 — sign without broadcasting

Construct each outgoing message with **send mode 3** (`PAY_GAS_SEPARATELY + IGNORE_ERRORS`). Render the same payload preview as `sendTransaction`, marked clearly as signed-but-not-sent. On user approval, build the signed internal BoC. On user rejection, return code `300`.

> **Note.** Wallet V5 implementations use the `internal_signed` opcode (`0x73696e74`) for the signed body.

### Step 5 — respond

Return `{ result: { internalBoc: <base64 BoC> } }`. Do not submit the signed message to the network. For any other signing failure, return code `0` with a description.

## UX rules

- Show the same payload preview as `sendTransaction` — destination, amount, payload.
- Mark the operation visibly as **signed but not sent** so the user understands the wallet will not deduct gas.
- Do not deduct fees in the preview — the wallet does not pay gas in this flow.

## See also

- [`spec/rpc.md` § `signMessage`](../spec/rpc.md#signmessage) — canonical RPC definition
- [`structured-items.md`](./structured-items.md) — building BoCs from `items`
