## Your first Melodeon covenant

Let's try running our first Melodeon program with `melorun`! Traditionally, this would be a "hello world" program, but as a covenant language, Melodeon doesn't really have I/O functionality like printing.

Instead, we simply write a covenant that _always returns true_. Such a covenant would allow anyone to spend any coin locked by it (more on coins and scripts in the [next section]()):

```
1
```

Yep, that's all!

We'll save that in a file, `just1.melo`, and run it.

```
melorun just1.melo
```

The following output should be produced

```
result: Covenant evaluates to TRUE
value: 1
```

This indicates that the covenant evaluated to `1`, which is a **truthy** value in Melodeon. In fact, all values other than the sole **falsy** value `0` is "truthy".

"Truthy" values behave like the `true` boolean. Among other things, this means that:

- If a covenant is [locking a coin](), returning a truthy value allows the coin to be spent.
- [If-then-else expressions]() evaluates to their first branch; e.g. `if 1 then 2 else 3` evaluates to `2`
