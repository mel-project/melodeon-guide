## Functions

Functions are simple enough in Melodeon. Here is a function to add two natural
numbers (Nat), which of course returns a natural as well.

```
def add(x: Nat, y: Nat): Nat =
    x + y
```

Melodeon enforces strong typing but also offers type inference in many cases.
In the case above, the return type Nat can be omitted as it is implied by the
body of the function.

### Structure of a program

The above code would compile as a valid program. This would basically just be
a library because there is no expression actually executing, just a single
function definition.

In fact a Melodeon program is a set of these definitions (functions and types),
followed by a single expression. The reason for the single expression will be
explained more in [What is a Covenant?](), but for now it suffices to notice
that since a Melodeon program is stateless/functional, a single expression is
all you need.

The notation Melodeon uses to seperate definitions from the executing
expression is three dashes `---`.

```
fn id<T>(x: T): T = x

---
id(x"deadbeef")
```
