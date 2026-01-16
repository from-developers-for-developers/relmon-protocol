# Reliable Monetary data protocol (RelMon protocol)

## Introduction

This document specifies the **RelMon protocol** (Reliable Monetary Data Protocol) and provides an overview of its **concrete format implementations**.

The RelMon protocol is designed to enable reliable exchange of monetary data between systems. By standardizing the structure and semantics of monetary data, it ensures interoperable implementations across diverse systems, preserving **exact monetary values**, full precision, consistent rounding, and a **calculation-free approach** for net, tax, and gross amounts.

The RelMon protocol is programming language, storage, and data format agnostic, defining only the **abstract logical model** and **the concrete format implementations**.

## Key Words

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

These terms define **normative requirements** for implementers and clarify which behavior is mandatory, recommended, or optional.

## Versioning 

The RelMon protocol specification uses [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR** version changes indicate incompatible modifications to the abstract protocol model.
- **MINOR** version changes indicate backward-compatible extensions, such as new optional fields or formats.
- **PATCH** version changes indicate non-breaking corrections, clarifications, or editorial updates.

Protocol identifiers include the version, for example:

- `relmon@1.0` – abstract protocol version 1.0

Implementations **MUST respect protocol versioning** to ensure compatibility between systems.

## Abstract Model

### Notation

The specification uses an **informal abstract data model notation** to describe the logical structure of RelMon monetary values.

This notation is:

- **Descriptive only**
- **Non-executable**
- **Not intended to be parsed or used as a schema language**

Concrete format implementations of the protocol are defined separately.

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

- **Protocol version**: `protocol` MUST follow the defined ProtocolIdentifier format and reflect the SemVer version of the abstract model..
- **Protocol modes**: `protocol` MAY define its modes affecting the format and the fields of the protocol.
    - **Extended**: Extended mode (`e`) indicates that the `RelMonObject` MUST contain **all** fields defined in the abstract model, including those normally optional. The **only exception** is the `components` field, which MAY be omitted or be empty.
    - **Compact**: this mode defines that the names of the fields of the `RelMonObject` are written in the compact mode. Compact mode affects field identifiers only and MUST NOT alter semantic meaning or numeric behavior. This mode MUST be identified as `c`.
        - `protocol` transforms to `pr`
        - `net` transforms to `n`
        - `tax` transforms to `t`
        - `gross` transforms to `g`
        - `taxRate` transforms to `tr`
        - `unit` transforms to `u`
        - `precision` transforms to `p`
        - `rounding` transforms to `r`
        - `components` transforms to `cs`
    - **Minors**: this mode defines that the fields `net`, `tax` and `gross` of the `RelMonObject` are the smallest units of the currency or asset and MUST be represented as integers. This mode MUST be identified as `m`.
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
    - `hup` value stands for "half-up" and acts as rounding away from zero. Examples: `1.5 => 2`, `2.5 => 3`, `-1.5 => -2`.
    - `hdown` value stands for "half-down" and acts as rounding towards zero. Examples: `1.5 => 1`, `2.5 => 2`, `-1.5 => -1`.
    - `heven` value stands for "half-even" and acts as rounding to the nearest even digit. Examples: `1.5 => 2`, `2.5 => 2`, `3.5 => 4`, `4.5 => 4`, `-1.5 => -2`, `-2.5 => -2`.
    - `up` value stands for "up" and acts as rounding away from zero, discarding any fractional part. Examples: `1.1 => 2`, `-1.1 => -2`.
    - `down` value stands for "down" and acts as rounding towards zero, discarding any fractional part. Examples: `1.9 => 1`, `-1.9 => -1`.
- **Optional metadata**: `comment` in `MonetaryComponent` is purely informational and MUST NOT affect numeric calculations. This field is ALWAYS optional.
- **Format agnostic**: The abstract model does not prescribe JSON, XML, URI, or binary layouts. Concrete representations map abstract fields to specific formats, as defined in separate sections.

#### Validation rules

The RelMon protocol defines the following rules for validating monetary values:

- **Consistency**
    - `gross = net + tax`
    - If `components` are present, the sum of all component `net` values MUST equal the parent `net`, and the sum of all component `tax` values MUST equal the parent `tax`.
- **Data types**
    - `net`, `tax`, and `gross` MUST be decimals unless **Minors mode** (`m`) is active, in which case they MUST be integers.
- **Precision**
    - If `precision` is specified, all decimal values in `net`, `tax`, and `gross` MUST comply with `[maxDigits, scale]`.
- **Rounding**
    - If rounding has been applied to comply with `precision`, the `rounding` field MUST be present and its value MUST be one of the allowed rounding modes.
    - If no rounding has occurred, the `rounding` field MAY be omitted.
    - Allowed values: `hup`, `hdown`, `heven`, `up`, `down`.
- **Optional fields**
    - Any optional fields, such as `unit` or `taxRate`, if present, MUST conform to their abstract type definitions.
- **Components**
    - Each `MonetaryComponent` MUST include `net` and `tax`.
    - The optional `taxRate` in a component MUST comply with the format defined in the abstract model.
    - The optional `comment` in a component MUST NOT affect numeric calculations.
    
#### Signed / Negative values semantics

Monetary values in RelMon may be negative to represent refunds, discounts, credits, or reversals. Implementations MUST handle negative values as follows:

- Fields `net`, `tax` and `gross` MAY be negative (prefixed with `-` symbol) for both decimals or minors.
- Positive values MUST NOT have a prefixed `-`.
- All root-level fields (`net`, `tax` and `gross`) MUST have the same sign.
- Components MAY independently be positive or negative, but each component's `net` and `tax` MUST share the same sign.
- Rounding modes specified in the rounding field apply symmetrically to negative values.

Implementations MUST ensure that arithmetic consistency rules (gross = net + tax and component sums) remain valid even when values are negative.

#### Conformance Profiles

RelMon defines conformance profiles to help implementers achieve predictable interoperability:

- **Core Profile**
    - Includes all **required fields**: `protocol`, `net`, `tax`, `gross`.
    - Optional fields (`taxRate`, `unit`, `precision`, `rounding`, `components`, `comment`) MAY be omitted.
    - Ensures basic interoperability across systems.
- **Extended Profile**
    - Includes **all fields of the abstract model**, including optional ones, except `components`, which MAY be empty or omitted.
    - Provides **full precision**, **reconstructibility**, and **semantic clarity**.
- **Compact Profile**
    - Mirrors the Core or Extended profile, but uses **compact field names** as defined in Compact mode (c).

Implementers MAY declare which profile their system conforms to, ensuring that exchanging systems can verify compatibility and expectations before transfer.