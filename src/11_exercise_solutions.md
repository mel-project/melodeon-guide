# Solutions to Exercises

***Solution 3.1.*** There's no standard answer to this one. Here's ours:
```
{1..2} | %[3]
```

***Solution 3.2.*** 
```
[%[2], {517}, []]
```

***Solution 4.1.*** 
```
4
```

***Solution 4.2.*** 
```
["melo", "haha", "deon"]
```

***Solution 4.3.***

<!-- Nope. You can only compare two values of the *same* type. `1 == [2]` fails to typecheck and does not compile:
```
melorun> 1 == [2]
Melodeon compilation failed
~/.melorun.melo.tmp:4: error: expected a subtype of Nat, got [{2}] instead
	1 == [2]
	^~~~~~~~
``` -->
Nope. You can only append two values of the *same* type. `"1" ++ [2]` fails to type-check and does not compile:
```
melorun> "1" ++ [2]
Melodeon compilation failed
~/.melorun.melo.tmp:3: error: cannot append values of types %[1] and [{2}]
	"1" ++ [2]
	^~~~~~~~~~
```


***Solution 4.4.***
```
{35} | %[8]
```

***Solution 4.5.***

Two alternatives:
```
[x * x for x in [1, 10, 100]]

for x in [1, 10, 100] collect x * x
```

***Solution 4.6.***
WIP

***Solution 4.7.***
```
let x = [1, 2] in
loop 2 do
    set! x = x ++ x
    return x
```

***Solution 6.1.***
```
def six1<T>(x: T) = 
    if x is Nat then x + 1
    else (if x is [Any;] then x ++ [1]
        else if x is %[] then x ++ "1"
            else fail!)
```

***Solution 6.2.***
```
# swaps the ith and jth indices of vec

def swap<$n, T>(vec: [T; $n], i: Nat, j: Nat) =
    let v_i = vec[i] in
    let v_j = vec[j] in
    vec[i => v_j][j => v_i]
```

***Solution 6.3.***
The function doesn't compile, because the length of `vec` is unknown from its type definition. We can use a constant generic parameter to solve this problem:
```
def f<$n>(vec: [Nat;$n]) =
    for i in vec fold accum = 0 :: Nat with accum + i
```

***Solution 6.4.***
```
def ident<$n>(s: %[2 * $n + 1]) = 
    s
```

***Solution 6.5.***
```
# swap is defined in Solution 6.2

def bubble_sort<$n>(vec: [Nat; $n]) =
    loop $n do
        set! vec = 
            (let i = $n :: Nat in
            let j = $n-1 :: Nat in
            (loop $n do 
                set! vec =
                (if j > 0 
                then (if vec[$n - i] > vec[$n - j]
                    then swap(vec, $n - i, $n - j)
                    else vec)
                else vec);
                set! i = i - 1;
                set! j = j - 1;
                return vec))
        return vec
```
Note that the trick with decreasing `i` and `j` is for safely indexing into the vector with an expression involving `$n`, so that the compiler can prove the indices are safe.

***Solution 6.6.***
```
def f(x: Any) =
    if x is %[5] then "bingo!" else "keep trying!"
```

***Solution 6.7.***
Yes, the expression compiles. In each iteration of the loop, `x` is mutated to the constant `7`, which doesn't depend on `x`. So the post-mutation type also doesn't depend on the type of `x`: it's the constant  `{7}`. The compiler can check that `{7}` is a subtype of `{1..10}`, so all is good.

***Solution 6.8.***
```
1
```

***Solution 6.9.***
```
WIP
```

***Solution 6.10.***
No, because the input to `h` needs to satisfy more than a subtype relation. It must satisfy the constraints imposed by the constant generic parameter `$n`. We can see this in the REPL:
```
melorun> h(Point{x: 1, y: 2})
Melodeon compilation failed
~/.melorun.melo.tmp:72: error: cannot fill type variables in argument type [T; ($n + 1)], given concrete Point
	h(Point{x: 1, y: 2})
	^~~~~~~~~~~~~~~~~~~~
```