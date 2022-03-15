## The Let Expression

The let expression is how you bind variables in Melodeon. Remember that Melodeon is a quasi-purely functional language, meaning that there is no mutation of state and everything is an expression. An expression is a "sub-program" that produces a value. 1 is an expression that returns 1, 1+2 is an expression that returns 3.

Not being able to mutate state does not mean you can't use variables, it just means the value of those variables is immutable in their defined scope. It also means that variable bindings are in fact an expression. We bind variables with a "let" expression:

```python
let x = 4,
    y = 3 in
    x+y
```

This expression returns the value 7. A let expression is essentially binding some variables to some values, and then defines an expression where those variables are in scope.

Whitespace is insignificant in melodeon. The scoped expression `x+y` in the example above is separated from the bindings simply by the keyword "in".

If the idea of a variable binding being an expression is strange to you, then you may find the following example disturbing. But hopefully it will help enlighten your mind to the possibilities.

```python
let z =
  let x = 2,
      y = 3 in
      x * y
  in z / 3
```
