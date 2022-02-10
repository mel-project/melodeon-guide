# Getting started

Welcome to Melodeon! To start writing scripts and running them you'll want to download the command-line tool, melorun.

If you have the [Rust compiler](https://www.rust-lang.org/) installed, the easiest way is simply the following:

```
cargo install melorun
```

Melorun is a convenient multi-tool which compiles Melodeon code and evaluates it. The Melodeon programming language was designed to run in the Themelio blockchain's virtual machine, MelVM. It does not compile to native machine code, so melorun is handling the compiling and executing of melodeon programs in one convenient interface.


## Run Your First Script
Let's try running a melodeon script with melorun. Print statements are not really useful on a blockchain so we'll skip the "hello world" example and instead write the simplest-concievable-but-also-useful melodeon script - simply a script that returns true. Such a script would allow anyone to spend a coin containing this script (more on coins and scripts in the next section on [covenants]()).

Ok here's the program which always returns true.
```1```

We'll save that in a file, `just1.melo`, and run it.

```
melorun just1.melo
```

The following output should be produced
```
result: Covenant evaluates true
value: 1
```

Essentially this is saying that the script evaluated to the value `1`, which is interpretted as `true` in MelVM. In fact any value besides 0 is true, and 0 itself is false.
