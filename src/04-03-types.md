# Types

## Structural typing

Melodeon is a statically typed language, like Rust, Java, or C. Unlike those languages, though it is _structurally typed_ in a way somewhat reminiscent of gradually-typed languages like TypeScript or Typed Racket.

Structural typing boils down to two principles:

- **Values are prior to type names**: In Melodeon, the _value_ that an expression evaluates to can be discussed without reference to any type names. For example, the expression `[1, 2, 3]` is fully described as a "vector with the numbers 1, 2, and 3". Static types and their names are defined in terms of these values, not the other way around.
- **Types are sets of values**: For example, the type name `Nat` simply represents the set of all numbers (i.e. 256-bit unsigned integers), nothing more and nothing less. Other types include `{3..100}` (all numbers between 3 and 100, inclusive), `[Nat, Nat | [{0}, {1}]]` (vectors where the first element is a number and the second element is either a number or the vector `[0, 1]`), etc.

Thus, unlike in most
