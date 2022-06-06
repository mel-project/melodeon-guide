## Using the REPL

Melorun also offers an interactive REPL (Read-Evaluate-Print-Loop) where you
can quickly try things out and even test functions you define in a file.

### Using the REPL by itself

You can start the REPL by using melorun's **interactive mode**:

```
melorun -i
```

This gives you a prompt that looks something like:

```
melorun>
```

You can type any Melodeon expression into the REPL, and it will evaluate it. For example, basic mathematical expressions work:

```
melorun> 1 + 2
3
melorun> 1 + 2 * 3
7
melorun> (1 + 2) * 3
9
```

Throughout this book, we will often use the REPL to illustrate the semantics of Melodeon.

### Loading files into the REPL

By passing an argument to `melorun`, you load that program too. For example, the following program defines a function `f` that, when called, simply returns the number `3`:

```
def f() = 3
```

If you put the above content in a file named `test.melo` and run `melorun -i test.melo`, then `f()` will be available to the REPL:

```
melorun> 1 + 2 + f()
6
```
