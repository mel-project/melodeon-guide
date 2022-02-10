# What's a covenant?

Melodeon is simply a new language and can be seen independently of a particular
platform. That being said, its origin comes from a need for a high-level
language to write scripts for a non-Turing complete, structurally typed,
stack-based machine - [the MelVM]().

The MelVM is an execution environment designed for UTXO-based smart contracts.
UTXO scripts have a (relatively) long history starting with Bitcoin, where an
unspent transaction's output (UTXO, or coin, in Themelio jargon) is attempted
to be spent by a "spender" transaction which passes some data to the UTXO's
script and a validator runs the script to see whether it evaluates to true. In
the case that it does, the "spender" transaction is valid and the UTXO is
spent.

*TODO find a good UTXO reference*

A covenant is a more powerful version of Bitcoin scripts. The difference is in
the information the script has access to. A covenant script can see the whole
transaction that is trying to spend it. Themelio covenants can also see some
other information like the current block height and the latest block header.

These simple additions enable some powerful properties. For instance,
self-propagating scripts are a script which ensures that the transaction that
spends it contains a coin with the same script as itself. Thus the script
becomes a kind of "stateful" smart contract where ownership and data can change
but the code does not.

## Why Melodeon for Covenants?

Melodeon compiles to [Mil]() (standing for MelVM Intermediate Lisp) which is an
intermediate-level language kind of like llvm's IR which stands between high
level languages and machine code.

A melodeon program can simply be described as a set of definitions and a single
expression. The single expression will return a value which the MelVM
interprets as `0 is false, otherwise true`. Therefore this expression decides
the spendability of a coin on the themelio blockchain.
