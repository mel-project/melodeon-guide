## Melodeon Expressions

Melodeon is an _expression based_ language, which means emphasis is put on "expressions" that evaluate to a value, rather than "statements" that produce side-effects. Thus, almost every syntax form in the language is an expression.

In this document, we go through the most commonly used expressions and their associated syntax.

### Literals

The simplest expressions are **literals**: representations of individual values rather than calculations. In Melodeon, there are the following kinds of literals:

- **Numeric literals**: numbers like `3` or `12345`. Because they correspond to MelVM's 256-bit integers, they must be unsigned integers between \\(0\\) and \\(2^{256}-1\\).
- **Vector literals**: represented by comma-separated values between square brackets, like `[1, 2, 3]`. Vectors in Melodeon can contain any other values, and the same vector can contain values of different types. For example, `[[1, 2], 3]` is a valid vector of length 2, that contains another length-2 vector and a number.
- **Bytestring literals**. This comes in two flavors:
  - **ASCII literals**: Printable, ASCII characters within double-quotes: `"hello world"`. This represents a bytestring that contains the ASCII representation of that string.
  - **Hex literals**: Hexadecimal characters in double-quotes following `x`: `x"deadbeef"`. This represents a bytestring that contains those literal bytes, here `de ad be ef`.
- **Struct literals**, like `Point{x: 2, y: 3}`. This represents _structs_, which we will talk about later (TODO).

### Numeric operators

Melodeon has a full set of operators that operate on . In order from _tightest_ to _loosest_ precedence:

**Arithmetic operators**:

| Operator | Meaning               | Precedence |
| -------- | --------------------- | ---------- |
| `x * y`  | `x` multiplied by `y` | 1          |
| `x / y`  | `x` divided by `y`    | 1          |
| `x % y`  | `x` modulo `y`        | 1          |
| `x + y`  | `x` added to `y`      | 2          |
| `x - y`  | `x` minus `y`         | 2          |

**Bitwise operators**

**Logical operators**

| Operator   | Meaning                                             | Precedence |
| ---------- | --------------------------------------------------- | ---------- |
| `!x`       | not `x` (`1` if `x` is falsy, `0` if `x` is truthy) | 0          |
| `x \|\| y` | `x` or `y` (`x` if `x` is truthy, otherwise `y`)    | 4          |
| `x && y`   | `x` and `y` (`y` if `x` is truthy, otherwise `x`)   | 4          |
