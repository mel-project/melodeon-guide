# Getting Started

While developing Melodeon covenants, we want to be able to run and test our code without deploying it onto a blockchain. Generally, this means using a REPL or _read-eval-print-loop_.

There are currently two official REPL tools: an offline, command-line tool called `melorun`, as well as an [online playground](https://play.melodeonlang.org/).

## Installing melorun

To install `melorun`, first make sure your machine has [Rust](https://www.rust-lang.org/tools/install) installed. Then, open a terminal and enter the following command:

```
cargo install melorun
```

## Quick start

Once `melorun` is installed, enter

```
melorun -i
```

in a terminal to fire up the REPL in interactive mode. Your terminal should display this prompt:

```
melorun>
```

Now, you can type any Melodeon expression following the prompt, hit enter, and the REPL will tell you what the expression evaluates to. It will also tell you the _type_ of the expression after `- :`, something we will discuss [shortly](3_simple_types.md).

Examples:

```
melorun> 5
- : Nat [more specifically: {5}]
5

melorun> 1 + 7
- : Nat [more specifically: {8}]
8

melorun> (1 + 2) * 3
- : Nat [more specifically: {9}]
9
```

## Executing covenants

To execute a complete covenant file (these must end in `.melo`) in the REPL, attach the file's path to the end of the `melorun` command:

```
melorun filename.melo
```

To interact with definitions from a `.melo` file, add the `-i` flag:

```
melorun -i filename.melo
```

Before going into two examples, recall that covenants are run in an _execution environment_ consisting of, in part, the transaction that is trying to spend the coin locked by the covenant. What value the covenant produces then determines whether the transaction is valid.

### Trivial example

Here's an example of a very simple covenant. First, create a file named `demo.melo`, with contents

```
def double(x: Nat) = 2*x

---

double(1)
```

This defines a function `double` that takes in a non-negative number and doubles it. In the main body of the covenant (separated by `---`), we then call double with `1`.

Then, open a fresh terminal and make sure you're in the same directory as `demo.melo`. Next, type

```
melorun -i demo.melo
```

We see this output:

```
- : Nat
2 (truthy)
melorun>
```

This indicates that this covenant returned `2`, which is a [_truthy_](3_simple_types.md#values) value. In short, this means that **this covenant will allow _whatever_ transaction tries to spend a coin locked by it**.

> 💡 **Aside**: Since we used `melorun`'s interactive mode, you can play with the `double` function in this REPL session:
>
> ```
> melorun> double(2)
> - : Nat
> 4
>
> melorun> double(4+1)
> - : Nat
> 10
> ```

### More interesting example

Edit `demo.melo` to contain the following instead:

```
def double(x: Nat) = 2*x

---

double(vlen(env_spender_tx().outputs)) == 4
```

Now, this covenant instead requires that _whatever transaction that spends the coin contain exactly 2 outputs_. (see TODO documentation on `env_spender_tx` and `Transaction`)

Now if we run the covenant like this, we get a falsy value (meaning "transaction is rejected"):

```
$ melorun -i demo.melo
- : Nat [more specifically: {0..1}]
0 (falsy)
```

Hmm... _what_ transaction is `melorun` handing to the covenant and getting rejected? It turns out that `melorun` by default populates the environment with a **dummy transaction** (more info in TODO):

> ```
> melorun> env_spender_tx()
> - : Transaction
> [0, [], [], 0, [], "", []]
> melorun> env_spender_tx().outputs
> - : [CoinData;]
> []
> ```

We can instead specify a _spend context_ (more info in TODO), which contains further information about the transaction to generate for testing the covenant. Make a file `context.yaml` containing:

```
spender_outputs:
  0:
    covhash: t48dn3f3k4xfevwwx97hzrgxmgcz3b6xw2sdnkfb6c6e3erqjq2sh0
    denom: MEL
    value: 10000
    additional_data: ""
  1:
    covhash: t48dn3f3k4xfevwwx97hzrgxmgcz3b6xw2sdnkfb6c6e3erqjq2sh0
    denom: MEL
    value: 10000
    additional_data: ""
```

This specifies two transaction outputs. Now, if we run

```
$ melorun -i demo.melo -s context.yaml
- : Nat [more specifically: {0..1}]
1 (truthy)
```

we get a "truthy" result. We can see that the dummy transaction now has outputs:

```
melorun> env_spender_tx().outputs
- : [CoinData;]
[[x"436a378e64ebddbe73a93c7f88769067c6b37782cb6b37accc3386ec5e571662", 10000, "m", ""],
 [x"436a378e64ebddbe73a93c7f88769067c6b37782cb6b37accc3386ec5e571662", 10000, "m", ""]]
```
