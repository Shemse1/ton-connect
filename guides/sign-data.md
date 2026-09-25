# Wallet guide — `signData`

`signData` asks the wallet to sign opaque application data — text, binary bytes or a TVM cell. Canonical RPC field definitions live in [`spec/rpc.md`](../spec/rpc.md#signdata); this page walks through the wallet-side UX and implementation rules.

## RPC format

See [`spec/rpc.md` § `signData`](../spec/rpc.md#signdata) for the request envelope, the three payload variants (`text`, `binary`, `cell`), the response shape and the signature constructions.

## Error codes

Use the `signData` error codes from [`spec/rpc.md` § `signData`](../spec/rpc.md#signdata).

## Implementation

### Step 1 — declare the feature

Advertise `SignData` in `DeviceInfo.features` listing only the payload `types` the wallet actually handles (`text`, `binary`, `cell`). If the wallet does not parse cells, omit `cell` — the SDK will not send cell payloads.

### Step 2 — parse the request

Parse `request.params[0]` as the payload JSON. Read the `type` discriminator and route to the matching variant. The payload also carries optional `network` and `from`.

### Step 3 — validate the payload

Reject with code `1` (`BAD_REQUEST`) when `type` is unknown, required fields for the variant are missing or `from` is set and locked to an account the wallet cannot sign from.

### Step 4 — display and sign

Display the payload per its variant before signing, with the dApp's domain visible:

- `text` — display the `text` field verbatim, monospace, no formatting, line breaks preserved; the user SHOULD scroll long messages.
- `binary` — display an "unknown content" warning; a hex preview MAY accompany it.
- `cell` — parse `cell` against `schema` and display the decoded preview. On parse failure or schema mismatch, display the same warning as for `binary`.

On user rejection, return code `300`.

### Step 5 — respond

Return `{ signature, address, timestamp, domain, payload }` as defined in [`spec/rpc.md` § `signData`](../spec/rpc.md#signdata).

## See also

- [`spec/rpc.md` § `signData`](../spec/rpc.md#signdata) — canonical RPC definition
