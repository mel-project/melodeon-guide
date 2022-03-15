## Using The Repl

Melorun also offers an interactive repl (Read-Evaluate-Print-Loop) where you
can quickly try things out and even test functions you define in a file.

The melorun repl expects to be provided with a melodeon file. It will import
the function and type definitions in this for use in the repl.

```
melorun -i test.melo
```

If you just want to run a repl the file can be empty.

In the repl you can define variables in a slightly more condensed syntax than
actual melodeon. Evaluations of expressions will be printed.

```
melorun> x = 1 + 2
melorun> x
1
```
