# Getting Started

Most likely, you'll want to develop Melodeon programs off the blockchain in `melorun`, a command-line REPL or _read-eval-print-loop_.

### Installation

To install `melorun`, first make sure your machine has [Rust](https://www.rust-lang.org/tools/install) installed. Then, open a terminal and enter the following command:

```
cargo install melorun
```

### Quick start

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

### Executing files

To execute a code file (these must end in `.melo`) in the REPL, attach the file's path to the end of the `melorun` command:

```
melorun filename.melo
```

To interact with definitions from a `.melo` file, add the `-i` flag:

```
melorun -i filename.melo
```

Here's a concrete example. First, create a file named `demo.melo`, with contents

```
def double(x: Nat) = 2*x
```

This defines a function `double` that takes in a non-negative number and doubles it. Then, open a fresh terminal and make sure you're in the same directory as `demo.melo`. Next, type

```
melorun -i "demo.melo"
```

You can now use the `double` function in this REPL session:

```
melorun> double(2)
- : Nat
4

melorun> double(4+1)
- : Nat
10
```
