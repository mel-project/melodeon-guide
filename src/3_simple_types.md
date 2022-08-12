# Simple Types

The most important thing about types in Melodeon is that **types are sets of values**. Here are a few example types:

- `{1}`: the set containing the single number `1`
- `[{1}, {2}, {3}]`: the set containing the single vector `[1, 2, 3]`
- `Nat`: the set of all natural numbers
- `Any`: all values belong to the type `Any`
- `Nothing`: no values belong to the type `Nothing`

An immediate consequence is that _every value belongs to an infinite number of types_. This means that it doesn't make sense to say a type `T` is _the_ type of a value `x`, only that `T` is a type of `x` or that `T` contains `x`.

Thus, let's begin our discussion of Melodeon's types with the different kinds of possible values.

## Values

Melodeon has three kinds of values:

1. **Naturals**: any integer between `0` and `2^256-1`, inclusive

2. **Vectors**: any fixed-length list. An example is `[1, 2, 3]`. Members of a vector can be of different types, like in `[1, 2, [1, 2]]`.

3. **Byte strings**: any string of bytes. A byte string doesn't have to be a valid string or be printable. There are two kinds of byte strings: ASCII strings and hex strings.
   1. **ASCII strings** are enclosed by double quotes, like `"melodeon"`
   2. **Hex strings** are enclosed by `x".."`, like `x"deadbeef"`

There are _no_ negative integers, rationals, floating points, or any other sorts of numbers in the core Melodeon language. There are also _no_ dedicated boolean values. In contexts that call for booleans, 0 has the semantic meaning of false, while all other values have the semantic meaning of truth. Hence, any value can be used where a boolean is expected.

Examples:

```
melorun> 1 == 2
0

melorun> 1 == 1
1

melorun> 3 && 4
4

melorun> if 35 then 1 else 0
1
```

Note that `Bool` is an alias for `{0..1}`.

## Type constructors

Types are sets of values, but not every set of values is a type in Melodeon. For instance, the type containing all values except 1 is not a Melodeon type. Melodeon types are what can be expressed by the following type constructors, or by a combination of them with type union. We'll denote "`x` is within type `T`" as `x : T` below.

- **Naturals**

  - `Nat`: all natural numbers modulo `2^256`

  - `{n}`: the type containing a single `Nat` `n`.

    Examples:

    ```
    1 : {1}

    2 : {2}

    34 : {34}
    ```

  - `{m..n}`: the type of the range of `Nat`'s from `m` to `n`, inclusive.

    Examples:

    ```
    1, 2, 3, 4 : {1..4}

    5, 6, 7 : {5..7}
    ```

- **Vectors**

  - `[T_1, T_2, ..., T_n]`: the type containing all vectors whose first element is a member of type `T_1`, second element is a member of type `T_2`, and so on.

    Examples:

    ```
    [1, 2, 3] : [{1..2}, {2..3}, {3..4}]

    [1, 3, [5]] : [{1}, {2..3}, [{4..6}]]
    ```

  - `[T; n]` is an abbreviation for the type of a vector of length `n` whose members are all of type `T`. A concrete example is `[Nat; 2]`, the set of all vectors of natural numbers of length 2.

  - `[T;]` is the type containing all vectors of _any_ length, whose members are all a member of type `T`. Vectors of arbitrary length are called **dynamic vectors**.

    Examples: `[Nat;]` is the set of all vectors of natural numbers. `[Any;]` is the set of all possible vectors.

- **Byte strings**

  - `%[n]` is the type including all byte strings of length `n`.
    Examples:

    ```
    "melodeon" : %[8]

    x"deadbeef" : %[4]
    ```

    (`x"deadbeef` is 4 hexadecimals!) Melodeon does not provide syntax for specifying types of particular byte strings, since such specificity for byte string types is rarely useful.

  - `%[]` is the type including all byte strings. It can also be thought of as the type of **dynamic byte strings**.

<!-- Note that Melodeon's type syntax does not make specifying every type easy. For example, there is no easy syntax for specifying something like the set or type of all vectors of length 50 whose first element belongs to `Nat` (it's possible: you just need to type `Any` 49 times in `[Nat, Any, ..., Any]`).
 -->

## Union types

The union of two Melodeon types is also a type. For example, `{1} | {3}` is a type that contains two elements `1` and `3` (the pipe symbol means type union in Melodeon). `{1} | [{3}]` is another type that contains elements `1` and `[3]`.

> **Note**: There are, however, no intersection or complement types, or any other set operations on types. This is because they would make it possible to express all of Boolean algebra in the type system, making typechecking equivalent to the [Boolean satisfiability problem](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem). As that problem is NP-complete, type-checking Melodeon programs will also be NP-complete (i.e. very hard). Back when we had type intersection in Melodeon, type-checking small programs took several minutes!

**_Exercise 3.1._** Write down a union type that contains exactly 3 elements.

## Subtyping

As you probably guessed, subtyping in Melodeon is the subset relation. That is, type `U` is a subtype of type `T` means that the set of values described by `U` is a subset of those described by `T`. For example,

```
{1} <: {1..5} <: Nat <: Nat | %[10]
```

where `<:` is notation for "is a subtype of" (it's not a part of Melodeon syntax).

Every type is a subtype of itself, `Nothing` is a subtype of every possible type, and every type is a subtype of `Any`.

## Type inference

Melodeon only performs _local_ type inference. This means the type checker only infers the type of a value from its _subexpressions_, not from how it will be used in the future. A consequence is that the types of all function arguments must be manually annotated—the type checker cannot infer the type of a function argument based on how it will be used in the function body.

But if all values belong to an infinite number of types, what type does type inference assign to values? The answer is: _the smallest type the compiler can determine given the context_. This is the natural choice because it gives the most information about the value.

Here are a few examples:

```
melorun> 1
- : Nat [more specifically: {1}]

melorun> [1, 2]
- : [{1}, {2}]

melorun> "melodeon"
- : %[8]

melorun> if 1 == 2 then 3 else 4
- : Nat [more specifically: {3} | {4}]

melorun> [1, x"deadbeef", 3]
- : [{1}, %[4], {3}]
```

To reiterate, every value without a type annotation is assigned the most specific type that the compiler can determine from the immediate context.

**_Exercise 3.2._** What type will the Melodeon compiler ascribe to the standalone expression

```
[x"f00d", 517, []]
```

## Alias types

You can define aliases for existing types like this:

```
type Alias = existing_type
```

For example, we can rename `Nat` to `Naturals`:

```
type Naturals = Nat
```

The standard library implements Booleans as type aliases:

```
type False = {0}
type True = {1}
type Bool = True | False
```

Like any other type name, type aliases _must_ start with an upper-case letter. Generally, type-level constructs (type alias and structure definitions) must be written in `UpperCamelCase`, while value-level constructs (variable and function names) should be `snake_case`. This is the same as [Rust's naming convention](https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md).
