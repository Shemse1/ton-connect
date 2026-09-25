# Bridge

This document is normative for dApps, wallets, bridge operators, SDKs. The keywords MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

The bridge is a transport for messages between a dApp and a wallet. Two variants are defined:

- [HTTP bridge](#http-bridge) — for external dApps and services.
- [JS bridge](#js-bridge) — for dApps opened inside a wallet's webview, or when the wallet is a browser extension.

Properties:

- The bridge is operated by the wallet provider. dApp developers do not choose or run a bridge — each wallet's bridge URL is listed in the [wallets list](./wallets-list.md).
- Messages MUST be end-to-end encrypted on the HTTP bridge. The bridge sees neither plaintext nor stable user identifiers (for example wallet addresses, account IDs or other cross-session identity markers).
- Communication is symmetrical. The bridge does not distinguish dApps from wallets — both are clients.
- The bridge keeps a separate queue of messages per recipient `client_id`.

## HTTP bridge

A client with `client_id` `A` connects to the bridge to receive incoming messages.

The paths `/events` and `/message` are relative to the bridge URL published in the [wallets list](./wallets-list.md) — for example `https://connect.ton.org/bridge` becomes `https://connect.ton.org/bridge/events`.

`client_id` is semi-private. dApps and wallets SHOULD NOT share their `client_id` with other entities to avoid having their messages removed unexpectedly.

A client MAY subscribe to multiple `client_id`s by enumerating them comma-separated: `?client_id=<A1>,<A2>,<A3>`.

### `GET /events` — subscribe

```http
GET /events?client_id=<to_hex_str(A)>[&trace_id=<uuid>][&heartbeat=<legacy|message>]
Accept: text/event-stream
```

On reconnect, pass `last_event_id` to receive every event published after that ID:

```http
GET /events?client_id=<to_hex_str(A)>&last_event_id=<lastEventId>[&trace_id=<uuid>][&heartbeat=<legacy|message>]
Accept: text/event-stream
```

### `POST /message` — send

```http
POST /message?client_id=<to_hex_str(A)>&to=<to_hex_str(B)>&ttl=300[&topic=<sendTransaction|signData|signMessage|disconnect>][&trace_id=<uuid>]

body: <base64_encoded_message>
```

The `topic` parameter is optional. If present, it SHOULD equal the RPC method inside the encrypted `message` so the bridge can route a push notification. Bridges do not validate `topic` against an enum — values not listed here are forwarded unchanged.

The bridge buffers each message up to `ttl` seconds (the sender specifies it on the request). It removes the message as soon as the recipient receives it.

If `ttl` exceeds the bridge's hard limit, the bridge MUST respond with HTTP 400. Bridges SHOULD support at least `ttl=300`.

### `trace_id` — analytics correlation

`trace_id` is an optional query parameter on `GET /events`, `POST /message` and the connect URL (see [`deeplinks.md` § Parameters](./deeplinks.md#parameters)). It carries a [UUID](https://www.rfc-editor.org/rfc/rfc9562) used to correlate logs and analytics across the dApp, bridge and wallet for a single user-visible operation. It is not security-sensitive and the bridge does not act on its value.

- Senders SHOULD use [UUIDv7](https://www.rfc-editor.org/rfc/rfc9562#name-uuid-version-7) so identifiers sort by time.
- A bridge that supports tracing MUST treat a malformed `trace_id` as absent. It SHOULD then generate a fresh UUIDv7 and use that for the request.
- A bridge MAY also generate a `trace_id` when the sender omitted it.
- A bridge that propagates tracing MUST include the resulting `trace_id` in the [`BridgeMessage` envelope](#bridgemessage-envelope) it writes to the recipient. Older or tracing-unaware bridges MAY omit the field; readers MUST tolerate its absence.
- Recipients (wallets and dApps) SHOULD reuse the inbound `BridgeMessage.trace_id` when posting their reply, so that a request and its response share an ID.

### `BridgeMessage` envelope

When the bridge forwards a message from `A` to `B`, it wraps it in a `BridgeMessage` and writes it to `B`'s SSE channel:

```json
{
  "from": "<to_hex_str(A)>",
  "message": "<base64_encoded_message>",
  "trace_id": "<uuid>"
}
```

| Field      | Required | Description |
|------------|----------|-------------|
| `from`     | yes      | Sender `client_id` as 64-character lowercase hex. |
| `message`  | yes      | Base64-encoded ciphertext: 24-byte nonce concatenated with the `nacl.box` output. See [`session.md`](./session.md#encryption). |
| `trace_id` | no       | UUID for analytics correlation. Set by bridges that support tracing; absent on older or tracing-unaware bridges. Readers MUST tolerate its absence. See [`trace_id`](#trace_id--analytics-correlation). |

See [`schemas/bridge-message.schema.json`](../schemas/bridge-message.schema.json) for the machine-readable schema.

### Heartbeat

The bridge MUST periodically send a heartbeat over each SSE connection. The format is controlled by the optional `heartbeat` query parameter on `/events`:

- `heartbeat=legacy` (default when omitted): the heartbeat is `event: heartbeat`. The browser SSE client does not expose this to JavaScript because it is not a standard `message` event.
- `heartbeat=message`: the heartbeat is `event: message`, `data: heartbeat`. Use this when the client wants to implement custom keep-alive logic.

### Operational rules

Protocol does not fix numeric global limits. Bridge operators SHOULD define and publish their limits for each deployment.

#### Rate limits

- The bridge MUST enforce request-rate limits on `GET /events` and `POST /message`.
- Limits SHOULD be applied per source IP and per `client_id`.
- Exceeding a limit MUST return HTTP `429 Too Many Requests`.
- The bridge SHOULD include `Retry-After` in `429` responses.

#### Message size limits

- The bridge MUST enforce a maximum request-body size for `POST /message` (base64-encoded ciphertext).
- Exceeding the size limit MUST return HTTP `413 Payload Too Large` or HTTP `400 Bad Request`.
- The configured limit MUST be documented by the bridge operator.

#### TTL limits

- The bridge MUST reject non-positive `ttl` values with HTTP `400`.
- If `ttl` exceeds the bridge maximum, the bridge MUST return HTTP `400`.
- Every bridge deployment SHOULD support `ttl=300` seconds or higher.
- The configured maximum `ttl` MUST be documented by the bridge operator.

#### CORS and origin policy

- Browser-facing bridge endpoints (`GET /events`, `POST /message`) MUST be callable from web dApps.
- A bridge deployment MUST either:
  - allow cross-origin requests from any origin; or
  - maintain an explicit allowlist policy that includes every supported dApp origin.
- If an origin-based deny policy is used, denied requests MUST return HTTP `403`.
- Wallet-side requests (mobile or server-side) MUST NOT rely on browser CORS behaviour.

## Deep-link transport

Universal links, unified `tc://` links, custom wallet schemes (`deepLink`), `ret` behaviour, URL-length constraints and embedded-request (`e`) wire format are specified in [`deeplinks.md`](./deeplinks.md).

Bridge implementations and SDKs MUST follow that document for deep-link parameter handling and embedded-request transport.

## JS bridge

The JS bridge is used by dApps embedded in a wallet's webview, via the injected binding `window.<walletJsBridgeKey>.tonconnect`. The `walletJsBridgeKey` value is published in the [wallets list](./wallets-list.md) as the `key` field of the wallet's JS bridge entry.

The JS bridge runs on the same device as the wallet and the dApp — communication is not encrypted. The dApp works directly with plaintext requests and responses, with no session keys.

```typescript
interface TonConnectBridge {
    deviceInfo: DeviceInfo;        // see Requests/Responses spec
    walletInfo?: WalletInfo;
    protocolVersion: number;       // max supported TON Connect version (e.g. 2)
    isWalletBrowser: boolean;      // true when the page is opened inside the wallet's browser
    connect(protocolVersion: number, message: ConnectRequest): Promise<ConnectEvent>;
    restoreConnection(): Promise<ConnectEvent>;
    send(message: AppRequest): Promise<WalletResponse>;
    listen(callback: (event: WalletEvent) => void): () => void;
}
```

Like the HTTP bridge, the wallet side ignores dApp requests until the user confirms the session. Messages arrive from the webview into the bridge controller and are silently dropped before that point.

### `walletInfo` (optional)

Wallet metadata. Use it to make an injectable wallet work with TON Connect even when the wallet is not yet listed in the [wallets list](./wallets-list.md).

```typescript
interface WalletInfo {
    name: string;
    image: string;       // PNG image URL
    tondns?: string;
    about_url: string;   // about page URL
}
```

If `TonConnectBridge.walletInfo` is set and the wallet is also listed in the [wallets list](./wallets-list.md), the values in `walletInfo` override those in the registry.

### `connect()`

Initiates a connect request. Equivalent to the QR/link flow on the HTTP bridge.

If the dApp was previously approved for the current account, the bridge connects silently and emits a `ConnectEvent`. Otherwise it shows the confirmation dialog.

The dApp SHOULD NOT call `connect()` without an explicit user action (e.g. a click on a Connect button). To restore a prior connection automatically, use `restoreConnection()`.

### `restoreConnection()`

Attempts to restore a prior connection.

If the dApp was previously approved for the current account, the bridge connects silently and emits a `ConnectEvent` with only a `ton_addr` data item.

Otherwise it returns `ConnectEventError` with code `100` (`UNKNOWN_APP`).

### `send()`

Sends an [`AppRequest`](./rpc.md#apprequest) to the bridge — every method except `ConnectRequest` (which goes into the QR code on the HTTP bridge and into `connect()` on the JS bridge). Returns a promise that resolves with the `WalletResponse` directly. The caller does not need `listen()` to await the reply.

### `listen()`

Registers a listener for wallet-initiated events. Returns an unsubscribe function. Currently `disconnect` is the only event. Future versions may add more.

## See also

- [`session.md`](./session.md) — end-to-end encryption on top of the HTTP bridge
- [`connect.md`](./connect.md) — `ConnectRequest` / `ConnectEvent` shapes
- [`rpc.md`](./rpc.md) — RPC methods and `WalletResponse`
- [`deeplinks.md`](./deeplinks.md) — universal link details
