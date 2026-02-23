# Reliable Monetary data protocol (RelMon protocol)

## Table of contents

* [Reliable Monetary data protocol (RelMon protocol)](#reliable-monetary-data-protocol-relmon-protocol)
  * [Table of contents](#table-of-contents)
  * [Introduction](#introduction)
  * [Key words](#key-words)
  * [Versioning](#versioning-)
  * [License](#license)
  * [Abstract logical model](#abstract-logical-model)
    * [Notation](#notation)
    * [The model](#the-model)
  * [Protocol specification](#protocol-specification)
    * [Protocol identifier](#protocol-identifier)
      * [Protocol modes](#protocol-modes)
    * [Monetary values](#monetary-values)
    * [Signed values semantics](#signed-values-semantics)
    * [Monetary bases](#monetary-bases)
    * [Determinism levels](#determinism-levels)
    * [Calculation scope](#calculation-scope)
    * [Arithmetics](#arithmetics)
      * [Precision](#precision)
      * [Rounding modes](#rounding-modes)
      * [Rounding application](#rounding-application)
      * [Rounding function](#rounding-function)
    * [Validation rules](#validation-rules)
    * [Summary table of the model fields](#summary-table-of-the-model-fields)
  * [Concrete default data format implementations](#concrete-default-data-format-implementations)
    * [JSON](#json)
    * [XML](#xml)
    * [URI-based notations](#uri-based-notations)
  * [FAQ and design rationale](#faq-and-design-rationale)
    * [Why different determinism levels?](#why-different-determinism-levels)
    * [Why not allow arbitrary precision for net, tax, and gross?](#why-not-allow-arbitrary-precision-for-net-tax-and-gross)
    * [Why is `taxRate` a decimal(6,3)?](#why-is-taxrate-a-decimal63)
    * [Why provide a `precision` field?](#why-provide-a-precision-field)
    * [Why include a rounding mode and application field?](#why-include-a-rounding-mode-and-application-field)
    * [Why is the default rounding mode - "heven", and the default rounding application - "tax"?](#why-is-the-default-rounding-mode---heven-and-the-default-rounding-application---tax)
    * [Why have Compact and Minors modes?](#why-have-compact-and-minors-modes)
    * [Can RelMon handle cryptocurrencies like BTC or ETH?](#can-relmon-handle-cryptocurrencies-like-btc-or-eth)
    * [Why allow negative values?](#why-allow-negative-values)
    * [Why is the protocol language, storage, and data format agnostic?](#why-is-the-protocol-language-storage-and-data-format-agnostic)
  * [Changelog](#changelog)
  * [TODOs](#todos)

## Introduction

This document specifies the **RelMon protocol** (Reliable Monetary Data Protocol, hereinafter referred to as **RelMon**) and defines its **concrete default data format implementations**.

RelMon ensures deterministic reconstruction of monetary components across systems. By standardizing the structure and semantics of monetary data, it ensures interoperable implementations across diverse systems, preserving **exact monetary values**, full precision, consistent rounding, and, following the maximum determinism level, a **derivation-free approach** for net, tax, and gross amounts.

RelMon is programming language-, storage-, and data format-agnostic, defining only the **abstract logical model** and **the concrete default data format implementations**.

An example of a RelMon object in JSON format (determinism level 3):

```json
{"protocol": "relmon@1.0.0/3", "net": "100.00", "tax": "21.00", "gross": "121.00"}
```

## Key words

The key words **MUST**, **MUST NOT**, **REQUIRED**, **shall**, **shall NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

These terms define **normative requirements** for implementers and clarify which behavior is mandatory, recommended, or optional.

## Versioning 

The RelMon specification uses [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR** version changes indicate incompatible modifications to the abstract protocol model.
- **MINOR** version changes indicate backward-compatible extensions, such as new OPTIONAL fields or formats.
- **PATCH** version changes indicate non-breaking corrections, clarifications, or editorial updates.

Protocol identifiers include the version, for example:

- `relmon@1.0.0` – abstract protocol version 1.0.0

Implementations **MUST respect protocol versioning** to ensure compatibility between systems.

## License

The RelMon specification is published as an **open and freely usable standard**.

This specification is licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license.

This means that:

- Anyone **MAY use, implement, and distribute** the RelMon specification
- Implementations **MAY be open-source or proprietary**
- The specification **MAY be modified or extended**
- Proper **attribution to the RelMon project** is REQUIRED

The license applies **only to the specification text itself**.

It does **not impose any licensing requirements** on implementations, SDKs, or software that uses the RelMon protocol.

For the full license text, see [this file](LICENSE).

## Abstract logical model

### Notation

The specification uses an **informal abstract data model notation** to describe the logical structure of RelMon monetary values.

This notation is:

- **Descriptive only**
- **Non-executable**
- **Not intended to be parsed or used as a schema language**

Concrete default data format implementations of the protocol are defined separately.

**Legend**

- `?` indicates an OPTIONAL field
- `:` indicates a type (`object`, `list`, `decimal`, `integer`, `text`)
- `=` indicates a value
- `<xxx>` indicates a field interpolated in the textual type, where `xxx` is the field name
- `|` - a separator for enum values
- `[?` indicates the OPTIONAL part
- `...` indicates that the construction can be repeated multiple times
- `[]` indicates an ordered list
- `decimal(p, s)` indicates a decimal number with at most `p` total digits and at most `s` fractional digits
- `#` indicates a clarifying comment
- Capitalized identifiers denote abstract types

### The model

```
RelMonObject: object =
{
    protocol:    ProtocolIdentifier

    # either net or gross is required
    net?:        decimal | integer
    gross?:      decimal | integer
    
    # either tax or taxRate is required
    tax?:        decimal | integer 
    taxRate?:    decimal(6, 3)

    unit?:       text
    precision?:  integer
    scope?:      CalculationScope
    rounding?:   Rounding
    components?: MonetaryComponent[]
}

ProtocolIdentifier: text = "relmon@<MAJOR>.<MINOR>.<PATCH>/<DeterminismLevel>[?:<MODE>[?.<MODE>...]]"
MAJOR: integer
MINOR: integer
PATCH: integer
MODE?: text
DeterminismLevel: integer = 1 | 2 | 3

CalculationScope: text = "r" | "c"

Rounding: object = {
    mode?:        text = "haway" | "hzero" | "heven" | "up" | "down"
    application?: text = "tax" | "total"
}

MonetaryComponent: object =
{
    # either net or gross is required
    net?:     decimal | integer
    gross?:   decimal | integer
    
    # either tax or taxRate is required
    tax?:     decimal | integer
    taxRate?: decimal(6, 3)
    
    comment?: text
}
```

An example of RelMonObject using the same logical notation:

```
RelMonObject = {
    protocol = "relmon@1.0.0/3:m"
    net = 20000
    gross = 24200
    tax = 4200
    taxRate = 21.00
    unit = "EUR"
    precision = 2
    scope = "c"

    rounding = { 
        mode = "heven"
        application = "tax"
    }

    components = [
        {
            net = 10000
            gross = 12100
            tax = 2100
            taxRate = 21.00
            comment = "Test component for demonstrating the RelMonObject"
        },
        
        {
            net = 10000
            gross = 12100
            tax = 2100
            taxRate = 21.00
            comment = "Another component"
        }
    ]
}
```

## Protocol specification

- The abstract model does not prescribe JSON, XML, URI, or binary layouts. 
- Concrete data format implementations map abstract fields to specific formats, as defined in separate sections.
- All decimal values MUST use `.` symbol as a decimal separator.

### Protocol identifier

- **Protocol version**: `protocol` MUST follow the defined `ProtocolIdentifier` format and reflect the SemVer version of the abstract model.
- **Protocol determinism level**: protocol identifier MUST include the determinism level of the protocol. 
- **Protocol modes**: `protocol` MAY define its modes affecting the format and fields of the protocol. Available modes are: `c` (compact), `m` (minors).

#### Protocol modes

RelMon defines modes that modify the format and field set of the protocol. 
The modes MUST NOT change the logical model of the protocol. 
The order of mode declaration MUST NOT affect the RelMon logical model.

- **Compact**: Compact mode (`c`) specifies that field names of `RelMonObject` are abbreviated. Compact mode affects field identifiers only and MUST NOT alter semantic meaning or numeric behavior. This mode MUST be identified as `c`.
  - `protocol` is represented via `p`
  - `net` is represented via `n`
  - `gross` is represented via `g`
  - `tax` is represented via `t`
  - `taxRate` is represented via `tr`
  - `unit` is represented via `u`
  - `precision` is represented via `pr`
  - `scope` is represented via `s`
  - `rounding{mode, application}` is represented via `r{m, a}`
  - `components` is represented via `cs`
    - `comment` is represented via `c`
    - `net`, `gross`, `tax`, `taxRate` are represented the same way as on the root-level (`n`, `g`, `t`, `tr`) 
- **Minors**: Minors mode (`m`) specifies that the `net`, `gross`, and `tax` fields of `RelMonObject` represent the smallest units of the currency or asset and MUST be represented as integers. This mode MUST be identified as `m`.

An example: `relmon@1.0.0/3:c.m`.

### Monetary values

- The `net`, `gross`, and `tax` fields represent canonical monetary values.
- They MUST be represented as decimals unless the `m` mode (minors) is used.
- If precision is specified, these values MUST conform to it and MUST be treated as exact values by implementations.
- All provided monetary values (`net`, `tax`, `gross`) both on the root-level and on the component-level MUST use the same number of decimal places.
- When the `unit` represents a fiat currency, it is RECOMMENDED for implementations to use the corresponding ISO 4217 currency code in the `unit` field. Examples: `EUR`, `USD`, `RUB`.

### Signed values semantics

Monetary values in RelMon MAY be negative. Implementations MUST handle negative values as follows:

- The `net`, `gross`, and `tax` fields MAY be negative (prefixed with `-` symbol) in both decimal and minors representations.
- Positive values MUST NOT have any prefix (no `+` prefix).
- All root-level fields (`net`, `gross`, and `tax`) MUST have the same sign.
- Components MAY independently be positive or negative, but each component's `net`, `gross`, and `tax` MUST share the same sign.
- Rounding modes specified in the `rounding` field apply symmetrically to negative values.

Implementations MUST ensure that arithmetic consistency rules (`gross = net + tax` and component sums) remain valid even when values are negative.

### Monetary bases

`RelMonObject` MUST contain one of the following valid monetary bases:

- Basis 1-net - **Rate-based derivation**: `net AND taxRate`
- Basis 1-gross - **Rate-based derivation**: `gross AND taxRate`
- Basis 2 - **Tax-amount provided**: `net AND tax` and/or `gross AND tax`

### Determinism levels

RelMon defines several determinism levels. A determinism level defines how many monetary values MUST be derived from other fields.

Each level specifies a set of fields that determines how much arithmetic is REQUIRED to derive missing values.
Lower levels require more calculations, while higher levels require fewer, with the highest level requiring no derivation at all.

Determinism levels are hereinafter referred to as "DL".

| Determinism level | Description                                                                                                                                                                                                                                                                                                          | Monetary basis                        | Requirements                                                                                                                     |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| DL1               | Minimum fields allowing full reconstruction.<br/>Only `net` or `gross` value is sent.<br/>The missing amount MUST be derived.                                                                                                                                                                                        | **Basis 1-net** OR **Basis 1-gross**  | - Deterministic reconstruction REQUIRED.<br/>- If `tax` is specified, it MUST match to the derived result from `net` or `gross`. |
| DL2               | Derived values partially materialized to ensure consistency.<br/>A derived value is included even though it can be recomputed.<br/><br/>Purpose:<br/><br/>- validation across systems<br/>- detect drift or calculation differences<br/>- improves interoperability<br/>- `gross` and `net` are authoritative values | **Basis 1-net** AND **Basis 1-gross** | - Materialized values MUST match recomputed values                                                                               |
| DL3               | Monetary values explicitly provided.<br/>No derivation needed or precision-loss safe derivation available.                                                                                                                                                                                                           | **Basis 2**                           | `gross = net + tax` and `net = gross - tax`                                                                                      |

### Calculation scope

One of the following calculation scopes MUST be used:

- `c` - calculation scope is **component-based**, meaning that all derivations are performed on a per-component basis;
  - including `components` field is REQUIRED in this case.
- `r` - calculation scope is **root-based**, meaning that all derivations are performed on the root-level `net` or `gross` field.

Regardless of the calculation scope, the root-level `net`, `gross`, and `tax` values MUST be respectively equal to the sum of all per-component `net`, `gross`, and `tax` fields.

An example:

```
net = 100.00
gross = 121.00
tax = 21.00
components = [
    { net: 50.00, gross: 60.50, tax: 10.50 },
    { net: 50.00, gross: 60.50, tax: 10.50 }
]
```

Summing the `net`, `gross`, and `tax` values of each component yields the root-level `net`, `gross`, and `tax` values.

Implementations MUST use the **default** calculation scope `r`, unless:

- an explicit calculation scope is specified;
- or a mutual agreement for the default value is set among involved systems.

### Arithmetics

- Implementations MUST NOT perform floating-point arithmetic, unless such arithmetic guarantees identical results.
- Implementations MUST produce results identical to exact decimal arithmetic under the declared:
  - precision;
  - rounding mode;
  - rounding application;
  - calculation scope.
- RECOMMENDED arithmetic strategies include:
  - decimal arithmetic; 
  - integer minor-unit arithmetic; 
  - fixed-scale decimals.

#### Precision

Precision defines the scale at which rounding occurs.
For example, `precision = 2` means rounding to two decimal places: `10.336 --> 10.34`.

- `precision` defines the maximum number of decimal places after the decimal point.
  - Example `precision = 3` --> all `100.000`, `100.00`, `100.0` are valid
- If `precision` is not specified, the effective precision MUST be inferred from the monetary values (`net`, `tax`, `gross`).
  ```
  if RelMonObject.precision is specified
    precision = RelMonObject.precision
  else
    precision = inferred from RelMonObject.net | RelMonObject.tax | RelMonObject.gross
  ```
- The number of decimal places used by monetary values MUST NOT exceed the declared or inferred precision.
- In **minors** mode (`m`) `precision` is not REQUIRED, but it MAY be specified for representational purposes.

#### Rounding modes

RelMon supports the following rounding modes:

| Rounding mode | Description                                            | Examples                                                                                              |
|---------------|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| `haway`       | Half away from zero. Rounds away from zero.            | - `1.5 => 2`<br/>- `2.5 => 3`<br/>- `-1.5 => -2`                                                      |
| `hzero`       | Half towards zero. Rounds towards zero.                | - `1.5 => 1`<br/>- `2.5 => 2`<br/>- `-1.5 => -1`                                                      |
| `heven`       | Half even. Rounds to the nearest even digit.           | - `1.5 => 2`<br/>- `2.5 => 2`<br/>- `3.5 => 4`<br/>- `4.5 => 4`<br/>- `-1.5 => -2`<br/>- `-2.5 => -2` |
| `up`          | Rounds away from zero, discarding any fractional part. | - `1.1 => 2`<br/>- `-1.1 => -2`                                                                       |
| `down`        | Rounds towards zero, discarding any fractional part.   | - `1.9 => 1`<br/>- `-1.9 => -1`                                                                       |

Implementations MUST use the **default** rounding mode `heven`, unless:

- an explicit rounding mode is specified;
- or a mutual agreement for the default value is set among involved systems.

#### Rounding application

Rounding MAY be applied to:

- `tax` - the rounding function is applied to `tax` after derivation;
- `total` - the rounding function is applied to `gross` or `net` after derivation.

Implementations MUST use the **default** rounding application `tax`, unless:

- an explicit rounding application is specified;
- or a mutual agreement for the default value is set among involved systems.

#### Rounding function

RelMon defines a deterministic rounding function: `round(value, precision, roundingMode = "heven")`, where:

- `value` is a decimal number to be rounded.
- `precision` defines the number of decimal places after the decimal point.
- `roundingMode` defines how ties and directional rounding are handled (default value `heven`).

The function returns a decimal number rounded to the specified precision.

Rounding MUST be applied in accordance with `roundingApplication`.

- If `roundingApplication = "tax"`, the derivation sequence is:
  - If `net` is provided:
    ```
    tax = round(net * (taxRate / 100))
    gross = net + tax
    ```
  - If `gross` is provided:
    ```
    tax = round(gross * taxRate / (100 + taxRate))
    net = gross - tax
    ```
- If `roundingApplication = "total"`, the derivation sequence is:
  - If `net` is provided:
    ```
    gross = round(net * (taxRate / 100 + 1))
    tax = gross - net
    ```
  - If `gross` is provided:
    ```
    net = round(gross / (taxRate / 100 + 1))
    tax = gross - net
    ```

### Validation rules

RelMon defines the following rules for validating monetary values:

- **Data types**
  - All fields MUST conform to their abstract type definitions.
  - `net`, `tax`, and `gross` MUST be decimals unless **minors mode** (`m`) is active, in which case they all MUST be integers. 
- **Consistency**
  - `gross = net + tax`
  - `net = gross - tax`
  - `gross` MAY be less than `net` only if all values are negative.
  - If `taxRate` on the root-level is present, it MUST be the same for all components.
- **Precision**
  - If `precision` is specified, all decimal values in `net`, `tax`, and `gross` MUST comply with the provided precision value.
- **Optional fields**
  - Any OPTIONAL fields, if present, MUST conform to their abstract type definitions.
  - If `scope = "c"`, the `components` field MUST be present.
  - If `rounding` field is specified, at least `mode` or `application` MUST be present.
- **Components**
  - Each `MonetaryComponent` MUST respect the monetary bases.
  - The OPTIONAL `taxRate` in a component MUST comply with the format defined in the abstract model.
  - The OPTIONAL `comment` in a component MUST NOT affect numeric calculations.
  - The sum of all component `net` values MUST equal the root-level `net`;
  - The sum of all component `gross` values MUST equal the root-level `gross`;
  - The sum of all component `tax` values MUST equal the root-level `tax`.

### Summary table of the model fields

**Root-level fields**

| Field                       | Compact notation | Type                | Possible values                                                                              | Default value                             | DL1                                                                               | DL2                                                                               | DL3                                   | Description                                       |
|-----------------------------|------------------|---------------------|----------------------------------------------------------------------------------------------|-------------------------------------------|-----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|---------------------------------------|---------------------------------------------------|
| protocol                    | p                | ProtocolIdentifier  | N/A                                                                                          | N/A                                       | REQUIRED                                                                          | REQUIRED                                                                          | REQUIRED                              | The protocol version, determinism level and modes |
| net                         | n                | decimal \| integer  | N/A                                                                                          | N/A                                       | Derived from `gross`.                                                             | REQUIRED                                                                          | OPTIONAL (if `gross` provided)        | Total net amount                                  |
| gross                       | g                | decimal \| integer  | N/A                                                                                          | N/A                                       | Derived from `net`                                                                | REQUIRED                                                                          | OPTIONAL (if `net` provided)          | Total gross amount                                |
| tax                         | t                | decimal \| integer  | N/A                                                                                          | N/A                                       | OPTIONAL                                                                          | OPTIONAL                                                                          | REQUIRED                              | Total tax amount                                  |
| taxRate                     | tr               | decimal(6,3)        | N/A                                                                                          | N/A                                       | REQUIRED (if calculation scope = `r` and tax rate is the same for all components) | REQUIRED (if calculation scope = `r` and tax rate is the same for all components) | OPTIONAL                              | Tax rate, if it is the same for all components    |
| unit                        | u                | text                | N/A                                                                                          | N/A                                       | OPTIONAL                                                                          | OPTIONAL                                                                          | OPTIONAL                              | Unit code (e.g. ISO4217)                          |
| precision                   | pr               | integer             | N/A                                                                                          | N/A                                       | OPTIONAL                                                                          | OPTIONAL                                                                          | OPTIONAL                              | Number of decimal places                          |
| scope                       | s                | text                | `c`, `r`                                                                                     | `r`                                       | OPTIONAL                                                                          | OPTIONAL                                                                          | OPTIONAL                              | Calculation scope                                 |
| rounding{mode, application} | r{m, a}          | text                | `{"mode": "haway" \| "hzero" \| "heven" \| "up" \| "down", "application": "tax" \| "total"}` | `{"mode": "heven", "application": "tax"}` | OPTIONAL                                                                          | OPTIONAL                                                                          | OPTIONAL                              | Rounding mode                                     |
| components                  | cs               | MonetaryComponent[] | N/A                                                                                          | N/A                                       | OPTIONAL (if calculation scope = `r`)                                             | OPTIONAL (if calculation scope = `r`)                                             | OPTIONAL (if calculation scope = `r`) | List of components (if calculation scope = `c`)   |

**Component-level fields**

| Field   | Compact notation | Type               | Possible values | Default value | DL1                   | DL2      | DL3                            | Description                                |
|---------|------------------|--------------------|-----------------|---------------|-----------------------|----------|--------------------------------|--------------------------------------------|
| net     | n                | decimal \| integer | N/A             | N/A           | Derived from `gross`. | REQUIRED | OPTIONAL (if `gross` provided) | Net amount                                 |
| gross   | g                | decimal \| integer | N/A             | N/A           | Derived from `net`    | REQUIRED | OPTIONAL (if `net` provided)   | Gross amount                               |
| tax     | t                | decimal \| integer | N/A             | N/A           | OPTIONAL              | OPTIONAL | REQUIRED                       | Tax amount                                 |
| taxRate | tr               | decimal(6,3)       | N/A             | N/A           | REQUIRED              | REQUIRED | OPTIONAL                       | Tax rate                                   |
| comment | c                | text               | N/A             | N/A           | No                    | No       | No                             | OPTIONAL arbitrary comment for a component |

## Concrete default data format implementations

RelMon defines multiple **concrete default data format implementations** to represent the abstract monetary model. This section provides a high-level overview and references for each format.

Default format implementations as of this version of the protocol are: JSON, XML, and multiple URI-based notations - JSON, XML, and minimalistic DL3 textual representation.

RelMon MAY be implemented in any other data format.

The **abstract model** remains unchanged across formats; these implementations define **how the abstract fields map to JSON, XML, or URI-based representations**.

### JSON

- All modes of the protocol are supported.
- The name of the element containing a RelMon object MAY be arbitrary.

The following example shows a RelMon object at DL3 with `components` (calculation scope = `c`).

```json
{
    "protocol": "relmon@1.0.0/3",
    "net": "100.00",
    "gross": "121.00",
    "tax": "21.00",
    "taxRate": "21.00",
    "unit": "EUR",
    "precision": 2,
    "scope": "c",
    "rounding": {"mode": "haway", "application": "tax"},
    "components": [
        {
            "net": "50.00",
            "gross": "60.50",
            "tax": "10.50",
            "taxRate": "21.00",
            "comment": "Base price of item 1"
        },
        {
            "net": "50.00",
            "gross": "60.50",
            "tax": "10.50",
            "taxRate": "21.00",
            "comment": "Base price of item 2"
        }
    ]
}
```

Another example (DL1) in **compact** mode.

```json
{"p": "relmon@1.0.0/1:c", "n": "100.00", "tr": "21.00"}
```

### XML

- All modes of the protocol are supported.
- The root name of the element MAY be arbitrary (e.g. `<Money>`, `<Price>`, etc.).

The following example shows a RelMon object at DL3 without `components`.

```xml
<RelMon>
    <protocol>relmon@1.0.0/3</protocol>
    <net>100.00</net>
    <gross>121.00</gross>
    <tax>21.00</tax>
    <taxRate>21.00</taxRate>
    <unit>EUR</unit>
    <precision>2</precision>
    <rounding>
        <mode>haway</mode>
        <application>tax</application>
    </rounding>
</RelMon>
```

Another example (DL3) in **compact** and **minors** mode, with the OPTIONAL `unit` field.

```xml
<RelMon>
    <p>relmon@1.0.0/3:c.m</p>
    <n>10000</n>
    <g>12100</g>
    <t>2100</t>
    <u>EUR</u>
</RelMon>
```

### URI-based notations

URI-based notations are compact, single-string representations suitable for **URLs**, **QR codes**, or **lightweight transfers**.

All modes are supported in all URI-based notations.

The name of the field containing the URI-based notation of RelMon MAY be arbitrary.

| Scheme              | Description                                                                                                                                                                                                                                                                                                                                                      | Format                                                  | Example                                                                                                                                                                 |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------| 
| `relmon-json://...` | JSON representation encoded in base64url as defined in RFC 4648 Section 5                                                                                                                                                                                                                                                                                        | `relmon-json://base64EncodedJson`                       | `relmon-json://eyJwIjogInJlbG1vbkAxLjAuMC8xOmMiLCAibiI6ICIxMDAuMDAiLCAidHIiOiAiMjEuMDAifQ==`                                                                            |
| `relmon-xml://...`  | XML representation encoded in base64url as defined in RFC 4648 Section 5                                                                                                                                                                                                                                                                                         | `relmon-xml://base64EncodedXml`                         | `relmon-xml://PFJlbE1vbj4KICAgIDxwPnJlbG1vbkAxLjAuMC8zOmMubTwvcD4KICAgIDxuPjEwMDAwPC9uPgogICAgPGc-MTIxMDA8L2c-CiAgICA8dD4yMTAwPC90PgogICAgPHU-RVVSPC91Pgo8L1JlbE1vbj4=` |
| `relmon-min://...`  | Minimalistic textual representation with DL3 containing only the protocol identifier, net, gross, and tax values separated by `;`.<br/><br/>**Note**:<br/>- In this notation the `ProtocolIdentifier` is shortened by removing `relmon@` prefix.<br/>-This notation supports minors via `m` mode.<br/> - The compact mode `c` is not supported in this notation. | `relmon-min://<ProtocolIdentifier>;<net>;<gross>;<tax>` | `relmon-min://1.0.0/3:m;10000;12100;2100`                                                                                                                               |

## FAQ and design rationale

This section is **informative only** and does not contain requirements. It explains key design decisions of RelMon to help implementers understand its rationale.

### Why does RelMon exist and how can it help systems?

RelMon exists because monetary data exchanged between systems is prone to silent inconsistencies. Two systems may independently compute the same invoice and arrive at different results due to differing rounding modes, precision, derivation order, or implicit assumptions about which value is authoritative.

Without a shared contract, these discrepancies are often discovered late - during reconciliation, auditing, or customer disputes - and are difficult to trace back to their root cause.

RelMon addresses this by providing an explicit, agreed-upon protocol that defines:

- which monetary values are provided and which are derived;
- how rounding and precision are applied;
- what the authoritative calculation sequence is.

By adopting RelMon, systems establish a deterministic contract for monetary data exchange, eliminating ambiguity and ensuring that all parties reconstruct identical results from the same inputs.

### Why different determinism levels?

Determinism levels (DL1–DL3) exist to define how much arithmetic is REQUIRED by consuming systems to reconstruct monetary values. They allow systems to adapt their implementations depending on what the data provider can or wants to supply:

- **DL1** (derivable): Only minimal values are provided (net or gross + taxRate). Receiving systems MUST compute the missing components. This level is flexible but places more responsibility on receivers for accurate rounding and derivation.
- **DL2** (partially materialized): Some values are included even if derivable (net + taxRate + gross), which helps reduce derivation effort and allows systems to validate calculations against the sender's values.
- **DL3** (fully materialized / derivation-free): All monetary components (net, tax, gross) are explicitly provided. Receiving systems do not need to perform arithmetic and can rely directly on the values, minimizing risk of rounding or calculation errors.

Determinism levels define a contract between data providers and consumers, clarifying what calculations are expected and what is guaranteed, while still maintaining full precision and consistent rounding.

### Why not allow arbitrary precision for net, tax, and gross?

RelMon intentionally restricts monetary values to a defined `precision` to ensure **deterministic reconstruction and consistent validation across systems**.

If arbitrary precision were allowed:

- Different systems might interpret or round values differently.
- Minor representation differences (for example `100.005` vs `100.004999`) could lead to different derived results.
- Validation rules such as `gross = net + tax` could fail after reconstruction due to inconsistent rounding behavior.

By requiring that all monetary values respect the declared precision:

- All systems operate on the **same canonical representation** of the amount.
- Rounding occurs in a **well-defined and predictable place** in the calculation process.
- Implementations can safely verify consistency and reproduce results.

RelMon does not restrict how systems internally store or compute values. Systems MAY use higher internal precision during calculations, but values exchanged through the protocol MUST conform to the declared precision to ensure interoperability and deterministic behavior.

### Why is `taxRate` a decimal(6,3)?

To support tax rates with **fractional precision**, e.g., 12.35% or 0.625%. The (6,3) format allows up to 999.999% while keeping three decimal places, which covers nearly all real-world tax regimes.

### Why is specifying different tax rates within a single component not allowed?

A `MonetaryComponent` represents a single taxable unit - one line item, one charge, or one fee - at a single tax rate. This is a deliberate design constraint.

Allowing multiple tax rates within a single component would require defining how those rates interact: whether they are additive, compounded (tax-on-tax), or applied in sequence. This introduces significant complexity in derivation, rounding order, and validation, which would undermine the protocol's goal of deterministic simplicity.

For items subject to multiple tax rates, the recommended approach is to split them into separate components, each with its own tax rate. This keeps the derivation rules unambiguous and the validation straightforward.

Support for compound or multi-rate taxation within a single component may be considered in a future major version of the protocol.

### Why provide a `precision` field?

Precision defines the **maximum decimal places** for monetary values. This ensures that values are consistently formatted and interpreted across systems. It is optional and MAY be provided to make derivation parameters explicit. If not specified, precision is inferred from the provided monetary values.

### Why include a rounding mode and application field?

Rounding rules are critical for deterministic monetary calculations. Even with the same net, tax, and gross values, different rounding methods can produce mismatched results across systems.

- Rounding mode specifies how rounding is performed (half-away-from-zero, half-towards-zero, half-even, up, down).
- Rounding application defines where rounding is applied during derivation:
  - `tax`: round the tax first, then calculate gross.
  - `total`: calculate gross first, then round the total or net.

By explicitly including both fields, RelMon ensures that all systems derive identical monetary values from the same inputs, preventing subtle discrepancies due to rounding differences. This is especially important for DL1, where derivation is necessary.

### Why is the default rounding mode - "heven", and the default rounding application - "tax"?

RelMon chooses `heven` (**half-even**) as the default rounding mode because it is widely used in finance and accounting for minimizing cumulative rounding bias:

- It avoids systematic upward or downward drift when summing multiple rounded amounts (e.g., across many components or invoices).
- It is consistent with common financial standards in banking, accounting, and tax reporting.

The default `tax` application is chosen because:

- Rounding the tax first (`tax = round(net * (taxRate / 100))`) is more consistent with legal and fiscal practices, where tax amounts are usually calculated per line item or per subtotal.
- It ensures that `gross = net + tax` remains exact, minimizing discrepancies when summing across multiple components.
- It aligns naturally with the typical use case: most systems know the net amount and compute tax as the primary derived value.

Together, these defaults provide a safe, finance-compatible baseline for deterministic calculations, allowing implementers to override them only if a specific business rule or legal requirement dictates otherwise.

### Why have compact and minors modes?

- **Compact** (`c`): Shortened field names to save bandwidth or storage.
- **Minors** (`m`): Integer representation for systems that operate in the smallest unit (e.g., cents, satoshis).

These modes give flexibility while maintaining **semantic consistency**.

### Can RelMon handle cryptocurrencies like BTC or ETH?

Yes. While ISO 4217 codes are suggested for fiat currencies, **custom currencies** are supported via the `unit` field. **Minors** mode can store satoshis, wei, or other smallest units as integers.

### Why allow negative values?

Negative values represent refunds, reversals, credits, or discounts. Consistent rules ensure that **arithmetic invariants** (`gross = net + tax`) and component sums remain valid, even when values are negative.

### Why is the protocol language, storage, and data format agnostic?

To allow **maximum interoperability**. The abstract model defines **semantic meaning**, while concrete data format implementations (JSON, XML, URI) map these fields into specific representations. This ensures consistency across systems, programming languages, and storage backends, and allows scaling across a wide range of computing systems.

## Changelog

| Date       | Version    | Description                                   |
|------------|------------|-----------------------------------------------|
| 2026-01-17 | draft      | First draft version of the protocol published |
| 2026-02-22 | draft      | Reworked the draft version of the protocol    |
| 2026-02-23 | 1.0.0-beta | BETA 1.0.0 version                            |

## TODOs

- Add OpenAPI schema for JSON format.
- Add XSD for XML format.
- Add possible validation/error codes.
- Add a conformance test suite.
