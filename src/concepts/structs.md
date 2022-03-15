## Structs

Structs are a way to couple data together into more complex structures.

```
struct Point {
    x: Nat,
    y: Nat,
}
```

We can instantiate a struct and access its fields.

```
let p = Point { 0, 1 } in
    p.x
```

Structs are types like any other primitive.

```
def add(p1: Point, p2: Point) =
    Point {
        p1.x + p2.x,
        p1.y + p2.y,
    }
```

TODO: Explain equivalence of structs and their representation in terms of
structural typing. Either here or in the section on structural typing.

### Alias Types

We can also create simple aliases for existing types which have no change in
their representation. For instance in the signed_int library, the Int type is
actually just

```
type Int = Nat
```

Then the functions such as negate and subtract are implemented on type Int
rather than Nat.
