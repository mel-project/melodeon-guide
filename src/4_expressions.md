# Expressions and Operators

Because it is a functional language without side effects, all useful computation in Melodeon produces a value. Something that evaluates to a value is an expression: so, **everything is an expression in Melodeon**.

## Operators

Literals representing [values](./simple_types.md) are the simplest Melodeon expressions. The following built-in operators provide building-blocks for more complex expressions. As explained below, the operators are grouped by input. In the `Precedence` field, a larger value signifies a higher order of precedence ("tighter" binding).

Also, try out the operators in the REPL as you read along!

### `Nat` operators

| Operator | Meaning                        | Precedence |
| -------- | ------------------------------ | ---------- |
| `x + y`  | `x` plus `y`                   | 5          |
| `x - y`  | `x` minus `y`                  | 5          |
| `x * y`  | `x` times `y`                  | 6          |
| `x ** y` | `x` raised to the power of `y` | 6          |
| `x / y`  | `x` divided by `y`             | 6          |
| `x % y`  | `x` modulo `y`                 | 6          |
| `>`      | greater than                   | 3          |
| `>= `    | greater than or equals to      | 3          |
| `<`      | less than                      | 3          |
| `<=`     | less than or equals to         | 3          |
| `==`     | is equal to                    | 3          |

`Nat` operators take natural numbers. For example, the compiler throws a type error if you try to compare two vectors with `<=`:

```
melorun> [1] <= [1, 2]
Melodeon compilation failed
~/.melorun.melo.tmp:3: error: expected a subtype of Nat, got [{1}] instead
    [1] <= [1, 2]
^~~~~~~~~~~~~

```

### Logical operators

| Operator                      | Meaning                      | Precedence |
| ----------------------------- | ---------------------------- | ---------- |
| `!x`                          | not `x`                      | 7          |
| <code>x &#124;&#124; y</code> | short-circuiting `x` or `y`  | 1          |
| `x && y`                      | short-circuiting `x` and `y` | 1          |

Logical operators take _any value_ as input, interpreting their inputs as booleans. As mentioned in the section on [values](simple_types.md), `0` is "falsy" and every other value is "truthy".

Examples:

```
melorun> 0 || 1
1

melorun> 0 && 1
0

melorun> 1 && 2
2

melorun> !3
0
```

As you might have noticed from the examples, `&&` and `||` are implemented with _short-circuit evaluation_.

`x && y` is equivalent to

```
if x then y else x
```

and `x || y` is

```
if x then x else y
```

Short-circuiting logical operators enable neat JavaScript-style idioms in Melodeon. For instance, suppose we want to multiply `x` by `2` if it's a `Nat`, but `x`'s most specific type is `Nat | %[8]` (in other words, we only know `x` is either a `Nat` or a byte string of length 8). We can express this logic with

```
x is Nat && x * 2
```

In this expression, `x is Nat` is a boolean expression that evaluates to true if `x` is a member of type `Nat` (you can read about the `is` operator in [Advanced Types](advanced_types.md)). Because `&&` has short-circuit logic, if `x is Nat`, then we'll get the result of `x * 2`. If `x is Nat` is false, then `x * 2` will never be evaluated, so we won't ever get a type error.

**_Exercise 4.1._** What does `0 || ([1] && "hello")` evaluate to?

### Bitwise operators

| Operator              | Meaning     | Precedence |
| --------------------- | ----------- | ---------- |
| `<<`                  | left shift  | 4          |
| `>>`                  | right shift | 4          |
| <code> &#124; </code> | bitwise or  | 2          |
| `&`                   | bitwise and | 2          |
| `^`                   | bitwise xor | 2          |
| `~`                   | bitwise not | 7          |

Bitwise operators take `Nat`'s as input, interpreting them as bit strings. Here are a few examples:

```
melorun> 1 << 2
4

melorun> 8 >> 2
2

melorun> 1 ^ 3
2

melorun> 6 & 4
4

melorun> 2 | 4
6

```

Try them out in the REPL!

### Vector operators

| Operator    | Meaning                                                         | Precedence |
| ----------- | --------------------------------------------------------------- | ---------- |
| `v ++ w`    | append two vectors                                              | 5          |
| `v[i]`      | reference: returns element at `i`th index of `v`                | 8          |
| `v[i..j]`   | functional slice: returns a copy of `v` from index `i` to `j-1` | 8          |
| `v[i => x]` | functional update: returns a copy of `v` with `x` at index `i`  | 8          |

Vector operators take vector inputs. All vectors are 0-indexed. Also, all vector operators return newly created vectors or elements: nothing is ever mutated.

<!--
For example, in

