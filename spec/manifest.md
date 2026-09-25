# App manifest

This document is normative for dApps, wallets, bridge operators, SDKs. The keywords MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

The app manifest is a JSON file the wallet fetches before showing the connect prompt. It carries the metadata the wallet displays to the user — the app's name, icon, terms of use and privacy policy. The dApp passes its URL to the wallet as [`ConnectRequest.manifestUrl`](./connect.md#connectrequest).

## File and URL

The manifest MUST be a valid JSON document encoded in UTF-8. The conventional file name is `tonconnect-manifest.json`, hosted at the root of the dApp's domain (for example, `https://example.com/tonconnect-manifest.json`). The dApp MAY host the file at any HTTPS URL the wallet can reach. The URL the dApp advertises in [`ConnectRequest.manifestUrl`](./connect.md#connectrequest) is authoritative.

The JSON Schema for the manifest is published at [`schemas/manifest.schema.json`](../schemas/manifest.schema.json).

## Fields

```jsonc
{
  "url": "<app-url>",                        // required
  "name": "<app-name>",                      // required
  "iconUrl": "<app-icon-url>",               // required
  "termsOfUseUrl": "<terms-of-use-url>",     // optional
  "privacyPolicyUrl": "<privacy-policy-url>" // optional
}
```

| Field              | Required | Type   | Description                                                                                                                                                          |
|--------------------|----------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `url`              | yes      | string | App URL. Identifies the dApp and is opened by the wallet when the user taps the app icon in the wallet's connected-apps list. SHOULD NOT include a trailing slash. The host portion is extracted as the `AppDomain` for [`ton_proof`](./connect.md#address-proof-signature-ton_proof) and `signData` signatures, so it MUST satisfy the [domain-binding rules](./connect.md#domain-binding-rules). |
| `name`             | yes      | string | Display name of the dApp, shown to the user during connect and in the wallet's connected-apps list. Not used as an identifier.                                       |
| `iconUrl`          | yes      | string | URL of the app icon. See [Icon](#icon).                                                                                                                              |
| `termsOfUseUrl`    | no       | string | URL of the dApp's terms of use.                                                                                                                                      |
| `privacyPolicyUrl` | no       | string | URL of the dApp's privacy policy.                                                                                                                                    |

The format is forward-compatible by reservation. See [Versioning](#versioning).

## Hosting

The manifest MUST be publicly accessible from the internet. It is fetched by every wallet that handles the dApp's connect request, including wallets running on the user's device with no relation to the dApp. The same applies to `iconUrl`, `termsOfUseUrl` and `privacyPolicyUrl`.

The manifest MUST be retrievable with an unauthenticated HTTP `GET`. The response body MUST parse as a JSON object matching the [schema](#fields).

The manifest SHOULD be reachable from any origin without CORS restrictions. Wallets fetch the file from a different origin than the dApp. A restrictive `Access-Control-Allow-Origin` header prevents the wallet from reading the response, and the wallet replies to the dApp with a [`ConnectEventError`](./connect.md#connectevent) carrying [error code `2`](./connect.md#connect-event-error-codes) (App manifest not found) or `3` (App manifest content error).

The response SHOULD carry `Content-Type: application/json`. Wallets MAY accept other text content types they can decode as JSON.

## Caching

Wallets MAY cache fetched manifests. When the response carries standard HTTP cache headers as defined in [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111) — `Cache-Control`, `Expires`, `ETag`, `Last-Modified` — wallets SHOULD honour them. Wallets SHOULD revalidate stale entries with conditional requests (`If-None-Match`, `If-Modified-Since`) rather than refetching unconditionally.

When the response carries no caching headers, wallets SHOULD treat the manifest as cacheable for an implementation-defined period, not exceeding 24 hours.

dApps SHOULD set explicit cache headers. A reasonable default:

```http
Cache-Control: public, max-age=3600
ETag: "<hex of sha256(body)>"
```

No cache-busting parameter is defined at the protocol level. To force every wallet to refetch the manifest, change the value of [`ConnectRequest.manifestUrl`](./connect.md#connectrequest) — for example, by appending a build hash query string (`?v=<sha>`) or by serving the file under a versioned path (`tonconnect-manifest-v2.json`). The same technique applies to a stale `iconUrl`.

## Icon

`iconUrl` MUST point to a raster image served over HTTPS. PNG and ICO are the interoperable formats. PNG is recommended. SVG MUST NOT be used.

| Property   | Constraint                                                                                              |
|------------|---------------------------------------------------------------------------------------------------------|
| Format     | PNG (recommended) or ICO. SVG is forbidden. Other raster formats (JPEG, WebP) MAY be accepted, but interoperability is not guaranteed. |
| Dimensions | Square. 180×180 px is recommended.                                                                      |
| Hosting    | Same as the manifest itself: publicly reachable, served without authentication or origin restrictions.  |

A wallet that cannot fetch or decode the icon MAY substitute a placeholder; it SHOULD NOT block the connect flow on icon failure alone.

## Versioning

The manifest format is unversioned in this revision of the specification. The top-level `version` field is reserved.

- dApps MUST NOT include a `version` field. Its semantics are reserved for a future revision of this specification; serving a manifest with a `version` field today causes undefined wallet behaviour.
- Wallets MUST ignore additional unknown top-level fields. The format extends forward-compatibly through new optional fields without a wallet upgrade.

A future incompatible change will be introduced either by activating the reserved `version` field or by a new conventional file name (for example, `tonconnect-manifest-v2.json`). Both options remain reserved.

## Example

```json
{
  "url": "https://example.com",
  "name": "Example App",
  "iconUrl": "https://example.com/icon-180.png",
  "termsOfUseUrl": "https://example.com/terms",
  "privacyPolicyUrl": "https://example.com/privacy"
}
```

## See also

- [`schemas/manifest.schema.json`](../schemas/manifest.schema.json) — JSON Schema
- [`connect.md` § `ConnectRequest`](./connect.md#connectrequest) — `manifestUrl`
- [`connect.md` § Address proof signature](./connect.md#address-proof-signature-ton_proof) — domain binding
