# Changelog

This changelog tracks protocol-affecting changes to TON Connect — new RPC methods, new feature flags, new connect-event fields, breaking serialization edits.

Format: each section starts with a date heading `## Month Day, Year`, followed by a bold version line `**TON Connect <YYYY.MM.DD>**`. Bullets begin with a past-tense verb: **Added**, **Replaced**, **Allowed**, **Fixed**, **Removed**, **Deprecated**. Each bullet names the affected identifier in `code font`. One bullet per change. No paragraphs.

## May 18, 2026

**TON Connect 2026.05.18**

- **Added** optional `trace_id` query parameter to bridge `GET /events` and `POST /message` for analytics correlation. UUIDv7 is recommended.
- **Added** optional `trace_id` query parameter to connect URLs for end-to-end analytics correlation.
- **Added** optional `trace_id` field to the `BridgeMessage` envelope, echoing the per-request UUID to the recipient.
- **Added** `NETWORK_ID` type (`NETWORK | string`) carried on every `network` field; any TON network `global_id` is now valid, not only `-239` and `-3`.
- **Added** optional `network` field on `TonAddressItem` so a dApp can request connection to a specific TON network.
- **Replaced** `network: NETWORK` on `TonAddressItemReply` and the `network` parameter of `sendTransaction`, `signMessage` and `signData` with `NETWORK_ID`.
- **Added** `signMessage` RPC method.
- **Added** `SignMessage` feature entry.
- **Added** structured `items` arrays to `sendTransaction` and `signMessage`, with `ton`, `jetton` and `nft` discriminator types, plus the matching `itemTypes` Feature parameter.
- **Added** `EmbeddedRequest` feature flag, the `e` query parameter on connect URLs and the `response` field on `ConnectEventSuccess`.

## September 3, 2025

**TON Connect 2025.09.03**

- **Added** optional `heartbeat=<legacy|message>` query parameter to bridge `GET /events` for opt-in standard SSE heartbeat delivery.

## August 12, 2025

**TON Connect 2025.08.12**

- **Allowed** the `from` field of `sendTransaction` and `signData` to use raw or user-friendly address format.

## June 24, 2025

**TON Connect 2025.06.24**

- **Added** `network` and `from` fields to all three `signData` payload variants (`text`, `binary`, `cell`).

## June 19, 2025

**TON Connect 2025.06.19**

- **Added** `'browser'` to the `DeviceInfo.platform` enum.

## June 3, 2025

**TON Connect 2025.06.03**

- **Added** `extraCurrencySupported` parameter to the `SendTransaction` feature.
- **Added** `types` parameter to the `SignData` feature.

## March 18, 2025

**TON Connect 2025.03.18**

- **Added** dot-separation requirement to the app domain in `tonProof`; bare names like `tonkeeper` or `tonhub` are reserved for native wallet integrations and MUST NOT be accepted from external dApps.

## March 10, 2025

**TON Connect 2025.03.10**

- **Added** `SignData` feature entry.

## March 7, 2023

**TON Connect 2023.03.07**

- **Replaced** the string-typed `Feature` entries in `DeviceInfo` with object-typed entries.
- **Added** `maxMessages` parameter to the `SendTransaction` feature.

## February 26, 2023

**TON Connect 2023.02.26**

- **Added** `network` and `from` fields to `sendTransaction`.

## February 15, 2023

**TON Connect 2023.02.15**

- **Added** `id` field to wallet events on the injected and HTTP bridges.
- **Added** `disconnect` request from dApp to wallet.
- **Removed** `disconnect` method from the injected JS bridge.

## February 6, 2023

**TON Connect 2023.02.06**

- **Replaced** per-wallet universal links with a unified universal deeplink for connect URLs.

## January 30, 2023

**TON Connect 2023.01.30**

- **Added** `signData` (formerly `signRaw`) RPC method.
- **Added** `topic` query parameter to bridge `POST /message` for push-notification routing.

## November 29, 2022

**TON Connect 2022.11.29**

- **Added** dApp manifest requirement and `manifestUrl` field to the connect request.

## November 28, 2022

**TON Connect 2022.11.28**

- **Added** `SendTransaction` feature entry to `DeviceInfo.features`.

## November 25, 2022

**TON Connect 2022.11.25**

- **Added** `maxProtocolVersion` field to `DeviceInfo`.

## November 4, 2022

**TON Connect 2022.11.04**

- **Added** `tonProof` connect item with payload challenge and signature response.

## October 20, 2022

**TON Connect 2022.10.20**

- **Added** initial protocol publication: `bridge.md`, `session.md`, `requests-responses.md`, `workflows.md`.
