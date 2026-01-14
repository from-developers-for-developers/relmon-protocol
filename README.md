# Reliable Monetary data protocol (RelMon protocol)

## Abstract Model

### Notation

This specification uses an **informal abstract data model notation** to describe the logical structure of RelMon monetary values.

This notation is:

- **Descriptive only**
- **Non-executable**
- **Not intended to be parsed or used as a schema language**

Concrete representations/implementations of the protocol are defined separately.

**Legend**

- `?` indicates an optional field  
- `[]` indicates an ordered list  
- Capitalized identifiers denote abstract types  

---

```
RelMonObject :=
{
    protocol: Text
    net: Decimal | Integer
    tax: Decimal | Integer
    gross: Decimal | Integer

    taxRate?: Decimal(5, 2)
    currency?: ISO4217Code
    precision?: DecimalPrecision
    rounding?: RoundingMode
    components?: MonetaryComponent[]
}

ProtocolIdentifier := Text
DecimalPrecision := [maxDigits: Integer, scale: Integer]
RoundingMode := "hup" | "hdown" | "heven" | "up" | "down"
MonetaryComponent :=
{
    net: Decimal | Integer
    tax: Decimal | Integer
    taxRate: Decimal(5, 2)
    comment?: Text
}
```