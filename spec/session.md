# Session

This document is normative for dApps, wallets, bridge operators, SDKs. The keywords MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

The session protocol defines client identifiers and end-to-end encryption between a dApp and a wallet. The HTTP bridge is fully untrusted: it sees only ciphertext and the routing addresses on the URL. The [JS bridge](./bridge.md#js-bridge) does not use this protocol, because the dApp and the wallet run on the same device.

## Client keypair

Each side generates an X25519 keypair for use with the NaCl `crypto_box` protocol.

```text
(a, A) <- nacl.box.keyPair()
```

`a` is the 32-byte secret key. `A` is the 32-byte public key. The dApp and the wallet each generate one keypair per session.

## `client_id`

The public key part of the [client keypair](#client-keypair) (32 bytes). On the bridge side, `client_id` is the 64-character lowercase hex encoding of `A` — see [`bridge.md`](./bridge.md#http-bridge).

## Session

A session is the pair of two `client_id` values: one for the dApp and one for the wallet. Each side exchanges its `client_id` during the connect flow described in [`connect.md`](./connect.md). Once exchanged, the pair MUST NOT change for the lifetime of the session — generating a new keypair produces a new session.

## Encryption

Every message exchanged on the HTTP bridge MUST be encrypted under this protocol. The initial `ConnectRequest` travels in a deep link or QR code (see [`deeplinks.md`](./deeplinks.md)) before bridge keys are established and is therefore out of scope for the encryption rule.

Given a binary message `m`, the recipient's `client_id` `X` and the sender's secret key `y`, the encrypted message `M` is constructed as:

```text
nonce <- random(24 bytes)
ct    <- nacl.box(m, nonce, X, y)
M     <- nonce ++ ct
```

The first 24 bytes of `M` are the nonce; the remainder is the ciphertext returned by `nacl.box`.

## Decryption

The recipient uses its secret key `x` and the sender's `client_id` `Y`:

```text
nonce <- M[0..24]
ct    <- M[24..]
m     <- nacl.box.open(ct, nonce, Y, x)
```

The recovered plaintext `m` is parsed per [`rpc.md`](./rpc.md). If `nacl.box.open` fails — wrong key, truncated input or tampered ciphertext — the recipient MUST discard the message and MUST NOT process it as plaintext.

## Nonce uniqueness

The nonce MUST be 24 fresh bytes drawn from a cryptographically secure random number generator. Implementations MUST NOT derive the nonce from a counter, a timestamp or any other predictable source. Under uniform random generation the probability of nonce collision within a session is negligible.

## Session lifetime

A session lives until one side discards its keypair. Termination is signalled at the RPC layer through the [`disconnect`](./rpc.md#disconnect) method (dApp-initiated) or the [disconnect event](./rpc.md#disconnect-event) (wallet-initiated). After either signal both sides SHOULD erase the corresponding secret key and MUST stop processing further messages from the peer `client_id`.

Wallets MAY also retire a session unilaterally — for example when the user removes the dApp from the wallet's session list. The session protocol itself imposes no fixed expiry; wallet retention rules are out of scope here.

## See also

- [`bridge.md`](./bridge.md) — transport that carries the encrypted ciphertext.
- [`connect.md`](./connect.md) — handshake during which `client_id` values are exchanged.
- [`rpc.md`](./rpc.md) — request and event format inside the encrypted envelope.
- [`deeplinks.md`](./deeplinks.md) — deep-link transport for the initial `ConnectRequest`.
