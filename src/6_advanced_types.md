# Advanced Types

In this section, we'll cover the somewhat more involved but definitely must-know aspects of Melodeon's type system. These are type generics, constant generics, casting, and structures.

## Generics

Melodeon provides generics in function definitions (see [Rust's explanation](https://doc.rust-lang.org/book/ch10-01-syntax.html) for an introduction). They are defined with a familiar syntax, where type variables are placed in angle brackets (`<..>`) after the function name. Here are a few examples:

```
def poly_equals<T>(x: T, y: T) =
    x == y


def compare_if_nat<U>(x: U, y: Nat) =
    if x is Nat then x == y else fail!
```

The purpose of generics is to make a function's output type depend on its input type. For instance, consider the standard library's definition of `vref` (vector reference) versus a similar function where `x` has the concrete type `[Any;]` rather than the generic type `T`:

```
# returns the nth element of x if it exists, otherwise 0

def vref<T>(x: [T;], n: Nat): T | False =
    if n < vlen(x) then
        unsafe_vref(x, n)
    else
        0

def any_vref(x: [Any;], n: Nat): Any | False =
    if n < vlen(x) then
        unsafe_vref(x, n)
    else
        0
```

Both functions accept vectors with elements of any type. However, from the return type annotations, we see that `vref` returns a value of type `T | False` (remember `False` is defined as an alias for `{0}` in the standard library), while `any_vref` returns an `Any | False`. This makes a big difference when you try to use the output:

```
melorun> vref([1, 2], 1) + 2
4

melorun> any_vref([1, 2], 1) + 2
Melodeon compilation failed
/~/.melorun.melo.tmp:58: error: expected a subtype of Nat, got Any instead
	any_vref([1, 2], 1) + 2
	^~~~~~~~~~~~~~~~~~~~~~~
```

`any_vref`'s return type `Any` is so uselessly broad that the compiler can't tell its output belongs to type `Nat`. So, be sure to use generics when you want your function to accept arguments of different types _and_ have a meaningfully specific return type.

**_Exercise 6.1._** (TBD)

## Const generics

Now, try writing a function that adds `1` to every member of a `Nat` vector. It should work on `Nat` vectors of any length. Here's a first attempt:

```
def add1(vec: [Nat;]) =
    [x + 1 for x in vec]
```

Let's try it out in the REPL...

```
❯ melorun -i demo.melo
Error: Melodeon compilation failed
/~/demo.melo:2: error: list comprehension needs a list with a definite length, got [Nat;] instead
	    [x + 1 for x in vec]
	    ^~~~~~~~~~~~~~~~~~~~
```

Ah right. Melodeon won't iterate over any vector without a known constant length. How should we proceed?

We need something that parameterizes over vector lengths and other _constants_, not just types. This is are called **const generics**. In Melodeon, constant generic parameters are variables starting with a `$`. We can use them to define an `add1` function that compiles:

```
def add1<$n>(vec: [Nat;$n]) =
    [x + 1 for x in vec]
```

```
❯ melorun -i demo.melo

melorun> add1([1, 2, 3])
[2, 3, 4]
```

Here are two more examples of looping with constant generics:

```
# returns the sum of all the members of a Nat vector

def members_sum_loop<$n>(vec: [Nat; $n]) =
    let x = 0 :: Nat in
    let i = 0 :: Nat in
    loop $n do
    	x <- x + vec[i];
    	i <- i + 1
    	return x

def members_sum_fold<$n>(vec: [Nat; $n]) =
    for x in vec fold
    	accum = 0 :: Nat
    	with accum + x
```

```
melorun> members_sum_fold([1,2,3,4])
10

melorun> members_sum_loop([1,2,3,4])
10
```

**_Exercise 6.2._** Write a function that swaps the `i`th and `j`th elements of a vector. It should work on vectors of arbitrary length.

**_Exercise 6.3._** Will this function compile? If not, why not, and rewrite it so that it compiles.

```
def f(vec: [Nat;]) =
    for i in vec fold accum = 0 :: Nat with accum + i
```

### Statically enforcing function argument properties

We've already seen that constant generics are indispensable for looping over variable-length vectors or byte strings. You can also use them to statically enforce certain properties of function arguments. This is very expressive: for example, the function

```
def f<$n>(x: {2*$n}) = x * 2
```

has the set of all even `Nat`'s as its input type. The compiler enforces this restriction:

```
melorun> f(2)
4

melorun> f(1)
Melodeon compilation failed
/~/.melorun.melo.tmp:8: error: cannot fill type variables in argument type {(2 * $n)}, given concrete {1}
	f(1)
	^~~~
```

Given an input `x` to the function `f`, the compiler tries to find a concrete `$n` that solves `x = 2 * $n`. If it succeeds, the compiler replaces `$n` with that specific value during compilation, getting rid of the constant generic. This way, all constant generics are in fact constant expressions at run-time.

Another example of input specification using constant generics:

```
# returns the first member of a *non-empty* vector

def head<$n>(vec: [Nat; $n + 1]) =
    vec[0]
```

Specifying the length of `head` as `$n + 1` restricts its inputs to non-empty vectors, because `1` is the smallest `m` for which there exists a `Nat` `$n` such that `$n + 1 = m` (alternatively, there is no natural number that satisfies the equation `$n + 1 = 0`). We see that the code will not compile with an empty vector:

```
melorun> head([])
Melodeon compilation failed
~/.melorun.melo.tmp:18: error: cannot fill type variables in argument type [Nat; ($n + 1)], given concrete []
	head([])
	^~~~~~~~

melorun> head([1])
1
```

**_Exercise 6.4._** Write an identity function that statically enforces its input to be an odd-length byte string.

**_Exercise 6.5._** Implement [bubble sort](https://en.m.wikipedia.org/wiki/Bubble_sort) in Melodeon. You might find `swap` from Exercise 6.2 handy.

In the above examples, the compiler tells us it couldn't find a natural number `$n` that satisfies `1 = 2 * $n` or `2 = 4 * $n` or `0 = $n + 1`, because none exists!

### Manually specifying type variables

In more complicated cases, the compiler might just not be smart enough to figure out the correct types on its own. Here's an example:

```
def f<$n>(x: {2*$n}) = x * 2

def g<$n>(x: {4*$n}) = x * 2

def h<$m>(x: {$m}) = g(f(x*2))
```

```
❯ melorun -i demo.melo
Error: Melodeon compilation failed
~/demo.melo:5: error: cannot fill type variables in argument type {(2 * $n)}, given concrete {($m * 2)}
	 def h<$m>(x: {$m}) = g(f(x*2))
	                        ^~~~
```

We can help out by manually specifying the type variables when calling `g` inside `h`:

```
def h<$m>(x: {$m}) = g<$n = $m>(f<$n = $m>(x*2))
```

### Syntax

Finally, if you include both constant generics and type generics in the same function definition, the constant generics must be written before the type variables in the angle brackets.

Examples:

```

❯ melorun -i demo.melo
Error: Melodeon compilation failed
/home/thisbefruit/demo.melo:42: error: expected type_name
def app_type<T,$n>(x: [T; $n], y: T) =
^

```

## Occurrence typing and `is` expressions

`is` expressions allow us to check whether a variable is a member of a certain type:

```

melorun> let x = 1 in x is Nat
1

melorun> let x = [1] in (if x is Nat then x else 0)
0

```

Note that `is` expressions do not work on literals (non-variables):

```

melorun> 1 is Nat
Melodeon compilation failed
~/.melorun.melo.tmp:3: error: expected EOI, add, sub, append, exp, mul, div, modulo, lor, land, lshift, rshift, bor, band, bxor, equal, gt, ge, lt, le, call_args, field_access, vector_ref, vector_slice, vector_update, as_type, or into_type
1 is Nat
^

```

Importantly, `is` expressions serve as _type tests_ for [occurrence typing](https://www.irif.fr/~gc/papers/popl22.pdfhttps://www.irif.fr/~gc/papers/popl22.pdf) in Melodeon. This is a typing technique that uses type tests to assign more specific types to the different occurrences of a variable in different branches of an `if` expression. Example:

```

# doubles vec[0] if vec has length 1, otherwise 0

def double_or_0(vec: [Nat;]) =
    if vec is [Nat; 1] then
        vec[0] * 2
    else
        0

melorun> double_or_0([1])
2

melorun> double_or_0([1, 2])
0

```

The program will not compile without the occurrence typing, since the compiler can't check that it's safe to access the 0th index of `vec`:

```

def double(x: [Nat;]) =
    x[0] * 2

❯ melorun -i demo.melo
Error: Melodeon compilation failed
error: type [Nat;] cannot be indexed into
x[0] * 2
^~~~

```

Another example:

```

melorun> let x = (if 1 == 2 then "contradiction!" else 0) in x + 2
Melodeon compilation failed
error: expected a subtype of Nat, got %[14] | {0} instead
let x = (if 1 == 2 then "contradiction!" else 0) in x + 2
^~~~~

```

[Recall](simple_types.md) that an `if` expression is typed as the union of the types of its branches. This means that `x` has the type `%[14] | {0}` (i.e., byte strings of length 14 union `{0}`). That obviously is not a valid input type to `+`! We can refine the type of `x` using occurrence typing:

```

melorun> let x = (if 1 == 2 then "contradiction!" else 0) in
(if x is Nat then x + 2 else fail!)
2

```

Currently, the only type test supported in Melodeon is the `is` expression. We look forward to adding support for user-defined type tests (like [these](https://docs.racket-lang.org/ts-guide/occurrence-typing.html) of Typed Racket) in future releases.

**_Exercise 6.6._** Fill in the body of the function

```

def f(x: Any) = ...

```

so that it returns `"bingo!"` if `x` is a byte string of length `5` and `"keep trying!"` otherwise.

## Casting

In the section on [type inference](simple_types.md), we mentioned that the Melodeon compiler assigns the most specific type it can to all values without type annotations. We can manually change the type ascription of any value or expression by safely casting it to a supertype or `unsafe`ly casting it to a subtype of its current type.

### Up-casting

We can change the type assignment for a value to any supertype of its current assignment using the double colon operator, `::`. Here are a few examples of successful super-casts:

```

melorun> 1 :: {1}
1

melorun> 1 :: {1..3}
1

melorun> 1 :: {1..3} :: Nat
1

```

You cannot cast to subtypes of the current type using `::`:

```

melorun> 1 :: Nat :: {1..3}
Melodeon compilation failed
~/.melorun.melo.tmp:3: error: cannot cast type Nat to non-supertype {1..3}
1 :: Nat :: {1..3}
^~~~~~~~~~~~~~~~~~

```

Here `1 :: Nat` has type `Nat`, which is a supertype of `{1..3}`. Hence the second cast fails.

#### Up-casting for `loop`

We briefly mentioned up-casting in the introduction to `loop`. This is in fact a little tricky, because the type of the mutated variable undergoes the same mutation in each loop. So casting to any finite types will _not_ work, even if it contains all the values `x` will be mutated to. For example:

```

melorun> let x = 0 :: {0..10} in
loop 5 do
    x <- x + 1
return x
Melodeon compilation failed
error: assigning value of incompatible type {1..11} to variable x of type {0..10}
```

Here, we cast the type of `x` to `{0..10}`, but the compiler inferred the type of `x + 1` as `{1..11}`. So we have to cast `x` to `Nat`:

```

melorun> let x = 0 :: Nat in
loop 5 do
  x <- x + 1
return x
5

```

**_Exercise 6.7._** Will this expression compile? If not, make the smallest possible change so that it does.

```

let x = 5 :: {1..10} in
loop 3 do
    x <- 7
return x

```

### Downcasting and `unsafe`

In Melodeon, you can annotate any value `x` with an arbitrary type `T` using the **transmutation operator**, `x :! T`, _except_ when the compiler can see that `x` can never be of type `T`. Transmutation comes in handy when you know for sure that a value belongs to a certain type, but the compiler isn't smart enough to also know that.

Any expression containing `:!` must be preceded by the `unsafe` keyword. This is because changing a value's type to a subtype is `unsafe`, as the compiler can't prove that the value does in fact belong to the new type. Here we see the difference between manual downcasting to a subtype and occurrence typing: in occurrence typing, the down-cast is conditional on passing a runtime type test which proves that our value is a member of the subtype. In manual down-casting, there is no such guarantee.

Examples:

```

melorun> unsafe 1 :! [{1}]
1

melorun> unsafe let x = 1 in x :! %[]
1

melorun> let x = 1 in (unsafe x :! {2})
1

melorun> unsafe let x = 12 in
let y = 3 :: Nat in
loop 4 do
set! y = y + 1
return (x :! [Any;])
12

```

We see that the unsafe keyword can be put at any level of an arbitrarily complex nested expression, as long as it comes before any `:!`. The `unsafe` scope is the whole expression to the right of `unsafe`. You can have as many transmutations as you want within the scope of the `unsafe` keyword:

```

melorun> unsafe let x = 1 :! {2},
y = 2 :! {3},
z = 3 :! {4} in
"melodeon"
"melodeon"

melorun> unsafe let x = 1 :! {2} in let y = 3 :! {4} in x + y
4

```

Very importantly, `:!` only changes a value's type annotation, not what it is as a value. So `:!` does _not_ change the interpretation of the raw bits of the value (in contrast, [transmutation in Rust](https://doc.rust-lang.org/stable/std/mem/fn.transmute.html) and certain types of casting in C do). This is because values are _prior_ to types in Melodeon, and transmutation operates on types. Thus, `:!` cannot make a value work with functions or operators for an incompatible type. Instead, it's for when the type system is too restrictive about sound operations.

Examples:

```

def plus1(x: Nat) =
x + 1

```

```

melorun> unsafe let x = [1] :! {1} in plus1(x)
execution failed

melorun> unsafe let x = 1 :! %[1] in x ++ x
execution failed

```

A consequence is that although type annotations (with `::` or `:!`) can change _compile-time_ behavior (like whether a program type-checks), they do not affect _run-time_ behavior. Here are some examples.

At run-time, `x is T` checks whether the _value_ `x`, not its type annotation, is a member of the type `T`. So transmuting `x` will not change the result of any `is` expressions involving `x`:

```

melorun> unsafe let x = 1 :! {4} in x is {4}
0

melorun> unsafe let x = 1 :! %[] in x is %[]
0

```

**_Exercise 6.8._** Recalling that `True` is the type alias for `{1}` defined in the standard library, what's the result of this expression?

```

unsafe let x = 1 :! {0} in
let y = x is True in
y

```

**_Exercise 6.9._** Will this expression compile? If so, what does it evaluate to? If not, why not?

```

WIP

```

## Structures

Melodeon also has built-in structures, which are named vectors with named fields. To define a structure `Name` with fields `var_1, ..., var_n`, of types `type_1, ..., type_n` respectively, write in a file

```

struct Name {
var_1: type_1,
...,
var_n: type_n
}

```

Like type aliases, structure names should be in `UpperCamelCase` (in fact, the compiler enforces that they _must_ begin with an upper-case letter).

An example structure definition is

```

struct Point {
x: Nat,
y: Nat
}

```

Here's a particular instance of `Point`:

```

Point{ x: 1, y: 1 }

```

### Field access

You can access a field of a structure `s` with this syntax:

```

s.field_name

```

For example:

```

melorun> let p = Point{x: 1, y: 10} in p.x
1

melorun> let p = Point{x: 1, y: 10} in p.y
10

```

### Structure type

Structures are the one exception in Melodeon's type system: they break the rule that types are sets of values. The structure type

```

struct Name {
var_1: type_1,
...,
var_n: type_n
}

```

is both the supertype and the subtype of the vector type

```

[type_1, ..., type_n]

```

So a structure type and its corresponding vector type correspond to the same set of values. Formally, however, the structure and the vector are _distinct types_.

#### Both subtype and supertype of vector types

The fact that one is both the supertype and subtype of the other means that they're interchangeable when the only requirement is satisfying a subtype relation.

**Example 1**: because `x :: T` only asks whether `x`'s current type is a subtype of `T`, structs and their corresponding vectors are interchangeable as `x`. As a concrete example, a `Point` can be safely cast to `[Nat, Nat]` and used as one...

```

melorun> (Point{x: 1, y: 10} :: [Nat, Nat])[0]
1

melorun> for x in (Point{x: 1, y: 10} :: [Nat, Nat]) collect x \* 2
[2, 20]

melorun> let vec = Point{x: 1, y: 10} :: [Nat;] in
loop 2 do
set! vec = vec ++ vec
return vec
[1, 10, 1, 10, 1, 10, 1, 10]

```

...and vice-versa:

```

melorun> [1, 10] :: Point
[1, 10]

melorun> let p = [1, 10] :: Point in p.y
10

```

So any struct can be cast to its corresponding vector and vice versa. This means that **with casting**, structs and vectors are _always_ compatible.

**Example 2**: as arguments to functions that don't involve constant generics, vectors and structs are also compatible without casting:

```

def double(x: [Nat; 2]) =
    for i in x collect i*2

def slope(p1: Point, p2: Point) : Nat =
    let rise = (p2.y - p1.y) in
    let run = (p2.x - p1.x) in
    rise / run

```

```

melorun> double(Point{x: 1, y: 2})
[2, 4]

melorun> slope([0, 0], [1, 1])
1

```

#### Distinct from vector types

Then, the fact that structs and their corresponding vectors are distinct types means they're incompatible in contexts that require more than a subtype relation. For example, you cannot use structs with vector operators or in `for` loops, because those call for actual vectors, not just a subtype of vectors:

```

melorun> Point{x: 1, y: 2}[0]
Melodeon compilation failed
~/.melorun.melo.tmp:70: error: type Point cannot be indexed into
Point{x: 1, y: 2}[0]
^~~~~~~~~~~~~~~~~~~~

melorun> Point{x: 1, y: 2}[0..1]
Melodeon compilation failed
(unknown location): error: type Point cannot be sliced into

melorun> for x in (Point{x: 1, y: 10}) collect x _ 2
Melodeon compilation failed
~/.melorun.melo.tmp:66: error: list comprehension needs a list with a definite length, got Point instead
for x in (Point{x: 1, y: 10}) collect x _ 2
^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

To wrap up, every structure type corresponds to a vector type that is both its subtype and supertype. It's distinct from that vector type, but it's also interchangeable with the vector type where satisfying a subtype relation is the only constraint.

**_Exercise 6.10._** Will the function `h` below take a `Point` structure as input? If not, why not?

```

def h<$n, T>(x: [T; $n+1]) =
    x[0]

```
