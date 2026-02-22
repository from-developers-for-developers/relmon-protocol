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
    * [Format agnostic](#format-agnostic)
    * [Protocol identifier](#protocol-identifier)
      * [Protocol modes](#protocol-modes)
    * [Monetary values](#monetary-values)
    * [Monetary bases](#monetary-bases)
    * [Calculation scope](#calculation-scope)
    * [Arithmetics](#arithmetics)
      * [Precision](#precision)
      * [Rounding modes](#rounding-modes)
      * [Rounding application](#rounding-application)
      * [Rounding function](#rounding-function)
    * [Units](#units)
    * [Tax rate on the root level](#tax-rate-on-the-root-level)
    * [Optional metadata](#optional-metadata)
    * [Signed values semantics](#signed-values-semantics)
    * [Validation rules](#validation-rules)
    * [Determinism levels](#determinism-levels)
    * [Summary table of the model fields](#summary-table-of-the-model-fields)
  * [Concrete default data format implementations](#concrete-default-data-format-implementations)
    * [JSON](#json)
    * [XML](#xml)
    * [URI-based notations](#uri-based-notations)
  * [FAQ and design rationale](#faq-and-design-rationale)
    * [**Why different determinism levels?**](#why-different-determinism-levels)
    * [Why is `taxRate` a decimal(6,3)?](#why-is-taxrate-a-decimal63)
    * [Why provide a `precision` field?](#why-provide-a-precision-field)
    * [Why include a rounding mode and application field?](#why-include-a-rounding-mode-and-application-field)
    * [Why is the default rounding mode - "heven", and the default rounding application - "tax"?](#why-is-the-default-rounding-mode---heven-and-the-default-rounding-application---tax)
    * [Why have Compact and Minors modes?](#why-have-compact-and-minors-modes)
    * [Can RelMon handle cryptocurrencies like BTC or ETH?](#can-relmon-handle-cryptocurrencies-like-btc-or-eth)
    * [Why allow negative values?](#why-allow-negative-values)
    * [Why is the protocol language, storage, and data format agnostic?](#why-is-the-protocol-language-storage-and-data-format-agnostic)
  * [Changelog](#changelog)

## Introduction

This document specifies the **RelMon protocol** (Reliable Monetary Data Protocol, further referred as **RelMon**) and defines its **concrete default data format implementations**.

RelMon ensures deterministic reconstruction of monetary components across systems. By standardizing the structure and semantics of monetary data, it ensures interoperable implementations across diverse systems, preserving **exact monetary values**, full precision, consistent rounding, and, following the maximum determinism level, a **derivable-free approach** for net, tax, and gross amounts.

RelMon is programming language, storage, and data format agnostic, defining only the **abstract logical model** and **the concrete default data format implementations**.

An example of using RelMon as JSON data format (determinism level 3):

```json
{"protocol": "relmon@1.0.0/3", "net": "100.00", "tax": "21.00", "gross": "121.00"}
```

## Key words

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

## License

RelMon specification is published as an **open and freely usable standard**.

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

- `?` indicates an optional field
- `:` indicates a type (`object`, `list`, `decimal`, `integer`, `text`)
- `=` indicates a value
- `|` - a separator for possible scalar values
- `[?` indicates the optional part
- `...` indicates that the construction can be repeated multiple times
- `[]` indicates an ordered list
- `,` - a separator for elements in the ordered list
- `xxx:` (string followed by `:` symbol) indicates the meaning of the element in the ordered list
- Capitalized identifiers denote abstract types

### The model

```
RelMonObject: object =
{
    protocol: ProtocolIdentifier

    net?: decimal | integer
    gross?: decimal | integer
    tax?: decimal | integer
    taxRate?: decimal(6, 3)

    unit?: text
    precision?: integer
    scope?: CalculationScope
    rounding?: Rounding
    components?: MonetaryComponent[]
}

ProtocolIdentifier: text = "relmon@MAJOR.MINOR.PATCH/DeterminismLevel[?:MODE[?.MODE...]]"
MAJOR: integer
MINOR: integer
PATCH: integer
MODE?: text
DeterminismLevel: integer = 1 | 2 | 3

CalculationScope: text = "r" | "a"

Rounding: object = {
    mode?: text = "hup" | "hdown" | "heven" | "up" | "down",
    application?: text = "tax" | "total"
}

MonetaryComponent: object =
{
    net?: decimal | integer
    gross?: decimal | integer
    tax?: decimal | integer
    taxRate?: decimal(6, 3)
    comment?: text
}
```

## Protocol specification

### Format agnostic

The abstract model does not prescribe JSON, XML, URI, or binary layouts. 
Concrete data format implementations map abstract fields to specific formats, as defined in separate sections.

### Protocol identifier

- **Protocol version**: `protocol` MUST follow the defined `ProtocolIdentifier` format and reflect the SemVer version of the abstract model.
- **Protocol determinism level**: protocol identifier MUST include the determinism level of the protocol. 
- **Protocol modes**: `protocol` MAY define its modes affecting the format and the fields of the protocol. Available modes are: `c` (compact), `m` (minors).

#### Protocol modes

RelMon defines two modes that modify the format and the set of the fields of the protocol. 
The modes MUST NOT change the logical model of the protocol. 
The order of mode declaration MUST NOT affect the RelMon logical model.

- **Compact**: Compact mode (`c`) defines that the names of the fields of the `RelMonObject` are written in the compact mode. Compact mode affects field identifiers only and MUST NOT alter semantic meaning or numeric behavior. This mode MUST be identified as `c`.
  - `protocol` is represented via `p`
  - `net` is represented via `n`
  - `tax` is represented via `t`
  - `gross` is represented via `g`
  - `taxRate` is represented via `tr`
  - `unit` is represented via `u`
  - `precision` is represented via `pr`
  - `scope` is represented via `s`
  - `rounding` is represented via `r`
  - `components` is represented via `cs`
- **Minors**: Minors mode (`m`) defines that the fields `net`, `gross` and `tax` of `RelMonObject` are the smallest units of the currency or asset and MUST be represented as integers. This mode MUST be identified as `m`.

An example: `relmon@1.0.0/3:c.m`.

### Monetary values

- Fields `net`, `gross`, `tax` are exact and MUST preserve full precision.
- They MUST be represented as decimals unless the `m` mode (minors) is used.

### Monetary bases

`RelMonObject` MUST contain one of the following valid monetary bases:

- Basis 1 - **Rate-based derivation**: `net AND taxRate` or `gross AND taxRate`
- Basis 2 - **Tax-amount provided**: `net AND tax` or `gross AND tax`
- Basis 3 - **Fully materialized**: `net AND tax AND gross`

### Calculation scope

One of the following calculation scopes MUST be used:

- `c` - calculation scope is **component-based**, meaning that all derivations are performed on a per-component basis;
  - including `components` field is REQUIRED in this case.
- `r` - calculation scope is **root-based**, meaning that all derivations are performed on the root-level `net` or `gross` field.

Regardless of the calculation scope, the root-level `net`, `gross` and `tax` values MUST be respectively equal to the sum of all per-component `net`, `gross` and `tax` fields.

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

Summing up `net`, `gross` and `tax` values of each component would result in the root-level `net`, `gross` and `tax` values.

Implementations MUST implement the **default** calculation scope `r`, unless:

- an explicit calculation scope is specified;
- or a mutual agreement for the default value is set among involved systems.

### Arithmetics

- Implementations MUST NOT perform floating-point arithmetic.
- Implementations MUST produce results identical to exact decimal arithmetic under the declared:
  - precision;
  - rounding mode;
  - calculation scope.
- Allowed arithmetic strategies include:
  - decimal arithmetic; 
  - integer minor-unit arithmetic; 
  - fixed-scale decimals.
- Binary floating-point arithmetic MUST NOT be used unless it guarantees identical results.

#### Precision

Precision defines the scale at which rounding occurs.

An example `precision = 2` means rounding to two decimal places: `10.336 --> 10.34`.

#### Rounding modes

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

Implementations MUST implement the **default** rounding mode `heven`, unless:

- an explicit rounding mode is specified;
- or a mutual agreement for the default value is set among involved systems.

#### Rounding application

The rounding may be applicable to:

- `tax` - the rounding function is applied to `tax` after the derivation;
- `total` - the rounding function is applied to `gross` or `net` after the derivation.

Implementations MUST implement the **default** rounding application `tax`, unless:

- an explicit rounding application is specified;
- or a mutual agreement for the default value is set among involved systems.

#### Rounding function

RelMon defines a deterministic rounding function: `round(value, precision, roundingMode = "heven", roundingApplication = "tax")`. Where:

- `value` is a decimal number to be rounded.
- `precision` defines the number of decimal places after the decimal point.
- `roundingMode` defines how ties and directional rounding are handled (default value `heven`).
- `roundingApplication` defines how exactly the rounding is applied (default value `tax`).

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
    net = gross / (taxRate / 100 + 1)
    tax = round(gross - net)
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

### Units

When the unit represents a fiat currency, it is RECOMMENDED for implementations to use the corresponding ISO 4217 currency code in the `unit` field. Examples: `EUR`, `USD`, `RUB`.

### Tax rate on the root level

The `taxRate` on the root level of `RelMonObject` MAY be present if the tax rate is the same across all components.

### Optional metadata

Fields `comment` in `MonetaryComponent` is purely informational and MUST NOT affect numeric calculations. This field is ALWAYS optional.

### Signed values semantics

Monetary values in RelMon may be negative. Implementations MUST handle negative values as follows:

- Fields `net`, `gross` and `tax` MAY be negative (prefixed with `-` symbol) for both decimals or minors.
- Positive values MUST NOT have any prefix (no `+` prefix).
- All root-level fields (`net`, `gross` and `tax`) MUST have the same sign.
- Components MAY independently be positive or negative, but each component's `net` and `tax` MUST share the same sign.
- Rounding modes specified in the `rounding` field apply symmetrically to negative values.

Implementations MUST ensure that arithmetic consistency rules (`gross = net + tax` and component sums) remain valid even when values are negative.

### Validation rules

RelMon defines the following rules for validating monetary values:

- **Consistency**
    - `gross = net + tax`
    - `net = gross - tax`
    - If `components` are present:
      - the sum of all component `net` values MUST equal the root-level `net`; 
      - the sum of all component `tax` values MUST equal the root-level `tax`;
      - the sum of all component `gross` values MUST equal the root-level `gross`.
- **Data types**
    - `net`, `tax`, and `gross` MUST be decimals unless **Minors mode** (`m`) is active, in which case they MUST be integers.
- **Precision**
    - If `precision` is specified, all decimal values in `net`, `tax`, and `gross` MUST comply with the provided precision value.
- **Optional fields**
    - Any optional fields, if present, MUST conform to their abstract type definitions.
- **Components**
    - Each `MonetaryComponent` MUST respect the monetary bases.
    - The optional `taxRate` in a component MUST comply with the format defined in the abstract model.
    - The optional `comment` in a component MUST NOT affect numeric calculations.

### Determinism levels

RelMon defines several determinism levels. A determinism level defines how many monetary values must be derived from other fields.

Each level specifies a set of fields that determines how much arithmetic is required to derive missing values. 
Lower levels require more calculations, while higher levels require fewer, with the highest level requiring no derivation at all.

Determinism level further will be referred to as "DL".

| Determinism level | Description                                                                                                                                                                                                                                                                                                          | Fields required                                  | Requirements                                                                                                                                         |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| DL1               | Minimum fields allowing full reconstruction.<br/>Only `net` or `gross` value is sent.<br/>The missing amount must be derived.                                                                                                                                                                                        | - `net AND taxRate`<br/>- or `gross AND taxRate` | - rounding mode MUST be defined<br/>- calculation scope MUST be defined (root/component)<br/>- deterministic reconstruction required                 |
| DL2               | Derived values partially materialized to ensure consistency.<br/>A derived value is included even though it can be recomputed.<br/><br/>Purpose:<br/><br/>- validation across systems<br/>- detect drift or calculation differences<br/>- improves interoperability<br/>- `gross` and `net` are authoritative values | `net AND taxRate AND gross`                      | - Materialized values MUST match recomputed values<br/>- Rounding application to `tax` is optional in DL2 only if all derivable amounts are included |
| DL3               | All monetary values explicitly provided.<br/>No reconstruction needed.                                                                                                                                                                                                                                               | `net AND gross AND tax`                          | `gross = net + tax` and `net = gross - tax`                                                                                                          |

### Summary table of the model fields

**Root-level fields**

| Field                       | Compact notation | Type                | Possible values                                                                            | Default value                             | DL1                                   | DL2                                   | DL3                                   | Description                                               |
|-----------------------------|------------------|---------------------|--------------------------------------------------------------------------------------------|-------------------------------------------|---------------------------------------|---------------------------------------|---------------------------------------|-----------------------------------------------------------|
| protocol                    | p                | ProtocolIdentifier  | N/A                                                                                        | N/A                                       | Required                              | Required                              | Required                              | The protocol version, determinism level and modes         |
| net                         | n                | decimal \| integer  | N/A                                                                                        | N/A                                       | Derived from `gross`.                 | Required                              | Optional (if `gross` provided)        | Total net amount                                          |
| gross                       | g                | decimal \| integer  | N/A                                                                                        | N/A                                       | Derived from `net`                    | Required                              | Optional (if `net` provided)          | Total gross amount                                        |
| tax                         | t                | decimal \| integer  | N/A                                                                                        | N/A                                       | Optional                              | Optional                              | Required                              | Total tax amount                                          |
| taxRate                     | tr               | decimal(6,3)        | N/A                                                                                        | N/A                                       | Required                              | Required                              | Optional                              | Tax rate if it is the same for all components             |
| unit                        | u                | text                | N/A                                                                                        | N/A                                       | Optional                              | Optional                              | Optional                              | Unit code (e.g. ISO4217)                                  |
| precision                   | pr               | integer             | N/A                                                                                        | N/A                                       | Required                              | Required                              | Optional                              | Decimal value precision                                   |
| scope                       | s                | text                | `c`, `r`                                                                                   | `r`                                       | Optional                              | Optional                              | Optional                              | Calculation scope                                         |
| rounding{mode, application} | r{m, a}          | text                | `{"mode": "hup" \| "hdown" \| "heven" \| "up" \| "down", "application": "tax" \| "total"}` | `{"mode": "heven", "application": "tax"}` | Required                              | Required                              | Optional                              | Rounding mode                                             |
| components                  | cs               | MonetaryComponent[] | N/A                                                                                        | N/A                                       | Optional (if calculation scope = `r`) | Optional (if calculation scope = `r`) | Optional (if calculation scope = `r`) | List of components (necessary if calculation scope = `c`) |

**Component-level fields**

| Field   | Compact notation | Type               | Possible values | Default value | DL1                   | DL2      | DL3                            | Description                                |
|---------|------------------|--------------------|-----------------|---------------|-----------------------|----------|--------------------------------|--------------------------------------------|
| net     | n                | decimal \| integer | N/A             | N/A           | Derived from `gross`. | Required | Optional (if `gross` provided) | Net amount                                 |
| gross   | g                | decimal \| integer | N/A             | N/A           | Derived from `net`    | Required | Optional (if `net` provided)   | Gross amount                               |
| tax     | t                | decimal \| integer | N/A             | N/A           | Optional              | Optional | Required                       | Tax amount                                 |
| taxRate | tr               | decimal(6,3)       | N/A             | N/A           | Required              | Required | Optional                       | Tax rate                                   |
| comment | c                | text               | N/A             | N/A           | No                    | No       | No                             | Optional arbitrary comment for a component |

## Concrete default data format implementations

RelMon defines multiple **concrete default data format implementations** to represent the abstract monetary model. This section provides a high-level overview and references for each format.

Default format implementation as of this version of the protocol are: JSON, XML, and multiple URI-based notations - JSON, XML, minimalistic DL3 textual representation.

RelMon is open to be implemented in any other data format.

The **abstract model** remains unchanged across formats; these implementations define **how the abstract fields map to JSON, XML, or URI-based**.

### JSON

- All modes of the protocol are supported.
- The name of the element containing RelMon object MAY be arbitrary.

The example here includes the RelMon object with DL3, including `components` fields (calculation scope = `c`).

```json
{
    "protocol": "relmon@1.0.0/3",
    "net": "100.00",
    "gross": "121.00",
    "tax": "21.00",
    "taxRate": "0.210",
    "unit": "EUR",
    "precision": 2,
    "scope": "c",
    "rounding": {"mode": "hup", "application": "tax"},
    "components": [
        {
            "net": "50.00",
            "gross": "60.50",
            "tax": "10.50",
            "taxRate": "0.210",
            "comment": "Base price of item 1"
        },
        {
            "net": "50.00",
            "gross": "60.50",
            "tax": "10.50",
            "taxRate": "0.210",
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

The example here includes the RelMon object in DL3, without `components` fields.

```xml
<RelMon>
    <protocol>relmon@1.0.0/3</protocol>
    <net>100.00</net>
    <gross>121.00</gross>
    <tax>21.00</tax>
    <taxRate>0.210</taxRate>
    <unit>EUR</unit>
    <precision>2</precision>
    <rounding>
        <mode>hup</mode>
        <application>tax</application>
    </rounding>
</RelMon>
```

Another example (DL3) in **compact** and **minors** mode, passing optional `unit` field.

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

Compact, single-string representations suitable for **URLs**, **QR codes**, or **lightweight transfers**. 

All modes are supported in all URI-based notations.

The name of the field containing the URI-based notation of RelMon MAY be arbitrary.

| Scheme              | Description                                                                                                                      | Example                                                                                                                                                                 |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------| 
| `relmon/json://...` | JSON representation (as base64) encoded in URI                                                                                   | `relmon/json://eyJwIjogInJlbG1vbkAxLjAuMC8xOmMiLCAibiI6ICIxMDAuMDAiLCAidHIiOiAiMjEuMDAifQ==`                                                                            |
| `relmon/xml://...`  | XML representation (as base64) encoded in URI                                                                                    | `relmon/xml://PFJlbE1vbj4KICAgIDxwPnJlbG1vbkAxLjAuMC8zOmMubTwvcD4KICAgIDxuPjEwMDAwPC9uPgogICAgPGc+MTIxMDA8L2c+CiAgICA8dD4yMTAwPC90PgogICAgPHU+RVVSPC91Pgo8L1JlbE1vbj4=` |
| `relmon/min://...`  | Minimalistic textual representation with DL3 containing only the protocol identifier, net, gross and tax values separated by `;` | `relmon/min://1.0.0/3:m;10000;12100;2100`                                                                                                                               |

## FAQ and design rationale

This section is **informative only** and does not contain requirements. It explains key design decisions of RelMon to help implementers understand its rationale.

### **Why different determinism levels?**

Determinism levels (DL1–DL3) exist to define how much arithmetic is required by consuming systems to reconstruct monetary values. They allow systems to adapt their implementations depending on what the data provider can or wants to supply:

- **DL1** (derivable): Only minimal values are provided (net or gross + taxRate). Receiving systems must compute the missing components. This level is flexible but places more responsibility on receivers for accurate rounding and derivation.
- **DL2** (partially materialized): Some values are included even if derivable (net + taxRate + gross), which helps reduce derivation effort and allows systems to validate calculations against the sender's values.
- **DL3** (fully materialized / derivation-free): All monetary components (net, tax, gross) are explicitly provided. Receiving systems do not need to perform arithmetic and can rely directly on the values, minimizing risk of rounding or calculation errors.

Determinism levels define a contract between data providers and consumers, clarifying what calculations are expected and what is guaranteed, while still maintaining full precision and consistent rounding.

### Why is `taxRate` a decimal(6,3)?

To support tax rates with **fractional precision**, e.g., 12.35% or 0.625%. The (6,3) format allows up to 999.999% while keeping three decimal places, which covers nearly all real-world tax regimes.

### Why provide a `precision` field?

Precision defines the **maximum total digits and fractional scale** for monetary values. This ensures that values are consistently formatted and interpreted across systems. It is optional, but required for extended mode.

### Why include a rounding mode and application field?

Rounding rules are critical for deterministic monetary calculations. Even with the same net, tax, and gross values, different rounding methods can produce mismatched results across systems.

- Rounding mode specifies how rounding is performed (half-up, half-down, half-even, up, down).
- Rounding application defines where rounding is applied during derivation:
  - `tax`: round the tax first, then calculate gross.
  - `total`: calculate gross first, then round the total or net.

By explicitly including both fields, RelMon ensures that all systems derive identical monetary values from the same inputs, preventing subtle discrepancies due to rounding differences. This is especially important for DL1, where derivation is necessary.

### Why is the default rounding mode - "heven", and the default rounding application - "tax"?

RelMon chooses `heven` (**half-even**) as the default rounding mode because it is widely used in finance and accounting for minimizing cumulative rounding bias:

- It avoids systematic upward or downward drift when summing multiple rounded amounts (e.g., across many components or invoices).
- It is consistent with common financial standards in banking, accounting, and tax reporting.

The default `tax` application is chosen because:

- Rounding the tax first (`tax = round(net * taxRate)`) is more consistent with legal and fiscal practices, where tax amounts are usually calculated per line item or per subtotal.
- It ensures that `gross = net + tax` remains exact, minimizing discrepancies when summing across multiple components.
- It aligns naturally with the typical use case: most systems know the net amount and compute tax as the primary derived value.

Together, these defaults provide a safe, finance-compatible baseline for deterministic calculations, allowing implementers to override them only if a specific business rule or legal requirement dictates otherwise.

### Why have Compact and Minors modes?

- **Compact** (`c`): Shortened field names to save bandwidth or storage.
- **Minors** (`m`): Integer representation for systems that operate in the smallest unit (e.g., cents, satoshis).

These modes give flexibility while maintaining **semantic consistency**.

### Can RelMon handle cryptocurrencies like BTC or ETH?

Yes. While ISO 4217 codes are suggested for fiat currencies, **custom currencies** are supported via `unit` field. **Minors** mode can store satoshis, wei, or other smallest units as integers.

### Why allow negative values?

Negative values represent refunds, reversals, credits, or discounts. Consistent rules ensure that **arithmetic invariants** (`gross = net + tax`) and component sums remain valid, even when values are negative.

### Why is the protocol language, storage, and data format agnostic?

To allow **maximum interoperability**. The abstract model defines **semantic meaning**, while concrete data format implementations (JSON, XML, URI) map these fields into specific representations. This ensures consistency across systems, programming languages, and storage backends, and eventually allows to scale and expand on wide ranges of computing systems.

## Changelog

| Date | Version | Description |
| ------ | ---------- | --------------- |
| 2026-01-17 | 1.0.0 | First version of the protocol published |
