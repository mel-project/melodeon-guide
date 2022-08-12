# Welcome to Melodeon!

Melodeon is a non-Turing-complete, yet powerful and general **covenant** language for the Themelio blockchain.

## About Themelio covenants

Covenants are Themelio's equivalent of "smart contracts" in its coin-based model. A covenant, somewhat analogous to a Bitcoin script, is a program attached to a coin (UTXO) that takes in a transaction and its execution context and decides whether or not that transaction can spend. On the blockchain itself, contracts are written in [MelVM](https://docs.themelio.org/specifications/melvm-specification/), a low-level stack-based "machine" language fulfilling a similar role as EVM in Ethereum.

Unlike coin scripts in most other UTXO blockchains, MelVM covenants are **general-purpose**, meaning that they can arbitrarily control the spending of a coin and are not limited to simple spending conditions like multi-signature requirements. This allows for essentially all on-chain constructs to be built on top of Themelio, such as naming systems, DeFi contracts, etc. (Note, though that the UTXO model is generally much more suited for event-focused logic that _commits to_ rather than directly encodes application actions and data, rather than highly stateful business logic. This is an intentional choice to encourage using Themelio as a root of trust rather than an execution platform.)

## About Melodeon

Melodeon is a high-level language that compiles to MelVM. It is:

- **Purely functional and stateless**: There is no mutation, storage, or I/O, mirroring Themelio's stateless, coin-based covenants. This avoids many subtle correctness issues that can emerge in traditional, Solidity-style blockchain contracts, and has the additional benefit of making the language easier to reason about.

- **Total**: All programs in Melodeon provably terminate. This allows us to calculate the amount of computation each covenant requires and exactly how much fees to charge well in advance, avoiding the complexity of runtime gas accounting common in smart-contract languages. (This does mean Melodeon is not Turing complete, but we promise that it will _feel_ like writing a normal, Turing-complete language)

- **Type-safe**: All types must be known at compile-time and all the safety guarantees that come with it. Similar to languages like TypeScript, Typed Racket, and (some aspects of) ML, we use a _structural typing system_ where types correspond to sets of values, rather than forcing a fixed hierarchy of type names.

This guide assumes some prior programming experience. Let's dive right in!
