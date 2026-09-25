# Glossary

One-line definitions for terms used across the TON Connect spec. Entries link to the canonical page.

**`AppDomain`** — dApp domain bound into `ton_proof` and `signData` signatures. See [`spec/connect.md`](./spec/connect.md#address-proof-signature-ton_proof).

**`AppRequest`** — JSON-RPC-style envelope a dApp sends to a wallet. See [`spec/rpc.md`](./spec/rpc.md#apprequest).

**Bridge** — Transport carrying TON Connect messages between a dApp and a wallet; comes in HTTP and JS flavours. See [`spec/bridge.md`](./spec/bridge.md).

**`BridgeMessage`** — Envelope the HTTP bridge writes onto the recipient's SSE stream. See [`spec/bridge.md`](./spec/bridge.md#bridgemessage-envelope).

**Client keypair** — X25519 `nacl.box` keypair each side generates per session. See [`spec/session.md`](./spec/session.md#client-keypair).

**`client_id`** — 32-byte X25519 public key identifying one side of a session, hex-encoded for transport. See [`spec/session.md`](./spec/session.md#client_id).

**Connect event** — Wallet's reply to a `ConnectRequest`: `connect` on success, `connect_error` on failure. See [`spec/connect.md`](./spec/connect.md#connectevent).

**Connect request** — First message a dApp sends, delivered via deep link or QR code. See [`spec/connect.md`](./spec/connect.md#connectrequest).

**`ConnectItem`** — Data item the dApp asks the wallet to share at connect time (`ton_addr`, `ton_proof`). See [`spec/connect.md`](./spec/connect.md#connectrequest).

**dApp** — Decentralised application that integrates TON Connect. See [`spec/overview.md`](./spec/overview.md#roles).

**Deep link** — URI form (`tc://...`, `tonkeeper-tc://...`) that opens a wallet on mobile. See [`spec/deeplinks.md`](./spec/deeplinks.md).

**`deepLink`** — Optional wallets-list field carrying a wallet-specific custom-scheme link. See [`spec/wallets-list.md`](./spec/wallets-list.md#entry-fields).

**`DeviceInfo`** — Wallet's self-description returned in the connect event. See [`spec/connect.md`](./spec/connect.md#deviceinfo).

**`e` parameter** — Optional deep-link query parameter carrying an embedded request. See [`spec/deeplinks.md`](./spec/deeplinks.md#embedded-requests-e).

**Embedded request** — RPC request packed into the connect URL so the wallet handles connection and action in one tap. See [`spec/deeplinks.md`](./spec/deeplinks.md#embedded-requests-e).

**`EmbeddedWireRequest`** — Compact wire format for an embedded request, with abbreviated field names. See [`spec/deeplinks.md`](./spec/deeplinks.md#embedded-requests-e).

**Feature flag** — Entry in `DeviceInfo.features` advertising a wallet capability: `SendTransaction`, `SignData`, `SignMessage`, `EmbeddedRequest`. See [`spec/connect.md`](./spec/connect.md#feature).

**`from`** — Optional sender address pinning the signing account on `sendTransaction`, `signMessage`, `signData`. See [`spec/rpc.md`](./spec/rpc.md#sendtransaction).

**Heartbeat** — Periodic SSE keep-alive emitted by the HTTP bridge. See [`spec/bridge.md`](./spec/bridge.md#heartbeat).

**HTTP bridge** — Server-relayed, end-to-end-encrypted bridge transport. See [`spec/bridge.md`](./spec/bridge.md#http-bridge).

**`id`** — Monotonically increasing identifier on `AppRequest`, `WalletResponse` and `WalletEvent`. See [`spec/rpc.md`](./spec/rpc.md#envelope).

**`id` parameter** — Deep-link query parameter carrying the dApp's `client_id` as hex. See [`spec/deeplinks.md`](./spec/deeplinks.md#parameters).

**Item** — Structured `TonItem`, `JettonItem` or `NftItem` in a `sendTransaction` or `signMessage` payload. See [`spec/rpc.md`](./spec/rpc.md#structured-items).

**JS bridge** — In-wallet browser transport on `window.<walletJsBridgeKey>.tonconnect`; plaintext, because the dApp and the wallet run on the same device. See [`spec/bridge.md`](./spec/bridge.md#js-bridge).

**`last_event_id`** — Query parameter on `GET /events` for resuming an SSE stream. See [`spec/bridge.md`](./spec/bridge.md#http-bridge).

**Manifest (app)** — JSON file the wallet fetches before the connect prompt; describes the dApp. See [`spec/manifest.md`](./spec/manifest.md).

**`manifestUrl`** — HTTPS URL the dApp publishes in `ConnectRequest`. See [`spec/manifest.md`](./spec/manifest.md).

**`maxProtocolVersion`** — Highest TON Connect protocol version the wallet supports; currently `2`. See [`spec/connect.md`](./spec/connect.md#deviceinfo).

**`NETWORK_ID`** — TON network `global_id` as a stringified integer carried on every `network` field; `-239` (mainnet) and `-3` (testnet) are the well-known values defined by the `NETWORK` enum, and any other valid TON network `global_id` is also a valid `NETWORK_ID`. See [`spec/connect.md`](./spec/connect.md#network_id).

**Nonce** — 24 fresh CSPRNG bytes prepended to each `nacl.box` ciphertext. See [`spec/session.md`](./spec/session.md#nonce-uniqueness).

**`r` parameter** — Deep-link query parameter carrying the `ConnectRequest` as URL-safe JSON. See [`spec/deeplinks.md`](./spec/deeplinks.md#parameters).

**`ret` / return strategy** — Deep-link query parameter telling the wallet where to send the user after signing. See [`spec/deeplinks.md`](./spec/deeplinks.md#return-strategy-ret).

**Send mode 3** — TVM `SENDRAWMSG` mode `PAY_GAS_SEPARATELY + IGNORE_ERRORS` used by `signMessage`. See [`spec/rpc.md`](./spec/rpc.md#signmessage).

**Session** — The pair of `client_id`s identifying one dApp-wallet relationship. See [`spec/session.md`](./spec/session.md).

**SSE** — Server-Sent Events; the streaming transport used by `GET /events`. See [`spec/bridge.md`](./spec/bridge.md#http-bridge).

**`StateInit`** — Initial code-and-data cell for a TON contract, carried as `walletStateInit` in `TonAddressItemReply` or as `Message.stateInit`. See [`spec/connect.md`](./spec/connect.md#verification).

**`tc://`** — Unified deep-link scheme every wallet MUST support. See [`spec/deeplinks.md`](./spec/deeplinks.md#unified-deep-link-tc).

**`ton_addr`** — `ConnectItem` requesting the connected account's address, network, public key and `walletStateInit`. See [`spec/connect.md`](./spec/connect.md#connectitemreply).

**`ton_proof`** — Signed message proving the user owns the connected wallet address. See [`spec/connect.md`](./spec/connect.md#address-proof-signature-ton_proof).

**`tondns`** — Optional wallets-list field naming the wallet's TON DNS entry; reserved. See [`spec/wallets-list.md`](./spec/wallets-list.md#entry-fields).

**`topic`** — Optional `POST /message` parameter naming the RPC method for push routing. See [`spec/bridge.md`](./spec/bridge.md#post-message--send).

**`trace_id`** — Optional UUID (UUIDv7 recommended) for analytics correlation, carried as a query parameter on bridge endpoints and connect links and echoed in the `BridgeMessage` envelope; tracing-unaware bridges MAY omit the field, and readers MUST tolerate its absence. See [`spec/bridge.md`](./spec/bridge.md#trace_id--analytics-correlation).

**`ttl`** — Required `POST /message` parameter — seconds the bridge buffers the message. See [`spec/bridge.md`](./spec/bridge.md#ttl-limits).

**Universal link** — Wallet-specific HTTPS link listed as `universal_url` in the wallets list. See [`spec/deeplinks.md`](./spec/deeplinks.md#universal-link).

**`v` parameter** — Deep-link query parameter naming the protocol version (currently `2`). See [`spec/deeplinks.md`](./spec/deeplinks.md#parameters).

**`valid_until`** — Optional payload field on `sendTransaction` and `signMessage`; unix seconds after which the wallet MUST reject. See [`spec/rpc.md`](./spec/rpc.md#sendtransaction).

**Wallet** — Application holding the user's keys and signing on their behalf. See [`guides/wallet-guidelines.md`](./guides/wallet-guidelines.md).

**`WalletEvent`** — Wallet-initiated message: `connect`, `connect_error` or `disconnect`. See [`spec/rpc.md`](./spec/rpc.md#walletevent).

**`WalletResponse`** — Wallet's reply to a specific `AppRequest`, matched by `id`. See [`spec/rpc.md`](./spec/rpc.md#walletresponse).

**`walletStateInit`** — Base64-encoded `StateInit` cell in `TonAddressItemReply`, used to recover the public key. See [`spec/connect.md`](./spec/connect.md#verification).

**Wallets list** — Public registry at [`ton-connect/wallets-list`](https://github.com/ton-connect/wallets-list); served as `wallets-v2.json`. See [`spec/wallets-list.md`](./spec/wallets-list.md).
