# Install Melodeon

Welcome to Melodeon! To start writing scripts and running them you'll want to download the command-line tool, melorun.

If you have the [Rust compiler](https://www.rust-lang.org/) installed, the easiest way is simply the following:

```
cargo install melorun
```

If you don't have rust, you can download it using [rustup](https://rustup.rs/),
an installer and version manager for rust.

Melorun is a convenient multi-tool which compiles Melodeon code and evaluates it. The Melodeon programming language was designed to run in the Themelio blockchain's virtual machine, MelVM. It does not compile to native machine code, so melorun is handling the compiling and executing of melodeon programs in one convenient interface.
