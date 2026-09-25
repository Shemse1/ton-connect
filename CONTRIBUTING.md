# Contributing

This document tells you how to propose a change to the TON Connect protocol.

## Proposing a change

1. **Open an issue first.** Describe the problem, not the solution. Spec edits without a discussion thread land slowly.
2. **Wait for editor feedback.** A maintainer accepts the proposal, asks for changes or rejects it with a reason.
3. **Submit a pull request** that implements the proposal. Reference the issue.
4. **Iterate during review.** Spec changes are normative — reviewers care about every MUST, SHOULD and MAY.
5. **Open companion implementation pull requests.** A spec PR MUST be accompanied by an implementation PR in every affected reference repository, linked from the spec PR description:
   - [`ton-connect/sdk`](https://github.com/ton-connect/sdk) — for any change to normative client-facing behaviour.
   - [`ton-connect/bridge`](https://github.com/ton-connect/bridge) — for any change to the HTTP bridge serialization.
   - [`ton-connect/wallets-list`](https://github.com/ton-connect/wallets-list) — for any change to the wallets-list schema or wallet-entry shape.

   The spec PR MUST NOT merge until each required companion PR is reviewed and ready to merge alongside it.
6. **Merge.** The editor merges after at least one approving review and a clean CI run on the spec PR and every companion PR.

> **Note.** Substantial protocol changes (new RPC methods, new feature flags) SHOULD go through a written design issue before any spec edit. Drive-by spec edits are limited to clarifications, typo fixes and missing references.

## RFC-2119 conventions

The keywords MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

Write these in **UPPERCASE** when imposing a conformance requirement. They define what an implementation has to do to claim compliance.

## Versioning policy

The protocol is calendar-versioned. Each release is identified by its release date in `YYYY.MM.DD` form and referred to as TON Connect `YYYY.MM.DD` in prose. The release commit is tagged with the same string in git. If more than one release lands on the same day, append a suffix: `2026.05.07-2`.

The canonical version of the spec is the most recent dated heading in [`CHANGELOG.md`](./CHANGELOG.md).

Calendar versioning carries no implicit major / minor axis. Protocol-level compatibility is negotiated separately through the integer `DeviceInfo.maxProtocolVersion` exchanged during the connect handshake. That field keeps its narrow meaning and MUST be bumped only when the bridge serialization changes in a way that breaks older clients. Behaviour-level capabilities are advertised through `DeviceInfo.features`, and adding a feature entry does not require a `maxProtocolVersion` bump.

Breaking changes are flagged in the changelog with the verbs **Removed** or **Replaced**, and called out in the release notes accompanying the tag.

## Naming conventions

All new additions to the protocol specification MUST follow these rules:

| Element                   | Convention | Examples                                                     |
|---------------------------|------------|--------------------------------------------------------------|
| Method names              | camelCase  | `sendTransaction`, `signMessage`, `signData`                 |
| Field names               | camelCase  | `stateInit`, `forwardAmount`, `nftAddress`, `internalBoc`    |
| Type discriminator values | snake_case | `"ton"`, `"jetton"`, `"nft"`, `"text"`, `"binary"`, `"cell"` |
| Feature `name` values     | PascalCase | `"SendTransaction"`, `"SignMessage"`, `"EmbeddedRequest"`    |

### Legacy fields

Some existing fields use snake_case (for example `valid_until`, `extra_currency`). These are kept as-is for backward compatibility. New fields MUST NOT introduce snake_case names.

## File naming and structure

- File names are kebab-case: `embedded-requests.md`, `sign-message.md`.
- One topic per file. If a page grows past ~600 lines, split it.
- Headings are sentence case — `Sending a transaction`, not `Sending A Transaction`.
- Headings prefer gerunds (`Sending …`, `Building …`) or noun phrases (`Code types`, `Wallet events`).

## Code blocks and identifiers

- All code blocks MUST be language-tagged: ` ```ts `, ` ```json `, ` ```http `, ` ```bash `, ` ```mermaid `.
- Identifiers — method names, field names, file paths, error codes — go in `code font`.
- Hex values: write them without `0x` unless the spec convention is otherwise. Bit and byte widths are hyphenated: 64-bit, 2048-bit.
- Cross-references are inline, never footnoted. Link text is the noun being referenced, not "click here".

## Voice

Precise, neutral, RFC-2119, no first-person plural in prescriptive prose, no contractions.

## See also

- [`README.md`](./README.md)
- [`CHANGELOG.md`](./CHANGELOG.md)
- [`GLOSSARY.md`](./GLOSSARY.md)
