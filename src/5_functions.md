# Functions

Unlike in most functional programming languages, functions are _not_ first-class citizens in Melodeon. They cannot be passed as arguments into other functions (that is, Melodeon doesn't have higher-order functions). Instead, they are always defined at the "top level".

Here's how to define a function in Melodeon:

```
def function_name(arg_1: type_1, arg_2: type_2, ..., arg_n: type_n) =
    expression
```

Argument type annotations are mandatory, as explained in the section on [type inference](simple_types.md). The function body must be a _single_ expression.

Examples:

```
def double(x: Nat) = x * 2
```

```
def poly(x: Nat) =
    x ** 2 + quadruple(x) + 8

def quadruple(x: Nat) =
    loop 2 do
        x <- x + x
    return x
```

```
def to_point(x: Nat, y: Nat) =
    Point{x: x, y: y}

struct Point {
    x: Nat,
    y: Nat
}
```

As we see from these examples, the order of definitions does not matter in Melodeon. Furthermore, function and variable names _must_ start with a lower-case letter and are idiomatically written in `snake_case`.

Optionally, you can annotate the output type, like this:

```
def double(x: Nat) : Nat = x * 2

def tuplify(x: Nat, y: Nat) : Point =
    Point{x: x, y: y}
```

Currently, all functions must be defined in `.melo` files; support for function definitions in the REPL is forthcoming!
