# RelMon URI implementation

This document defines the official URI-based implementations of the RelMon abstract model.

If this document conflicts with [README.md](README.md), the core specification in `README.md` governs protocol semantics.

## Scope

URI-based notations are compact, single-string representations suitable for URLs, QR codes, or lightweight transfers.

- All modes are supported in URI-based notations, except where explicitly noted.
- The name of the field containing a URI-based RelMon value MAY be arbitrary.

## Schemes

| Scheme | Description | Format | Example |
|--------|-------------|--------|---------|
| `relmon-json://...` | JSON representation encoded in base64url as defined in RFC 4648 Section 5 | `relmon-json://base64EncodedJson` | `relmon-json://eyJwIjogInJlbG1vbkAxLjAuMC8xOmMiLCAibiI6ICIxMDAuMDAiLCAidHIiOiAiMjEuMDAifQ==` |
| `relmon-xml://...` | XML representation encoded in base64url as defined in RFC 4648 Section 5 | `relmon-xml://base64EncodedXml` | `relmon-xml://PFJlbE1vbj4KICAgIDxwPnJlbG1vbkAxLjAuMC8zOmMubTwvcD4KICAgIDxuPjEwMDAwPC9uPgogICAgPGc-MTIxMDA8L2c-CiAgICA8dD4yMTAwPC90PgogICAgPHU-RVVSPC91Pgo8L1JlbE1vbj4=` |
| `relmon-min://...` | Minimalistic textual representation with DL3 containing only the protocol identifier, net, gross, and tax values separated by `;`.<br/><br/>**Note**:<br/>- In this notation the `ProtocolIdentifier` is shortened by removing the `relmon@` prefix.<br/>- This notation supports minors via `m` mode.<br/>- Compact mode `c` is not supported in this notation. | `relmon-min://<ProtocolIdentifier>;<net>;<gross>;<tax>` | `relmon-min://1.0.0/3:m;10000;12100;2100` |
