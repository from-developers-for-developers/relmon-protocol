# Reliable Monetary data protocol (RelMon protocol)

## Introduction

This document specifies the **RelMon protocol** (Reliable Monetary Data Protocol, further referred as **RelMon**) and defines its **concrete default data format implementations**.

RelMon is designed to enable reliable exchange of monetary data between systems. By standardizing the structure and semantics of monetary data, it ensures interoperable implementations across diverse systems, preserving **exact monetary values**, full precision, consistent rounding, and a **calculation-free approach** for net, tax, and gross amounts.

RelMon is programming language, storage, and data format agnostic, defining only the **abstract logical model** and **the concrete default data format implementations**.

## Key Words

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

These terms define **normative requirements** for implementers and clarify which behavior is mandatory, recommended, or optional.

## Versioning 

RelMon specification uses [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR** version changes indicate incompatible modifications to the abstract protocol model.
- **MINOR** version changes indicate backward-compatible extensions, such as new optional fields or formats.
- **PATCH** version changes indicate non-breaking corrections, clarifications, or editorial updates.

Protocol identifiers include the version, for example:

- `relmon@1.0.0` – abstract protocol version 1.0.0

Implementations **MUST respect protocol versioning** to ensure compatibility between systems.

## Abstract Model

### Notation

The specification uses an **informal abstract data model notation** to describe the logical structure of RelMon monetary values.

This notation is:

- **Descriptive only**
- **Non-executable**
- **Not intended to be parsed or used as a schema language**

Concrete default data format implementations of the protocol are defined separately.

**Legend**

- `?` indicates an optional field  
- `[]` indicates an ordered list  
- Capitalized identifiers denote abstract types  

---

```
RelMonObject :=
{
    protocol: ProtocolIdentifier
    net: Decimal | Integer
    tax: Decimal | Integer
    gross: Decimal | Integer

    taxRate?: Decimal(6, 3)
    unit?: Text
    precision?: DecimalPrecision
    rounding?: RoundingMode
    components?: MonetaryComponent[]
}

ProtocolIdentifier := "relmon@MAJOR.MINOR.PATCH[:mode1[.mode2[.mode3...]]]"
DecimalPrecision := [maxDigits: Integer, scale: Integer]
RoundingMode := "hup" | "hdown" | "heven" | "up" | "down"
MonetaryComponent :=
{
    net: Decimal | Integer
    tax: Decimal | Integer
    taxRate?: Decimal(6, 3)
    comment?: Text
}
```

### Specification clarification

- **Protocol version**: `protocol` MUST follow the defined `ProtocolIdentifier` format and reflect the SemVer version of the abstract model.
- **Protocol modes**: `protocol` MAY define its modes affecting the format and the fields of the protocol. Available modes are: `e` (extended), `c` (compact), `m` (minors).
- **Monetary values**: `net`, `tax`, and `gross` are exact and MUST preserve full precision. 
    - Implementations MUST NOT perform floating-point arithmetic. 
    - These fields MUST be represented as decimals unless the `m` mode is used.
- **Consistency rules**: `gross = net + tax`.
- **Units**: When the unit represents a fiat currency, implementations SHOULD use the corresponding ISO 4217 currency code in the `unit` field. Examples: `EUR`, `USD`, `RUB`.
- **Precision**: The optional `precision` field defines `[maxDigits, scale]`, where `maxDigits` defines the total amount of digits in the value, and `scale` the amount of digits after the point. All decimal values in the fields `net`, `tax`, and `gross` MUST comply if present.
    - In case of using the mode `m`, the implementor MAY use the `precision` field to properly format monetary values in the representational layer.
- **Rounding**: The optional `rounding` field specifies the method to apply in cases of arithmetic operations.
    - The `rounding` field MUST be present if `precision` is set AND rounding has been applied.
    - The `rounding` field MUST NOT be present if `m` mode is used.
- **Optional metadata**: `comment` in `MonetaryComponent` is purely informational and MUST NOT affect numeric calculations. This field is ALWAYS optional.
- **Format agnostic**: The abstract model does not prescribe JSON, XML, URI, or binary layouts. Concrete data format implementations map abstract fields to specific formats, as defined in separate sections.

### Protocol modes

RelMon defines 3 modes that modify the format and the set of the fields of the protocol. The modes MUST NOT change the logical model of the protocol. The order of mode declaration MUST NOT affect the RelMon logical model.

- **Extended**: Extended mode (`e`) indicates that `RelMonObject` MUST contain **all** fields defined in the abstract model, including those normally optional. The **only exception** is the `components` field, which MAY be omitted or be empty.
- **Compact**: Compact mode (`c`) defines that the names of the fields of the `RelMonObject` are written in the compact mode. Compact mode affects field identifiers only and MUST NOT alter semantic meaning or numeric behavior. This mode MUST be identified as `c`.
    - `protocol` is represented via `pr`
    - `net` is represented via `n`
    - `tax` is represented via `t`
    - `gross` is represented via `g`
    - `taxRate` is represented via `tr`
    - `unit` is represented via `u`
    - `precision` is represented via `p`
    - `rounding` is represented via `r`
    - `components` is represented via `cs`
- **Minors**: Minors mode (`m`) defines that the fields `net`, `tax` and `gross` of `RelMonObject` are the smallest units of the currency or asset and MUST be represented as integers. This mode MUST be identified as `m`.

An example: `relmon@1.0.0:e.c.m`.

### Rounding modes

RelMon supports the following rounding modes:

- `hup` value stands for "half-up" and acts as rounding away from zero. Examples: 
    - `1.5 => 2`
    - `2.5 => 3`
    - `-1.5 => -2`
- `hdown` value stands for "half-down" and acts as rounding towards zero. Examples: 
    - `1.5 => 1`
    - `2.5 => 2`
    - `-1.5 => -1`
- `heven` value stands for "half-even" and acts as rounding to the nearest even digit. Examples: 
    - `1.5 => 2`
    - `2.5 => 2`
    - `3.5 => 4`
    - `4.5 => 4`
    - `-1.5 => -2`
    - `-2.5 => -2`
- `up` value stands for "up" and acts as rounding away from zero, discarding any fractional part. Examples: 
    - `1.1 => 2`
    - `-1.1 => -2`
- `down` value stands for "down" and acts as rounding towards zero, discarding any fractional part. Examples: 
    - `1.9 => 1`
    - `-1.9 => -1`.

### Validation rules

RelMon defines the following rules for validating monetary values:

- **Consistency**
    - `gross = net + tax`
    - If `components` are present, the sum of all component `net` values MUST equal the parent `net`, and the sum of all component `tax` values MUST equal the parent `tax`.
- **Data types**
    - `net`, `tax`, and `gross` MUST be decimals unless **Minors mode** (`m`) is active, in which case they MUST be integers.
- **Precision**
    - If `precision` is specified, all decimal values in `net`, `tax`, and `gross` MUST comply with `[maxDigits, scale]`.
- **Rounding**
    - If rounding has been applied, the `rounding` field MUST be present and its value MUST be one of the allowed rounding modes.
    - If no rounding has occurred, the `rounding` field MAY be omitted.
    - Allowed values: `hup`, `hdown`, `heven`, `up`, `down`.
- **Optional fields**
    - Any optional fields, such as `unit` or `taxRate`, if present, MUST conform to their abstract type definitions.
- **Components**
    - Each `MonetaryComponent` MUST include `net` and `tax`.
    - The optional `taxRate` in a component MUST comply with the format defined in the abstract model.
    - The optional `comment` in a component MUST NOT affect numeric calculations.
    
### Signed / Negative values semantics

Monetary values in RelMon may be negative. Implementations MUST handle negative values as follows:

- Fields `net`, `tax` and `gross` MAY be negative (prefixed with `-` symbol) for both decimals or minors.
- Positive values MUST NOT have any prefix (no `+` prefix).
- All root-level fields (`net`, `tax` and `gross`) MUST have the same sign.
- Components MAY independently be positive or negative, but each component's `net` and `tax` MUST share the same sign.
- Rounding modes specified in the `rounding` field apply symmetrically to negative values.

Implementations MUST ensure that arithmetic consistency rules (`gross = net + tax` and component sums) remain valid even when values are negative.

### Conformance Profiles

RelMon defines conformance profiles to help implementers achieve predictable interoperability:

- **Core Profile**
    - Includes all **required fields**: `protocol`, `net`, `tax`, `gross`.
    - Optional fields (`taxRate`, `unit`, `precision`, `rounding`, `components`, `comment`) MAY be omitted.
    - Ensures basic interoperability across systems.
- **Extended Profile**
    - Includes **all fields of the abstract model**, including optional ones (except `components`, which MAY be empty or omitted).
    - Provides **full precision**, **reconstructibility**, and **semantic clarity**.
- **Compact Profile**
    - Mirrors the **Core** or **Extended** profile, but uses **compact field names** as defined in Compact mode (`c`).

Implementers MAY declare which profile their system conforms to, ensuring that exchanging systems can verify compatibility and expectations before transfer.

## Concrete default data format implementations

RelMon defines multiple **concrete default data format implementations** to represent the abstract monetary model. This section provides a high-level overview and references for each format.

Default format implementation as of this version of the protocol are: JSON, XML, and multiple URI-based notations - JSON, XML, minimalistic notation and [Protocol Buffers](https://protobuf.dev/) notation.

RelMon is open to be implemented in any other data format.

The **abstract model** remains unchanged across formats; these implementations define **how the abstract fields map to JSON, XML, or URI-based**.

### JSON

The example here includes the **extended** mode of RelMon object, including `components` fields.

All modes of the protocol are supported.

The name of the element containing RelMon object MAY be arbitrary.

```json
{
  "protocol": "relmon@1.0.0:e",
  "net": "100.00",
  "tax": "21.00",
  "gross": "121.00",
  "taxRate": "0.210",
  "unit": "EUR",
  "precision": [5, 2],
  "rounding": "hup",
  "components": [
    {
      "net": "50.00",
      "tax": "10.50",
      "taxRate": "0.210",
      "comment": "Base price of item 1"
    },
    {
      "net": "50.00",
      "tax": "10.50",
      "taxRate": "0.210",
      "comment": "Base price of item 2"
    }
  ]
}
```

### XML

The example here includes the **extended** mode of RelMon object, excluding `components` fields.

All modes of the protocol are supported.

The root name of the element MAY be arbitrary (e.g. `<Money>`, `<Price>`, etc.).

```xml
<RelMon>
  <protocol>relmon@1.0.0:e</protocol>
  <net>100.00</net>
  <tax>21.00</tax>
  <gross>121.00</gross>
  <taxRate>0.210</taxRate>
  <unit>EUR</unit>
  <precision maxDigits="5" scale="2"/>
  <rounding>hup</rounding>
</RelMon>
```

### URI-based notations

Compact, single-string representations suitable for **URLs**, **QR codes**, or **lightweight transfers**. 

All modes are supported in all URI-based notations.

The name of the field containing the URI-based notation of RelMon MAY be arbitrary.

| Scheme | Description | Example |
| -------- | ------- |  ------- | 
| `relmon/json://...` | JSON representation (as base64) encoded in URI | `relmon/json://eyJwciI6InJlbG1vbkAxLjAuMDpjIiwibmV0IjoiMTAwLjAwIiwiZ3Jvc3MiOiIxMjEuMDAiLCJ0YXgiOiIyMS4wMCJ9` |
| `relmon/xml://...` | XML representation (as base64) encoded in URI | `relmon/xml://PFJlbE1vbiBwcm90b2NvbD0icmVsbW9uQDEuMC4wOmMubSI+PG4+MTAwMDA8L24+PGc+MTIxMDA8L2c+PHQ+MjEwMDwvdD48L1JlbE1vbj4=` |
| `relmon/min://...` | Minimalistic textual representation containing only the protocol version, net, gross and tax values separated by `;` | `relmon/min://1.0.0:m;10000;12100;2100` |
| `relmon/protobuf://...` | Protobuf repsentation (as base64) encoded in URI | `TODO` |

## FAQ and design rationale

This section is **informative only** and does not contain requirements. It explains key design decisions of RelMon to help implementers understand its rationale.

### **Why separate `net`, `tax`, and `gross`?**

To make the protocol *calculation-free*. Systems do not need to compute `tax` or `gross` from `net`; each value is explicitly specified. This eliminates differences caused by rounding or arithmetic in heterogeneous systems.

### Why is `taxRate` a decimal(6,3)?

To support tax rates with **fractional precision**, e.g., 12.35% or 0.625%. The (6,3) format allows up to 999.999% while keeping three decimal places, which covers nearly all real-world tax regimes.

### Why provide a `precision` field?

Precision defines the **maximum total digits and fractional scale** for monetary values. This ensures that values are consistently formatted and interpreted across systems. It is optional, but required for extended mode.

### Why include a `rounding` field?

Rounding modes specify how **intermediate or formatted values** are handled when reduction of scale occurs. Specifying the mode ensures consistent rounding behavior across implementations.

Included rounding modes are all **finance-relevant**: `half-up`, `half-down`, `half-even`, `up`, `down`.

### Why have Extended, Compact, and Minors modes?

- **Extended** (`e`): All fields included, full precision and reconstructibility.
- **Compact** (`c`): Shortened field names to save bandwidth or storage.
- **Minors** (`m`): Integer representation for systems that operate in the smallest unit (e.g., cents, satoshis).

These modes give flexibility while maintaining **semantic consistency**.

### Can RelMon handle cryptocurrencies like BTC or ETH?

Yes. While ISO 4217 codes are suggested for fiat currencies, **custom currencies** are supported via `unit` field. **Minors** mode can store satoshis, wei, or other smallest units as integers.

### Why allow negative values?

Negative values represent refunds, reversals, credits, or discounts. Consistent rules ensure that **arithmetic invariants** (`gross = net + tax`) and component sums remain valid, even when values are negative.

### Why is the protocol language, storage, and data format agnostic?

To allow **maximum interoperability**. The abstract model defines **semantic meaning**, while concrete data format implementations (JSON, XML, URI) map these fields into specific representations. This ensures consistency across systems, programming languages, and storage backends, and eventually allows to scale and expand on wide ranges of computing systems.