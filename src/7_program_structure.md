# Program Structure

An executable `.melo` file contains 0 or more definitions (these can be of functions, structures, and `type` aliases) and, optionally, three dashes (`---`) followed by a _single_ expression.

Here's an example file `fibonacci.melo`. It defines a `fibonacci` function and calls it on `15`

```
# returns the nth fibonacci number, starting from 0
def fibonacci<$n>(n: {$n}) =
    let i = 0 :: Nat in
    let j = 1 :: Nat in
    let tmp = 0 :: Nat in
    loop $n do
        tmp <- j;
        j <- i + j;
        i <- tmp;
    return i

---

fibonacci(15)
```

When you start a REPL instance with `fibonacci.melo`, the expression `fibonacci(15)` is immediately evaluated:

```
$ melorun -i fibonacci.melo
- : Nat
610 (truthy)
melorun>


```

The three dashes are needed to end the definitions section because whitespace is insignificant in Melodeon, _and_ Melodeon doesn't have delimiters. While this makes Melodeon syntax both elegant and flexible, it also means that the compiler cannot tell where the last function definition ends and the standalone expression begins without some help from us.

**Comments** start with a single `#`. For an example, see the first line of `fibonacci.melo` above.
