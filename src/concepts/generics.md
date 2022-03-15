## Generics

### Type Generics

We can write functions to be generic over the type of inputs and output. This
way we don't have to rewrite an algorithm for every type we want to implement
it on.

Here's the identity function.

```
def id<T>(x: T): T = x
```

Again, specifying the return type was redundant and we could have left it to
inference. `id` takes some value x of some unknown type T and just returns it.
Type generics must be uppercase.

The type of T is determined at compile time (not run time) when id is called.

```
id(5)
```

The above expression would interpret T to be of type Nat during the call. Type
generics in Melodeon work very similar as in Rust. If this isn't making sense
it may be useful to check out their [docs on
generics](https://doc.rust-lang.org/book/ch10-01-syntax.html) for more details.

Multiple generics can also be interpreted by the compiler.

```
def tuplize<A, B>(a: A, b: B): [A, B] = [a, b]
---
tuplize(2, "hello")
```

### Constant Generics

Constant generics really bring the features of Melodeon together. Without them
length-typed arrays would make programming very difficult *..cough Solidity..*

They offer a way for types in a function definition to generic over a number (a
constant). They are alot like type generics except instead of types, they are
for numbers used in types. We define them with a special `$` notation, where
constant generics are *lowercase*:

```
def concat<$n>(xs: [Nat; $n], ys: [Nat; $n]): [Nat; 2*$n] = xs ++ ys
```

Wow that's cool! Take a moment to soak that in. We took two arrays of size
n and returned an array of size 2*n. We could have even made this generic over
the inner array type (Nat), and would have a fully library-ready function!

The 2*$n is a special expression called a constant generic expression.
Certain operations are allowed to be done on constant generics. Those include
addition and multiplication. You cannot subtract or divide them. That would lead to
very confusing logical problems when reasoning about types.

Constant generics can also be used in any expression that expects a constant
generic. For instance, in the upper bound for a loop.
