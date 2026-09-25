# Connect

This document is normative for dApps, wallets, bridge operators, SDKs. The keywords MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

The connect flow is the first exchange in a TON Connect session. The dApp sends a `ConnectRequest`. The wallet replies with a `ConnectEvent`. After a successful connect, both sides know each other's `client_id` and the wallet has shared the requested data items.

## `ConnectRequest`

```typescript
type ConnectRequest = {
  manifestUrl: string;
  items: ConnectItem[]; // data items to share with the app
}

type ConnectItem = TonAddressItem | TonProofItem;

type TonAddressItem = {
  name: "ton_addr";
  network?: NETWORK_ID; // optional TON network global_id the dApp wants to connect to
}

type TonProofItem = {
  name: "ton_proof";
  payload: string; // arbitrary payload, e.g. nonce + expiration timestamp
}
```

| Field        | Description                                                      |
|--------------|------------------------------------------------------------------|
| `manifestUrl` | Link to the dApp's `tonconnect-manifest.json` (see [`manifest.md`](./manifest.md)). |
| `items`      | Data items to share with the dApp.                               |

Wallets MUST reply with a per-item error (code 400) for any requested item they do not support — see [Connect item error codes](#connect-item-error-codes).

## `ConnectEvent`

The wallet responds with a `ConnectEvent` if the user approves the request. On rejection it returns `ConnectEventError`.

```typescript
type ConnectEvent = ConnectEventSuccess | ConnectEventError;

type ConnectEventSuccess = {
  event: "connect";
  id: number; // increasing event counter
  payload: {
    items: ConnectItemReply[];
    device: DeviceInfo;
  }
  response?: WalletResponse; // present when an embedded request was processed; see deeplinks.md
}

type ConnectEventError = {
  event: "connect_error";
  id: number;
  payload: {
    code: number;
    message: string;
  }
}
```

The `response` field is present only when the connect URL carried an [embedded request](./deeplinks.md#embedded-requests-e). See [`deeplinks.md` § Embedded requests](./deeplinks.md#embedded-requests-e) for the semantics.

### `DeviceInfo`

```typescript
type DeviceInfo = {
  platform: 'iphone' | 'ipad' | 'android' | 'windows' | 'mac' | 'linux' | 'browser';
  appName: string;            // e.g. "tonkeeper"
  appVersion: string;         // e.g. "2.3.367"
  maxProtocolVersion: number;
  features: Feature[];        // supported features and RPC methods
}
```

`appName` MUST equal the wallet's [`app_name`](./wallets-list.md#entry-fields) entry in the wallets list.

### `Feature`

```typescript
type Feature =
  | {
      name: 'SendTransaction';
      maxMessages: number;                       // max messages in one `SendTransaction` the wallet supports
      extraCurrencySupported?: boolean;          // whether the wallet supports extra currencies
      itemTypes?: ('ton' | 'jetton' | 'nft')[];  // supported structured item types; absent ⇒ only raw `messages`
    }
  | {
      name: 'SignData';
      types: ('text' | 'binary' | 'cell')[];     // supported data types for signing
    }
  | {
      name: 'SignMessage';
      maxMessages: number;                       // max messages in one `SignMessage` the wallet supports
      extraCurrencySupported?: boolean;          // whether the wallet supports extra currencies
      itemTypes?: ('ton' | 'jetton' | 'nft')[];  // supported structured item types; absent ⇒ only raw `messages`
    }
  | {
      name: 'EmbeddedRequest';                   // wallet can process the `e` parameter in the connect URL
    };
```

Wallets MUST list every feature they implement. SDKs MUST treat absent features as unsupported and refuse to send the corresponding request.

### `ConnectItemReply`

```typescript
type ConnectItemReply = TonAddressItemReply | TonProofItemReply;

// Untrusted data returned by the wallet. To prove the user owns this address
// and public key, request a `ton_proof` item.
type TonAddressItemReply = {
  name: "ton_addr";
  address: string;          // TON address raw (`0:<hex>`)
  network: NETWORK_ID;      // TON network global_id of the connected account
  publicKey: string;        // hex string without 0x
  walletStateInit: string;  // base64 (not url-safe) state-init cell for the wallet contract
}

type TonProofItemReply = TonProofItemReplySuccess | TonProofItemReplyError;

type TonProofItemReplySuccess = {
  name: "ton_proof";
  proof: {
    timestamp: string;      // 64-bit unix epoch time of the signing operation (seconds)
    domain: {
      lengthBytes: number;  // AppDomain length
      value: string;        // app domain name (URL part, without encoding)
    };
    signature: string;      // base64-encoded signature
    payload: string;        // payload from the request
  }
}

type TonProofItemReplyError = {
  name: "ton_proof";
  error: {
    code: ConnectItemErrorCode;
    message?: string;
  }
}
```

See [Address proof signature (`ton_proof`)](#address-proof-signature-ton_proof) for the signature format and verification flow.

### `NETWORK_ID`

```typescript
enum NETWORK {
  MAINNET = '-239',
  TESTNET = '-3'
}

type NETWORK_ID = NETWORK | string;
```

`NETWORK_ID` is the type carried on every `network` field in TON Connect — `TonAddressItem.network`, `TonAddressItemReply.network` and the `network` parameter of `sendTransaction`, `signMessage` and `signData`. It is a stringified TON network global_id. The `NETWORK` enum names the two baseline values; any other TON network global_id is also valid and MUST be passed through verbatim by SDKs and wallets.

## Connect event error codes

| Code | Name                     | Description                  |
|------|--------------------------|------------------------------|
| 0    | `UNKNOWN_ERROR`          | Unknown error.               |
| 1    | `BAD_REQUEST`            | Bad request.                 |
| 2    | `MANIFEST_NOT_FOUND`     | App manifest not found.      |
| 3    | `MANIFEST_CONTENT_ERROR` | App manifest content error.  |
| 100  | `UNKNOWN_APP`            | Unknown app.                 |
| 300  | `USER_DECLINED`          | User declined the connection.|
| 400  | `METHOD_NOT_SUPPORTED`   | Method not supported.        |

## Connect item error codes

| Code | Name                   | Description           |
|------|------------------------|-----------------------|
| 0    | `UNKNOWN_ERROR`        | Unknown error.        |
| 400  | `METHOD_NOT_SUPPORTED` | Method not supported. |

If the wallet does not support a requested `ConnectItem` (for example `ton_proof`), it MUST reply with a per-item error of the matching shape:

```typescript
type ConnectItemReplyError = {
  name: "<requested-connect-item-name>";
  error: {
    code: 400;
    message?: string;
  }
}
```

## Address proof signature (`ton_proof`)

When a `ConnectRequest` includes a `TonProofItem`, the wallet proves ownership of the selected account's key. dApps use the proof to authenticate the user — typically as a login step against an off-chain backend.

### Bound fields

The signed message binds:

- A unique prefix to separate this signature from on-chain messages (`ton-connect`).
- The wallet address.
- The dApp's domain (from the manifest).
- A signing timestamp.
- The dApp's custom payload (typically a server-issued nonce, cookie ID and expiration time).

### Byte layout

```text
message = utf8_encode("ton-proof-item-v2/") ++
          Address ++
          AppDomain ++
          Timestamp ++
          Payload
```

| Field      | Encoding                                                                 |
|------------|--------------------------------------------------------------------------|
| `Address`  | `workchain` (32-bit signed, big-endian) ++ `hash` (256-bit unsigned, big-endian) |
| `AppDomain`| `Length` (32-bit unsigned, little-endian) ++ `EncodedDomainName` (UTF-8) |
| `Timestamp`| 64-bit unsigned, little-endian, unix seconds at signing time             |
| `Payload`  | Variable-length opaque bytes                                              |

`Payload` is variable-length untrusted data. To avoid an unnecessary length prefix, it is placed last.

### Signature

```text
signature = Ed25519Sign(privkey, sha256(0xffff ++ utf8_encode("ton-connect") ++ sha256(message)))
```

### Domain-binding rules

For standard dApp connections the domain name MUST contain at least one `.` character with valid characters on both sides. Names without proper dot separation — for example the literal strings `tonkeeper` or `tonhub` — are reserved for native wallet integrations and MUST NOT be accepted from external dApps.

### Verification

Verifying a `TonProofItemReply` involves four steps:

1. **Try to extract the public key from `walletStateInit`.**
   1. Parse `TonAddressItemReply.walletStateInit` as a `StateInit` cell.
   2. Compare `walletStateInit.code` against known wallet contract codes. On a match, parse `walletStateInit.data` according to that wallet version.
   3. Take `publicKey` from the parsed data.
2. **If the public key cannot be extracted locally, fall back to an on-chain call.**
   1. Call `get_public_key` on the contract at `TonAddressItemReply.address`.
   2. Take `publicKey` from the result.
3. **Verify the public key and address.**
   1. Check that `TonAddressItemReply.publicKey` matches the key extracted from `walletStateInit` or `get_public_key`.
   2. Check that `TonAddressItemReply.walletStateInit.hash()` equals `TonAddressItemReply.address.hash()` (BoC hash).
4. **Verify the signature.**
   1. Reconstruct the message bytes (see [Byte layout](#byte-layout)) and compute the `hash` (see [Signature](#signature)).
   2. Verify that `proof.signature` corresponds to `hash` and `publicKey`.

The local-parsing path is preferred. The on-chain fallback exists for wallets whose contract code is not yet recognised by the verifier. Prioritising local parsing reduces RPC calls.

## Conformance

### Connect items

Wallets MUST support the `ton_addr` connect item — without it a connect cannot complete. Wallets SHOULD support the `ton_proof` connect item; a wallet that does not support it MUST reply with a per-item error of code 400 for that item (see [Connect item error codes](#connect-item-error-codes)), and the connect itself still succeeds.

### Connected account is fixed for the session

The `TonAddressItemReply.address` returned in the connect event is bound to the session for its entire lifetime.

- A wallet MAY hold many accounts derived from one keypair. At connect time the user picks one; only that account is exposed to the dApp.
- The exposed account MUST NOT change for the lifetime of the session.
- To switch accounts, the user MUST disconnect and reconnect inside the dApp UI.

## See also

- [`manifest.md`](./manifest.md) — required and optional manifest fields
- [`rpc.md`](./rpc.md) — methods invoked after connect
- [`deeplinks.md`](./deeplinks.md) — embedded request encoding
- [`bridge.md`](./bridge.md) — transport
- [Wallet guidelines](../guides/wallet-guidelines.md) — wallet UX rules for the proof prompt
