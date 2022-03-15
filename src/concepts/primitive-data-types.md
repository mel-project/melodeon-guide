## Primitive Data Types

Now that you can write expressions and define functions, it is time to
understand the primitive data types so that you can construct full programs.

### Numbers

Melodeon is a language for blockchain execution. Blockchain VMs tend to work in
an unsigned 256-bit address space. While this is not exactly the case in MelVM,
the U256 is still a fundamental data type. In Melodeon we call this number the
`Nat`, for natural number (non-negative).

Melodeon has no native concept of signed numbers. Rather, these are implemented
as user libraries. Most blockchain logic does not require signed numbers. But
if you are wondering why signed numbers are not native, see [Occurance
typing]().

### Literals

- ASCII literals - "hello world" - represent ASCII strings stored in compact
  bytes
- Hex literals - x"deadbeef" - Hex string stored in compact bytes
- Vector literals - [1, 2, 3] - An instantiation of a vector

### Vectors

Vectors are typical and as expected with one caveat; their length is part of
their type. That is, a vector of 3 Nats, `[Nat; 3]` is a different type from
`[Nat; 4]`.

Having length information at the type level makes it much easier to write
secure programs. For instance, an out-of-bounds index will be a type error!
More on the magic of length-typed vectors in the section on [Type Generics]().

Notice the notation above, `[Nat; 3]` means 3 Nats. An equivalent notation is
`[Nat Nat Nat]`. Vectors can also be heterogenous in their types. For instance,
`[Nat Bool Nat]` is a valid vectors.

### Dynamic Vectors

Dynamic vectors are useful for those especially difficult cases to define.
These are vectors of unknown length. Indexing into them is considered unsafe.
Their notation simply omits a length `[Nat;]`.

### Byte Arrays

Byte arrays a special type; `%[32]` (32 bytes). They can also be dynamic;
`%[]`.

While `[Bool;]` may function perfectly well as a byte array, under the hood
each Bool will be represented with 256 bits. A byte array is a compact,
contiguous byte representation. It is the more efficient choice.
