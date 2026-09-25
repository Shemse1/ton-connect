# Wallet guidelines

Rules every TON Connect-compatible wallet follows. The protocol and method shapes are normative in `spec/`; this document collects the wallet-side requirements and UX guardrails the protocol implies. Per-method implementation walk-throughs live in their own guides:

- [`send-transaction.md`](./send-transaction.md)
- [`sign-message.md`](./sign-message.md)
- [`sign-data.md`](./sign-data.md)
- [`structured-items.md`](./structured-items.md)
- [`embedded-requests.md`](./embedded-requests.md)

## Connect handshake

The wallet receives a `ConnectRequest` containing `manifestUrl` and a list of [`ConnectItem`](../spec/connect.md). It MUST:

1. Fetch and parse `manifestUrl` (`tonconnect-manifest.json`). Reject with code `2` (manifest not found) or `3` (manifest content error) on failure.
2. Display the app's name, icon and origin from the manifest before prompting the user.
3. Generate the wallet's client keypair and reply with [`ConnectEvent`](../spec/connect.md) over the bridge.

If the request includes `TonProofItem`, see [`connect.md` § Address proof signature](../spec/connect.md#address-proof-signature-ton_proof) for signing rules.

## `ton_proof` prompt

The wallet SHOULD show a dedicated confirmation prompt that includes:

- dApp identity from the manifest (`name`, icon, origin / domain).
- A clear action label (for example `Sign in` or `Confirm identity`).
- A statement that this is an off-chain signature request, not an on-chain transfer.
- The proof domain and a short, readable representation of the payload / nonce.

The wallet MUST NOT present a `ton_proof` prompt as a transfer confirmation and MUST NOT show network-fee deduction for this action.

## Networks

The `network` field carries a [`NETWORK_ID`](../spec/connect.md#network_id) — a stringified TON network `global_id`. `-239` and `-3` (mainnet, testnet) are the well-known values; any other TON network `global_id` is also valid. Wallets MUST treat `network` as an opaque identifier, MUST NOT special-case unknown values to "mainnet" and MUST enforce exact string equality between the request and the active wallet network.

### Hide the testnet from regular users

Testnet exists for developers. Regular users SHOULD NOT see it.

- Switching to testnet SHOULD NOT be a one-tap action in the main UI.
- The wallet SHOULD NOT prompt the user to switch to testnet, even when the dApp is on testnet.

Users who switch by accident often cannot find the way back. dApps should host different network instances on different domains (`dapp.com`, `testnet.dapp.com`) instead of switching networks at runtime. There is no `NetworkChanged` or `ChainChanged` event in the protocol for this reason.

### Refuse cross-network requests

If the dApp's `network` field differs from the wallet's current network:

- Show an alert.
- The wallet MUST NOT allow the user to send the transaction.
- The wallet MUST NOT offer to switch networks.

This protects the user from sending real funds on mainnet when the dApp expects testnet, or vice versa.

## Multi-account

A wallet may hold many accounts derived from one keypair. Supporting this is recommended.

### No current "active" account

TON Connect is not built around a single selected account that emits `AccountChanged` events. The wallet holds many accounts; the user picks one at confirmation time, and the transaction sends from that account.

### Follow the `from` field when set

When the dApp sets `from` in `SendTransaction`:

- The wallet MUST NOT allow the user to pick a different sender.
- If sending from the requested address is impossible, show an alert and the wallet MUST NOT allow the transaction.

### Login is bound at connect time

When the dApp connects, the user picks one account. The dApp continues to work with that account regardless of which account the user later browses in the wallet.

There is no `AccountChanged` event. To switch accounts the user disconnects and reconnects inside the dApp UI. Wallets SHOULD provide a way to disconnect a specific dApp session, since a dApp's UI may be incomplete.

## Disconnect

- The dApp can initiate disconnect via the [`disconnect` RPC method](../spec/rpc.md#disconnect). The wallet MUST acknowledge and stop accepting requests for that session. The wallet SHOULD NOT emit a [disconnect event](../spec/rpc.md#disconnect-event) back to the dApp in this case.
- The wallet can initiate disconnect (for example from a session-management screen). It MUST emit a `disconnect` event so the dApp can clean up.
- Provide a UI that lists active dApp sessions and allows manual disconnect.

## Deep-link return targets

The `ret` deep-link parameter (see [`spec/deeplinks.md` § Return strategy](../spec/deeplinks.md#return-strategy-ret)) tells the wallet where to send the user after signing or declining. Standard values are `back`, `none` or a custom URL.

## Browser detection

The dApp SDK detects whether it runs inside a wallet's webview via the `isWalletBrowser` flag on the JS bridge interface. Wallet implementers expose this flag through `window.<walletJsBridgeKey>.tonconnect`.

## Registration

Register the wallet in the [wallets-list](https://github.com/ton-connect/wallets-list) repo so dApps can discover it. Submit a pull request that fills out the wallet entry. See [`spec/wallets-list.md`](../spec/wallets-list.md) for entry fields and validation rules.

## Bridge

If the wallet operates its own HTTP bridge, see [`spec/bridge.md`](../spec/bridge.md). For a quick start the common bridge `https://connect.ton.org/bridge` is available.

The wallet's side of the bridge API is not mandated by this spec.

## See also

- [`spec/connect.md`](../spec/connect.md) — connection format and `ton_proof` signing
- [`spec/rpc.md`](../spec/rpc.md) — RPC methods
- [`spec/bridge.md`](../spec/bridge.md) — transport
