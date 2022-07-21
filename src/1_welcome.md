# Welcome to Melodeon!

Melodeon is a non-Turing-complete, yet powerful and general _covenant-scripting_ language for the Themelio blockchain.

Broadly speaking, covenants are Themelio's equivalent of "smart contracts" in its coin-based model. A covenant is a program attached to a coin (UTXO) that takes in a transaction and its execution context, produces `True` or `False`. They're attached to coins (UTXOs) and used for determining whether a given transaction, in its context, is allowed to spend that coin.

[**insert** stmt about the power of Themelio covenants; how they compare to smart contracts; a few example covenants, like social recovery wallets and DAOs].

Melodeon is:

- **Purely functional and stateless**: There is no mutation, storage, or I/O. [**insert** discussion of what a pain mutable state in smart contracts is? Anyway something to justify this]

- **Total**: All programs in Melodeon provably terminate. This allows us to calculate the amount of computation each covenant requires and exactly how much fees to charge well in advance, avoiding the complexity of runtime gas accounting common in smart-contract languages. (This does mean Melodeon is not Turing complete, but we promise that it will _feel_ like writing a normal, Turing-complete language)

- **Strong, structural, static typing**: All types must be known at compile-time and all the safety guarantees that come with it. Similar to languages like TypeScript, Typed Racket, and (some aspects of) ML, types are _structural_ and correspond to sets of values, rather than forming a hierarchy of type names.

This guide assumes some prior programming experience. Let's dive right in!