```
melorun> vec = [1, 2, 3]

melorun> vec[1]
2

```

the 2 that is returned is a different copy from the `2` in `vec`. In

```
melorun> v = [1, 2]

melorun> w = [3]

melorun> v ++ w
[1, 2, 3]

```

the returned vector `[1, 2, 3]` also does not share any memory with `v` or `w`. -->

#### Slice

The slice operator `v[i..j]` returns a new vector that contains the elements of `v` between indices `i` and `j`, including `i` but _not_ including `j`. For example:

```
melorun> let v = [1, 2, 3] in v[0..1]
[1]

melorun> let v = [1, 2, 3] in v[1..3]
[2, 3]

melorun> let v = [1, 2, 3] in v[0..3]
[1, 2, 3]
```

#### Update

Here are a few examples of update. They emphasizes that update does _not_ mutate the vector:

```
melorun> ([1, 2, 3] :: [Nat; 3])[1 => 3]
[1, 3, 3]

melorun> ([1, 2, 3] :: [Nat; 3])[2 => 5]
[1, 2, 5]

melorun> let v = [1, 2, 3] in (v :: [Nat; 3])[1 => 10] ++ v
[1, 10, 3, 1, 2, 3]
```

**_Exercise 4.2._** Define `vec` as follows:

```
vec = ["melo", "deon"]
```

What does this expression evaluate to?

```
vec[1 => "haha"] ++ vec[1..2]
```

### Structure operators

| Operator | Meaning                                                                    | Precedence |
| -------- | -------------------------------------------------------------------------- | ---------- |
| `s.x`    | field access: returns the value associated with field `x` of structure `s` | 8          |

This operator is analogous to vector reference. It's included here for completeness; learn about structures and field access in [`Advanced Types`](advanced_types.md) and `let` expressions below.

For example, given a file containing this definition:

```
struct Point {
    x: Nat,
    y: Nat
}
```

we have:

```
melorun> let p = Point{ x: 1, y: 2 } in p.x
1

melorun> let p = Point{ x: 1, y: 2 } in p.y
2
```

### Byte string operators

<!-- As you've probably noticed, `==` is an overloaded operator (that means it's multiple functions with the same name).  -->

