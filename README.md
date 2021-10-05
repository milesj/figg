# Figg

## Syntax

## Types

The following types are acceptable values to assign to both properties and variables.

### Booleans

Booleans are a primitype type for declaring a binary true/false (or on/off) value. The value is represented by the `true` and `false` keywords (lowercase only). 

```
propTruthy = true
propFalsy = false
```

### Numbers

Numbers are a [double-precision 64-bit binary format (IEEE 754)](https://en.wikipedia.org/wiki/Floating-point_arithmetic) value, 
like `double` in Java or C#. This means it can represent fractional values, but there are some limits to what it can store. 
A number only keeps about 17 decimal places of precision, with the largest value a number can hold being 1.8E308.

```
propInt = 123
propFloat = 456.78
```

For large numbers and better readability, the `_` character can be used to group the digits per thousand.

```
propInt = 123_456_789 # 123,456,789
```
