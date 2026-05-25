# RelMon XML implementation

This document defines the official XML implementation of the RelMon abstract model.

If this document conflicts with [README.md](README.md), the core specification in `README.md` governs protocol semantics.

## Scope

- All modes of the protocol are supported.
- The root element name MAY be arbitrary, for example `<Money>` or `<Price>`.

## Mapping

The XML implementation maps the abstract RelMon fields directly to XML elements.

- In standard mode, element names follow the abstract model field names.
- In compact mode (`c`), element names MUST use the compact identifiers defined by the core specification.
- In minors mode (`m`), `net`, `gross`, and `tax` MUST be represented as integers.

## Examples

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

Another example (DL3) in compact and minors mode, with the OPTIONAL `unit` field.

```xml
<RelMon>
    <p>relmon@1.0.0/3:c.m</p>
    <n>10000</n>
    <g>12100</g>
    <t>2100</t>
    <u>EUR</u>
</RelMon>
```
