# RelMon JSON implementation

This document defines the official JSON implementation of the RelMon abstract model.

If this document conflicts with [README.md](README.md), the core specification in `README.md` governs protocol semantics.

## Scope

- All modes of the protocol are supported.
- The name of the field containing a RelMon object MAY be arbitrary.

## Mapping

The JSON implementation maps the abstract RelMon fields directly to JSON object properties.

- In standard mode, property names follow the abstract model field names.
- In compact mode (`c`), property names MUST use the compact identifiers defined by the core specification.
- In minors mode (`m`), `net`, `gross`, and `tax` MUST be represented as integers.

## Examples

The following example shows a RelMon object at DL3 with `components` (`scope = "c"`).

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

Another example (DL1) in compact mode.

```json
{"p": "relmon@1.0.0/1:c", "n": "100.00", "tr": "21.00"}
```