In Melodeon, `++` is an overloaded operator (that means it's multiple functions with the same name). It also works for byte strings:

| Operator | Meaning                  | Precedence |
| -------- | ------------------------ | ---------- |
| `s ++ t` | concatenates `s` and `t` | 5          |

<!-- ***Exercise 4.3.*** Is `1 == [2]` a valid Melodeon expression? -->

**_Exercise 4.3._** Is `"1" ++ [2]` a valid Melodeon expression?

## If expressions

Melodeon has an ML-like branching expression:

```
if x then y else z
```

This returns `y` if x is "truthy" (i.e. not `0`), otherwise `z`. The inferred type of an `if` expression is the inferred type of `y` union the type of `z`. For instance,

```
melorun> if 35 then 1 else [10]
- : {1} | [{10}]
1

```

**_Exercise 4.4._** What is the type of `if 35 then 35 else "melodeon"`?

## Let expressions

Variable bindings in Melodeon are also expressions. They're similar to variable bindings in ML languages:

```
let x = expression_1 in
expression_2
```

binds variable `x` to `expression_1` in the scope of `expression_2`. Variable names must begin with a lower-case letter and contain only alphanumeric characters. Idiomatically, variables are written in `snake_case`. Here is an example:

```
melorun> let x = 1 in x + 2
3
```

Because `let` bindings are themselves expressions, they can be chained. For instance,

```
melorun> let x = 1 in let y = 2 in let z = 3 in x * y + z
5
```

You can also define multiple variables in one `let` expression, using this syntax:

```
let var_1 = expr_1,
    var_2 = expr_2,
    ...,
    var_n = expr_n
in
    expression
```

A concrete example:

```
melorun> let x = 1, y = 2 in x + y
3
```

## Iteration expressions

Melodeon has three _bounded_ iteration constructs: `for`, `fold`, and `loop`. Each has a _stopping value_ that determines how many iterations are executed. This is the vector length in `for` and `fold` and the iteration number in `loop`. To prevent unbounded computation (whose fees cannot be pre-determined!), this stopping value must be known at compile time—that is, it must be a constant expression.

A constant expression is either a number whose values is known at compile time, or such numbers combined with `+` and `*`. Numeric constants like `1` and `5` are constant expressions; so are `3 + 5` and `1 * 4`. Constant generics are the final kind of constant expressions: learn about them in [Advanced Types](advanced_types.md)!

Melodeon does not currently support any form of recursion. We are considering adding provably bounded recursion similar to `Fixpoint` in Coq in the future.

### `for`

`for` applies an `expression` to every member of a `vector`. It's the equivalent of `map` in other functional languages. `for` has two equivalent forms:

```
for <variable> in <vector> collect <expression>
```

and also

```
[<expression> for <x> in <vector>]
```

which may be familiar from Python.

The `collect` keyword in the first syntax and the square brackets in the second indicate that the output of `for` is always a vector.

Examples:

```
melorun> for x in [1, 2, 3] collect x + 1
[2, 3, 4]

melorun> x && 0 for x in ["melo", "deon"]
[0, 0]

melorun> for n in [1, 2, 3] collect "4"
["4", "4", "4"]
```

`for` only works on fixed-length vectors, as we explained above. For example, if we [cast](advanced_types.md) the type of `[1, 2, 3]` in the first example above to a dynamic-length vector, then it will fail to compile.

```
melorun> for x in ([1, 2, 3] :: [Nat;]) collect x + 1
Melodeon compilation failed
~/.melorun.melo.tmp:15: error: list comprehension needs a list with a definite length, got [Nat;] instead
	for x in ([1, 2, 3] :: [Nat;]) collect x+1
	^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

**_Exercise 4.5._** Write a `for` expression, in either syntax, that squares each element of the vector `[1, 10, 100]`.

### fold: WIP

Fold takes a vector, a base value, and a "collector" expression. It applies the expression to each element of the vector with the base value, updating the base value as it proceeds. Its syntax is

```
for x in vec fold accum = base with expr
```

where for every element of `vec`, `fold` computes `expr` and sets `accum` to be the new `expr`, and `base` is the starting value of `accum`. It's typically used to combine the elements of a vector.

Examples:

```
melorun> for n in [1,2,3] fold y = 0 :: Nat with y + x
6

melorun> for x in [1,2,3] fold accum = 0 :: Nat with 5
5

melorun> for v in [[1], [2], [3]] fold accum = [] with accum ++ v
TODO!
```

**WIP: casting rules for fold**

**_Exercise 4.6._** on casting: WIP

### `loop`: pseudo-imperative loops

```
loop n do
    body
return ret
```

executes `body` `n` times and returns `ret`, where `body` is a chain of 0 or more _pseudo-mutations_ of the form `variable_name <- expression` separated by semicolons, and `n` is a constant expression.

A pseudo-mutation is variable mutation restricted to the scope of the loop:

```
x <- new_value
```

sets a special _copy_ of a previously defined variable `x` to `new_value`. This copy is in scope only in the loop body. So under the hood, this expression

```
let x = 0 in
loop 5 do
    x <- x+1
return x
```

is really

```
let x = 0 in
let y = 0 in
loop 5 do
    y <- y + 1
return y
```

Pseudo-mutations are only valid inside loop bodies.

Examples:

```
melorun> let x = 0 :: Nat in
            loop 5 do
            x <- x+1
            return x
5

melorun> let x = 0 :: Nat in
            loop 5 do
                x <- x + 1;
                x <- x + 1
                return x
10
```

> **Note**: `:: Nat` here casts `x` to type `Nat`. [Upcasting](advanced_types.md) is necessary in these examples because the compiler automatically assigns type `{0}` to `x` in both cases (read about Melodeon's type inference in [Simple Types](simple_types.md)), but clearly the new values assigned to `x` do not belong to `{0}`. In general, mutated variables often need to be upcasted before mutation to avoid a type error.

To reiterate, the iteration number following the keyword `loop` must be a constant expression. An example of a non-constant expression is the argument to a function. To see this fail in action, try running the REPL with a `.melo` file containing this function:

```
def f(n: Nat) =
    let x = 0 in
    loop n do
        x <- x + 1
    return x
```

Here's what we got:

```
❯ melorun -i loop.melo
Error: Melodeon compilation failed
~/loop.melo:4: error: expected const_mult_expr
	    loop n do
	         ^

```

**_Exercise 4.7._** Using a loop, write an expression that produces 4 appended copies of the vector `[1, 2]` (evaluating to `[1, 2, 1, 2, 1, 2, 1, 2])`.

## Assert and Fail

`fail!` crashes the program.

Examples:

```
melorun> fail!
MelVM exection failed

melorun> let x = 1 in (if x is %[] then "great!" else fail!)
MelVM execution failed
```

```
assert! expr1 in expr2
```

returns `expr2` if `expr1` evaluates to true, otherwise `fail!`. It's equivalent to

```
if expr1 then expr2 else fail!
```

Examples:

```
melorun> assert! 1==1 in 3
3

melorun> assert! 1 == 2 in 3
MelVM execution failed

```
