# Wallet guide — `sendTransaction`

`sendTransaction` asks the wallet to validate, sign and broadcast one or more outgoing messages from the connected account.

Canonical RPC field definitions live in [`spec/rpc.md`](../spec/rpc.md#sendtransaction); this page walks through the wallet-side UX and implementation rules.

## RPC format

See [`spec/rpc.md` § `sendTransaction`](../spec/rpc.md#sendtransaction) for the request envelope, payload fields and response shape.

## Error codes

Use the central error catalogue in [`spec/rpc.md`](../spec/rpc.md#central-error-catalogue). For this method, the main path is `1` for pre-signing validation failures, `300` when the user rejects, `0` for unexpected internal errors.

## Implementation

### Step 1 — declare the feature

Advertise `SendTransaction` in `DeviceInfo.features` with the wallet's runtime limits: `maxMessages`, optional `extraCurrencySupported`, optional `itemTypes` when structured items are supported. Only advertise values the wallet enforces at runtime.

### Step 2 — parse the request

Parse `request.params[0]` as the transaction payload JSON. The payload carries `valid_until`, `network`, `from` and exactly one of `messages` or `items`.

### Step 3 — validate the payload

Reject with code `1` (`BAD_REQUEST`) when:

- neither `messages` nor `items` is present, or both are present;
- `valid_until` is set and the current time is past it;
- `network` is set and does not match the active wallet network;
- `from` is set and the wallet cannot sign from that address;
- the item or message count exceeds the advertised `maxMessages`.

### Step 4 — sign and broadcast

Build outgoing messages from `messages` (raw) or `items` (see [`structured-items.md`](./structured-items.md)). Render the confirmation UI with sender, every destination and value, asset type, decoded payload where possible and the batch size. On user approval, sign the external message and broadcast it on-chain. On user rejection, return code `300`.

### Step 5 — respond

Return the broadcast BoC in `SendTransactionResponse`. If broadcast fails after signing, return code `0` with an operational error message.

## UX rules

- Show all outgoing messages in order for batch requests.
- Show expiration (`valid_until`) if present so users can detect stale requests.
- Do not allow changing sender when `from` is set.
- Distinguish `sendTransaction` from `signMessage`: this flow broadcasts on-chain and may consume network fees.

## See also

- [`spec/rpc.md`](../spec/rpc.md#sendtransaction) — canonical RPC definition
- [`structured-items.md`](./structured-items.md) — building BoCs from `items`
- [`wallet-guidelines.md`](./wallet-guidelines.md) — wallet-side rules
