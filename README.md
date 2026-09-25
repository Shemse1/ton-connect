# TON Connect Specification

TON Connect is the protocol that connects a [TON](https://ton.org) wallet to a decentralised application. It defines how a dApp discovers the wallet, how the wallet authenticates the user's account and how subsequent requests travel end-to-end encrypted over an untrusted relay.

## Building a dApp

If you only want to integrate TON Connect into a dApp, follow the dApp-developer documentation instead of reading the protocol spec directly.

- [TON Connect overview](https://docs.ton.org/ecosystem/ton-connect/overview)
- [TON Connect for dApps](https://docs.ton.org/ecosystem/ton-connect/react)

If no SDK exists for your language, take the [JS SDK](https://github.com/ton-connect/sdk/) as a reference and implement your own wrapper.

## Layout

- [`spec/`](./spec) — normative protocol specification. Start with [`spec/overview.md`](./spec/overview.md) for the architecture, message flow and conformance levels.
- [`guides/`](./guides) — wallet implementation guides.
- [`schemas/`](./schemas) — machine-readable JSON Schema artifacts for the app manifest, wallets list, bridge envelope and RPC envelopes.
- [`GLOSSARY.md`](./GLOSSARY.md) — every defined term in one place.
- [`CHANGELOG.md`](./CHANGELOG.md) — protocol-affecting changes, calendar-versioned.
- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — how to propose a change.

## Companion repositories

- [`ton-connect/sdk`](https://github.com/ton-connect/sdk) — TypeScript SDK packages: `@tonconnect/sdk`, `@tonconnect/ui`, `@tonconnect/ui-react`.
- [`ton-connect/bridge`](https://github.com/ton-connect/bridge) — Go reference implementation of the HTTP bridge.
- [`ton-connect/wallets-list`](https://github.com/ton-connect/wallets-list) — public registry of TON Connect-compatible wallets (`wallets-v2.json`).

## Where to start

| You are                                  | Start with                                                                                                       |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Integrating TON Connect into a dApp      | the SDK at [`ton-connect/sdk`](https://github.com/ton-connect/sdk)                                               |
| Implementing a wallet                    | [`spec/overview.md`](./spec/overview.md), then [`guides/wallet-guidelines.md`](./guides/wallet-guidelines.md)    |
| Operating a bridge                       | [`spec/bridge.md`](./spec/bridge.md)                                                                             |
| Looking up a message format              | [`schemas/`](./schemas) and the matching [`spec/`](./spec) page                                                  |
| Proposing a protocol change              | [`CONTRIBUTING.md`](./CONTRIBUTING.md)                                                                           |

## Q&A

### What should I read if I am building an HTML/JS app?

Use the [TON Connect documentation](https://docs.ton.org/ecosystem/ton-connect/overview) and do not worry about the underlying protocol.

### How do I add TON Connect support to a language without an SDK?

Take the JS SDK as a reference and check out the protocol pages under [`spec/`](./spec).

### How do I detect whether the dApp is embedded in the wallet?

The JS SDK does that for you — get the wallets list with `connector.getWallets()` and check the `embedded` property of the matching entry. If you build your own SDK, check `window.<walletJsBridgeKey>.tonconnect.isWalletBrowser`. See [`spec/bridge.md` § JS bridge](./spec/bridge.md#js-bridge).

### How do I detect whether the wallet is a browser extension?

Same as the embedded-dApp case. The JS SDK reports it via the `injected` property of the `connector.getWallets()` entry. If you build your own SDK, check that `window.<walletJsBridgeKey>.tonconnect` exists.

### How do I implement backend authorisation with TON Connect?

See the [`ton-connect/demo-dapp-backend`](https://github.com/ton-connect/demo-dapp-backend) reference and [`spec/connect.md` § Address proof signature](./spec/connect.md#address-proof-signature-ton_proof) for the signature format and verification flow.

### How do I run my own bridge?

You do not need to unless you are building a wallet.

If you build a wallet, you will need to provide a bridge. See the [reference Go implementation](https://github.com/ton-connect/bridge) and [`spec/bridge.md`](./spec/bridge.md). For a quick start, the common bridge at `https://connect.ton.org/bridge` is available. The wallet's side of the bridge API is not mandated by this spec.

### I make a wallet — how do I add it to the wallets list?

Submit a pull request to [`ton-connect/wallets-list`](https://github.com/ton-connect/wallets-list) and fill out your wallet's entry. See [`spec/wallets-list.md`](./spec/wallets-list.md) for the entry-field reference.

## Licence

See [`LICENCE`](./LICENCE).
