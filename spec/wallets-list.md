# Wallets list

This document is normative for dApps, wallets, bridge operators, SDKs. The keywords MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

The wallets list is a public JSON registry of TON Connect-compatible wallets. The canonical file is [`wallets-v2.json`](https://github.com/ton-connect/wallets-list/blob/main/wallets-v2.json). The schema is [`wallets-v2.schema.json`](../schemas/wallets-v2.schema.json).

The SDK fetches the file at runtime to render the wallet picker.

## Entry fields

| Field           | Required    | Type     | Description                                                                                                                                                        |
|-----------------|-------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `app_name`      | yes         | string   | Wallet identifier (non-empty). MUST match the wallet's runtime `DeviceInfo.appName`. When the entry includes a `js` bridge, SHOULD also equal that bridge's `key`. |
| `name`          | yes         | string   | Display name shown in the wallet picker (non-empty).                                                                                                               |
| `image`         | yes         | string   | HTTPS URL of a PNG icon — MUST end in `.png` (lowercase extension).                                                                                                |
| `about_url`     | yes         | string   | HTTPS URL to the wallet's info or landing page.                                                                                                                    |
| `universal_url` | conditional | string   | HTTPS base for the wallet's universal link. REQUIRED when the entry lists an `sse` bridge; MAY be omitted otherwise.                                               |
| `deepLink`      | conditional | string   | Custom-scheme deep link, e.g. `tonkeeper-tc://`. SHOULD be provided when the entry lists an `sse` bridge; MAY be omitted otherwise.                                |
| `tondns`        | no          | string   | TON DNS name ending in `.ton`. Reserved for future use by the protocol.                                                                                            |
| `bridge`        | yes         | array    | One or two bridge entries — see [`bridge`](#bridge).                                                                                                               |
| `platforms`     | yes         | string[] | Platforms the wallet supports — see [`platforms`](#platforms).                                                                                                     |
| `features`      | yes         | array    | TON Connect feature entries — see [`features`](#features).                                                                                                         |

The `image` icon SHOULD be 288×288 px, on a non-transparent background, without rounded corners. Authors SHOULD compress the PNG with `pngquant`, ImageMagick or TinyPNG to reduce bandwidth.

### `bridge`

The `bridge` array MUST contain one or two entries, with at most one entry of each type. Each entry takes one of two shapes:

```json
[
  { "type": "sse", "url": "https://connect.ton.org/bridge" },
  { "type": "js",  "key": "tonkeeper" }
]
```

| Type   | Required fields | Meaning                                                     |
|--------|-----------------|-------------------------------------------------------------|
| `sse`  | `url`           | HTTPS URL of the wallet's HTTP (SSE) bridge. See [`bridge.md` § HTTP bridge](./bridge.md#http-bridge). |
| `js`   | `key`           | The `window` property name where the wallet exposes its JS bridge object. With `key: "tonkeeper"`, the `TonConnectBridge` object is reachable as `window.tonkeeper.tonconnect`. See [`bridge.md` § JS bridge](./bridge.md#js-bridge). |

A wallet MAY list both — `js` for the embedded-browser case and `sse` for everything else.

When a wallet lists an `sse` bridge it MUST provide `universal_url` and SHOULD provide `deepLink`. A wallet that lists only a `js` bridge — typically a browser extension — MAY omit both.

### `platforms`

Allowed values: `ios`, `android`, `chrome`, `firefox`, `safari`, `macos`, `windows`, `linux`. The array MUST contain at least one platform; entries MUST be unique. The SDK filters the picker by the user's detected platform.

The values group as:

- mobile apps: `ios`, `android`.
- desktop apps: `macos`, `windows`, `linux`.
- browser extensions: `chrome`, `firefox`, `safari`.

### `features`

The `features` array advertises which TON Connect capabilities the wallet supports. The array MUST contain at least one entry.

The schema accepts four feature shapes:

```typescript
type ListedFeature =
  | {
      name: 'SendTransaction';
      maxMessages: number;            // integer ≥ 1
      extraCurrencySupported?: boolean;
      itemTypes?: ('ton' | 'jetton' | 'nft')[]; // ≥ 1 unique entry when present
    }
  | {
      name: 'SignData';
      types: ('text' | 'binary' | 'cell')[]; // ≥ 1 unique entry
    }
  | {
      name: 'SignMessage';
      maxMessages: number;            // integer ≥ 1
      extraCurrencySupported?: boolean;
      itemTypes?: ('ton' | 'jetton' | 'nft')[]; // ≥ 1 unique entry when present
    }
  | {
      name: 'EmbeddedRequest';
    };
```

| Feature           | Meaning                                                                                                   |
|-------------------|-----------------------------------------------------------------------------------------------------------|
| `SendTransaction` | Wallet accepts the `sendTransaction` RPC. `maxMessages` caps the number of outbound messages per request. `extraCurrencySupported` indicates support for extra-currency transfers. `itemTypes` lists which structured-item kinds (`ton`, `jetton`, `nft`) the wallet accepts; absent means only raw `messages` are supported. |
| `SignData`        | Wallet accepts the `signData` RPC for the listed payload `types`. See [`rpc.md` § `signData`](./rpc.md#signdata). |
| `SignMessage`     | Wallet accepts the `signMessage` RPC — signs an internal message without broadcasting, enabling gasless flows where a relayer submits the signed BoC. Shape mirrors `SendTransaction`. See [`rpc.md` § `signMessage`](./rpc.md#signmessage). |
| `EmbeddedRequest` | Wallet handles a request embedded in the connect URL via the `e` query parameter, allowing connect and one RPC call in a single tap. See [`deeplinks.md` § Embedded requests](./deeplinks.md#embedded-requests-e). |

Every required field listed for a feature shape MUST be present; entries MUST NOT include other keys.

The wallets-list entry advertises *static* capability — what the wallet binary supports. A wallet's runtime `DeviceInfo.features` is authoritative for the connected session. SDKs SHOULD refuse RPC methods that the runtime `DeviceInfo.features` does not list, even when the wallets-list entry claims support.

## How a wallet gets listed

1. Implement the protocol — at minimum the [connect handshake](./connect.md) and [`sendTransaction`](./rpc.md#sendtransaction). [`guides/wallet-guidelines.md`](../guides/wallet-guidelines.md) covers the wallet-side UX baseline.
2. Open a pull request against [`ton-connect/wallets-list`](https://github.com/ton-connect/wallets-list) that appends your entry to the end of `wallets-v2.json`.
3. The PR runs schema validation against `wallets-v2.schema.json`. The entry MUST validate.
4. After merge, the new entry is served to all SDKs that consume `wallets-v2.json`.

## Versioning

The list is versioned by filename. The current file is `wallets-v2.json` and the current schema is `wallets-v2.schema.json`. The schema itself carries no version field — implementations identify the version by the filename they fetched.

## See also

- [`schemas/wallets-v2.schema.json`](../schemas/wallets-v2.schema.json) — current schema
- [`connect.md`](./connect.md) — `DeviceInfo.features`
- [`deeplinks.md`](./deeplinks.md) — `universal_url` and `deepLink` are consumed here
- [`bridge.md`](./bridge.md) — `bridge` entries are consumed here
