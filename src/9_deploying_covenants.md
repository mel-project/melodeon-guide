# Deploying Covenants

Now that we've learned the basics of Melodeon programs, we'll learn how to actually deploy covenants onto the blockchain. Throughout this chapter, we use the following covenant as an example --- a 2-of-3 multisignature:

```
let alice_public_key = x"b5c7302463eda5f951d29ed1c6f7e9c7056628fb967b92653e0f9a1c15b6ad7e" in
let bob_public_key = x"37290798b218b2312faa66f91191f6180c530df0fabf3f2791417e48d69d9b8b" in
let charlie_public_key = x"315a04b0ca34049f5f47621da1b72cdeadbf51c5aacbbe14d88a8b9d77229deb" in

(ed25519_signed_by(alice_public_key, 10) +
ed25519_signed_by(bob_public_key, 11) +
ed25519_signed_by(charlie_public_key, 12)) >= 2
```

More precisely, this checks that _at least 2_ of the following must be true:

- the 11th signature (indices in Melodeon start with 0, so `10` is the 11th signature) of the spending transaction is a valid signature from Alice
- the 12th signature is a valid signature from Bob
- the 13th signature is a valid signature from Charlie

(see the TODO for `ed25519_signed_by`)

## Testing covenants before deployment

Both `melorun` and [the online playground](https://play.melodeon.org) support specifying the **spend environment**. This is a special, [YAML-formatted](https://en.wikipedia.org/wiki/YAML) document that allows you to specify a particular spending transaction without constructing every field of it manually --- see its spec TODO.

Here, we only care about the `ed25519_signers`, which is a mapping betwween an integer `n` and the _private_ key that signs the `n`th signature of the transaction. Put this in the spend environment:

```yaml
ed25519_signers:
  # Alice's PRIVATE key
  10: 85f884fb8400117a29ef65b3a3ace00e7a6e1dde65479361c67e564fd00de417b5c7302463eda5f951d29ed1c6f7e9c7056628fb967b92653e0f9a1c15b6ad7e
  # Bob's PRIVATE key
  11: 820384db2898cb4abaa6662de1087c955a7e08b13abcb69c8e7941a69ff372e437290798b218b2312faa66f91191f6180c530df0fabf3f2791417e48d69d9b8b
  # Charlie's PRIVATE key
  12: 35f29230304c0f0e0c730758caabd121a312dec4c1c0b859a75b1b59513ab4ca315a04b0ca34049f5f47621da1b72cdeadbf51c5aacbbe14d88a8b9d77229deb
```

You can apply this spend environment with the `-s` flag of `melorun`, or just [use the playground](https://play.melodeonlang.org/#SQQAAPGBzMzMMcwAZf4jAP4yzAXM_jN7JhZK1Dv-XAn-XAlXI1iDAv5fPCZalxz_AV9rAkf-PQD6_lxlWv41HP8GNzMwMjQ2M0IE_jUs_wI5NTEY_wEyOUL-MRz-Niz-NwL-ORz_BjcwNTY2MjgsWv8COTY3Wv8EOTI2NTMC_jAs_jkE_jEc_wExNVr-Nuj-NwL-XGVOdgA_WgZadgAB9Un_BzM3MjkwNzk4Wv8CMjE4Wv8DMjMxMiwEBP8BNjYs_wQ5MTE5MSz_AzYxODAc_wI1MzAYLP4wLARaLP4zLP8GMjc5MTQxNwL_ATQ4GP8BNjkY_jla_jhaeAA6mU-X7wDwEf8BPVwJ-v5cZf8CMzE1BP8BMDRa_jAcBP8EMzQwNDks-QD0KAQ0NzYyMRgE_jFa_wE3Mhxq6Fos_wE1MRz-NQTvWlz_ATE0GP8BODgE_jha_jkY_wQ3NzIyOWp2AABqAfsEKEL_BTI1NTE5X9Q7ixj-X3_-KHkB3VMk_wEwKQD_AStcixgtAAswAQAtAB8xLQAGDOUA-ARTJP8CMikpAN7-PQD-MmUxzABljwDwS93_ATpcKTT_ATA6AP8BODUs_wI4ODQsWv8GODQwMDExNwT_ATI5Aiz_ATY1Wv4zBP4z7wL_ATAwAv43BP42Av4xGGr_BzY1NDc5MzYxHP8BNjcC_wI1NjQsGCYAb2r_AjQxN08CRQC3AMAxOgD_BTgyMDM4NBgxAv8-ODk4HFr-NARaBAT_AzY2NjJq_wMxMDg3HP8COTU1BP43Av8BMDha_wExMwRaHFr_ATY5HP44Av8DNzk0MQT_ATY5LCz_AjM3MgL_CDSQAkUAuQAQMnAB8CUzNSz_BzI5MjMwMzA0HP4wLP4wAv4wHP8FNzMwNzU4HAQEWhj_AjEyMQT_AjMxMmoc_jQcgAPQMFr_Ajg1OQT_ATc1WpoCsQQ1OTUxMwRa_jQcKwAPzQI4YDI5alplMQ). Here, we sign the transaction with _all three signatures_ that the covenant is looking for, which makes it return `1` (and allow the transaction through).

Feel free to play around by removing some of the signers --- the covenant will still return `1` if at least 2 signatures are intact, but not otherwise.

## Testing on the testnet

The next step now is to test our covenant live on the official testnet.

### Compiling to MelVM bytecode

To do that, we need to first compile our Melodeon program to MelVM bytecode. This can be done with the `-c` (or `--compile`) flag of `melorun`. Run `melorun -c -i multisig.melo`, where `multisig.melo` contains the above covenant.

You'll see something the following two lines on `stdout`:

```
t48rt8nvzc1kvc9s1040b7746mdgh8ty8a0pdwqc9kd6d45myq2hx0
f020b5c7302463eda5f951d29ed1c6f7e9c7056628fb967b92653e0f9a1c15b6ad7e430021f02037290798b218b2312faa66f91191f6180c530df0fabf3f2791417e48d69d9b8b430022f020315a04b0ca34049f5f47621da1b72cdeadbf51c5aacbbe14d88a8b9d77229deb430023f20102f201064200005043005ef2010c43005f42005e5342005f25a1000442005f42005e50a00001f20043005d42005da1000542005d420023420001320020a00001f200f2010642000050430053f2010b4300544200535342005425a1000442005442005350a00001f200430052420052a10005420052420022420001320020a00001f200f2010642000050430048f2010a4300494200485342004925a1000442004942004850a00001f200430047420047a10005420047420021420001320020a00001f200101024f20102f201064200005043003df2010c43003e42003d5342003e25a1000442003e42003d50a00001f20043003c42003ca1000542003c420023420001320020a00001f200f2010642000050430032f2010b4300334200325342003325a1000442003342003250a00001f200430031420031a10005420031420022420001320020a00001f200f2010642000050430027f2010a4300284200275342002825a1000442002842002750a00001f200430026420026a10005420026420021420001320020a00001f20010102621
```

The first line is the **hash** of the covenant in the standard "address" format, while the second line contains the hex-encoded MelVM itself itself.

### Sending a coin locked by this covenant

The next step is to send a coin locked by this covenant. Let's start `melwalletd` in testnet mode and use `melwallet-cli` to send some money to this covenant:

(in one terminal tab)

```
$ melwalletd --wallet-dir ~/.wallets --network testnet
```

(in another terminal tab)

We make a wallet `demo` with a bunch of testnet play money:

```
$ melwallet-cli create -w demo
$ melwallet-cli unlock -w demo
```

We can now send 100 (fake) MEL to the _covenant hash_ we saw earlier:

```
$ melwallet-cli send -w demo --to t48rt8nvzc1kvc9s1040b7746mdgh8ty8a0pdwqc9kd6d45myq2hx0,100.0
TRANSACTION RECIPIENTS
Address                                                 Amount          Additional data
t48rt8nvzc1kvc9s1040b7746mdgh8ty8a0pdwqc9kd6d45myq2hx0  100.000000 MEL  ""
t92zte6gh26y5s02g9h31vgmnv0q4vya8paw5vghfe6c5b4hvj28pg  8.187089 MEL    ""
t92zte6gh26y5s02g9h31vgmnv0q4vya8paw5vghfe6c5b4hvj28pg  8.187089 MEL    ""
 (network fees)                                         0.000395 MEL
Proceed? [y/N]
y
Transaction hash:  630024d8f526fa15cb53d40aec440e63ae32c636229696e312e66f311fee7c6b
(wait for confirmation with melwallet-cli wait-confirmation -w demo 630024d8f526fa15cb53d40aec440e63ae32c636229696e312e66f311fee7c6b)
```

After this transaction confirms, this locks up 100 MEL at the **first output** of the transaction with hash `630024d8f526fa15cb53d40aec440e63ae32c636229696e312e66f311fee7c6b`, that can only be spent by a transaction that satisfies the covenant we wrote earlier. (You can even see it on [Melscan](https://scan-testnet.themelio.org/blocks/48418/630024d8f526fa15cb53d40aec440e63ae32c636229696e312e66f311fee7c6b) --- the above is a transcript of an actaul testnet deployment!)

### Spending the locked coin

To test our covenant in action, the next step is to create a transaction that satisfies the covenant. This is actually a little tricky, since we must manually create a transaction with the signatures in exactly the right spots, something that's generally not a built-in functionality of wallets like `melwallet-cli`/`melwalletd`. Instead, we need to create a transaction with the signatures from Alice, Bob, and Charlie "left blank", then sign the transaction manually.

Note that in the following steps, we use **bash/zsh** syntax for string interpolation to avoid repeatedly typing long strings. On Windows, Git Bash is a good way of accessing bash syntax.

1. **Creating the transaction**: using _any_ wallet, run the following `melwallet-cli` command:

   ```
   melwallet-cli send -w demo --force-spend 630024d8f526fa15cb53d40aec440e63ae32c636229696e312e66f311fee7c6b-0 --fee-ballast 1000 --add-covenant f020b5c7302463eda5f951d29ed1c6f7e9c7056628fb967b92653e0f9a1c15b6ad7e430021f02037290798b218b2312faa66f91191f6180c530df0fabf3f2791417e48d69d9b8b430022f020315a04b0ca34049f5f47621da1b72cdeadbf51c5aacbbe14d88a8b9d77229deb430023f20102f201064200005043005ef2010c43005f42005e5342005f25a1000442005f42005e50a00001f20043005d42005da1000542005d420023420001320020a00001f200f2010642000050430053f2010b4300544200535342005425a1000442005442005350a00001f200430052420052a10005420052420022420001320020a00001f200f2010642000050430048f2010a4300494200485342004925a1000442004942004850a00001f200430047420047a10005420047420021420001320020a00001f200101024f20102f201064200005043003df2010c43003e42003d5342003e25a1000442003e42003d50a00001f20043003c42003ca1000542003c420023420001320020a00001f200f2010642000050430032f2010b4300334200325342003325a1000442003342003250a00001f200430031420031a10005420031420022420001320020a00001f200f2010642000050430027f2010a4300284200275342002825a1000442002842002750a00001f200430026420026a10005420026420021420001320020a00001f20010102621 --dry-run > initial-prepared.hex
   ```

   Let's unpack this command a little. It asks the `demo` wallet to format a transaction that

   - _sends all inputs back to `demo`'s own address_ --- denoted by the lack of a `--to` argument
   - _spends the covenant-locked coin_ we just created earlier --- the out-of-wallet coin `630024d8f526fa15cb53d40aec440e63ae32c636229696e312e66f311fee7c6b-0`
   - _adds additional fees_ to cover the signatures from Alice, Bob, and Charlie we're gonna add later. A "fee ballast" of 1000 essentially instructs the wallet to calculate the fee level assuming the transaction will grow by at most 1000 bytes, plenty for most multisignature wallets.
     > **Note:** We cannot add fees _after_ we obtain the signatures, because A, B, and C's signatures need to sign off on a transaction that already has a fee.
   - _supplies the content of the covenant_ inline as a huge hex string

     > **Note:** This is because when we created the coin, we locked it with the covenant through referring to the _covenant hash_. When we spend the coin, we then need to supply the actual contents of the covenant. This construct, borrowed from Bitcoin's ["pay-to-script-hash"](https://en.bitcoin.it/wiki/Pay_to_script_hash), ensures that bulky covenants do not burden the coin state (and instead only need to be archived in the blockchain history, which does not need frequent access).
     >
     > This does mean that any transaction that wants to spend a coin locked by a covenant must first obtain the full MelVM code for that covenant.

   - _runs in "dry run" mode_, which instead of attempting to send the transaction, merely prepares it and dumps it out as a hex string.
   - _redirects the output to `initial-prepared.hex`_, using `bash` syntax.

   After running this command, `initial-prepare.hex` contains a large hex string representing the prepared transaction:

   ```
   0001630024d8f526fa15cb53d40aec440e63ae32c636229696e312e66f311fee7c6b000217f4e34222378b900a0988c3b852bb05c9bf2916570bb845ee330ab24772122dfc28effa02016d0017f4e34222378b900a0988c3b852bb05c9bf2916570bb845ee330ab24772122dfc29effa02016d00fbaf020251420009f100000000000000000000000000000000000000000000000000000000000000064200005050f0205df45d38f0727ac106b053986c8f8696b78c57c3adf2f23c75f67144a74050ce420001320020fb0202f020b5c7302463eda5f951d29ed1c6f7e9c7056628fb967b92653e0f9a1c15b6ad7e430021f02037290798b218b2312faa66f91191f6180c530df0fabf3f2791417e48d69d9b8b430022f020315a04b0ca34049f5f47621da1b72cdeadbf51c5aacbbe14d88a8b9d77229deb430023f20102f201064200005043005ef2010c43005f42005e5342005f25a1000442005f42005e50a00001f20043005d42005da1000542005d420023420001320020a00001f200f2010642000050430053f2010b4300544200535342005425a1000442005442005350a00001f200430052420052a10005420052420022420001320020a00001f200f2010642000050430048f2010a4300494200485342004925a1000442004942004850a00001f200430047420047a10005420047420021420001320020a00001f200101024f20102f201064200005043003df2010c43003e42003d5342003e25a1000442003e42003d50a00001f20043003c42003ca1000542003c420023420001320020a00001f200f2010642000050430032f2010b4300334200325342003325a1000442003342003250a00001f200430031420031a10005420031420022420001320020a00001f200f2010642000050430027f2010a4300284200275342002825a1000442002842002750a00001f200430026420026a10005420026420021420001320020a00001f20010102621000140dcf2921c7651347bdddf0644e4a3511327733acc444ce73eb59f9e5e7c71424325b271db1b24776ff32085baba606cd8195a0a5b071a0a3577779d55212bbe02
   ```

2. **Signing the transaction**: we use the `themelio-crypttool` tool (which can be installed with `cargo install themelio-crypttool`) to manually sign the transaction. We sign with the three private keys (again, feel free to omit one, as this is a 2-of-3 multisig)

   - Sign with Alice's private key, saving the result to `signed-alice.hex`:
     ```
     $ themelio-crypttool sign-tx $(cat initial-prepared.hex) --posn 10 --secret 85f884fb8400117a29ef65b3a3ace00e7a6e1dde65479361c67e564fd00de417b5c7302463eda5f951d29ed1c6f7e9c7056628fb967b92653e0f9a1c15b6ad7e > signed-alice.hex
     ```
   - Sign with Bob's private key, saving the result to `signed-bob.hex`:
     ```
     $ themelio-crypttool sign-tx $(cat signed-alice.hex) --posn 11 --secret 820384db2898cb4abaa6662de1087c955a7e08b13abcb69c8e7941a69ff372e437290798b218b2312faa66f91191f6180c530df0fabf3f2791417e48d69d9b8b > signed-alice-bob.hex
     ```
   - Sign with Charlie's private key, saving the result to `signed-bob.hex`:
     ```
     $ themelio-crypttool sign-tx $(cat signed-alice-bob.hex) --posn 12 --secret 35f29230304c0f0e0c730758caabd121a312dec4c1c0b859a75b1b59513ab4ca315a04b0ca34049f5f47621da1b72cdeadbf51c5aacbbe14d88a8b9d77229deb > signed-alice-bob-charlie.hex
     ```

3. **Sending the transaction**: we use the `send-raw` verb of `melwallet-cli`:
   ```
   $ melwallet-cli send-raw -w demo $(cat signed-alice-bob-charlie.hex)
   TRANSACTION RECIPIENTS
   Address                                                 Amount         Additional data
   t92zte6gh26y5s02g9h31vgmnv0q4vya8paw5vghfe6c5b4hvj28pg  49.999656 MEL  ""
   t92zte6gh26y5s02g9h31vgmnv0q4vya8paw5vghfe6c5b4hvj28pg  49.999657 MEL  ""
   (network fees)                                         0.000687 MEL
   Proceed? [y/N]
   y
   Transaction hash:  ac810425e235f52cb252bc3a0218688606c012edb29b5b75755019209a4e0812
   (wait for confirmation with melwallet-cli wait-confirmation -w demo ac810425e235f52cb252bc3a0218688606c012edb29b5b75755019209a4e0812)
   ```
   After the transaction confirms, we see [on Melscan](https://scan-testnet.themelio.org/blocks/50772/ac810425e235f52cb252bc3a0218688606c012edb29b5b75755019209a4e0812) that the 100 MEL locked up (less fees) is sent back to `demo`'s address.

## Deploying on the mainnet

Following the same steps with a mainnet `melwalletd` and mainnet money, you can deploy a 2-of-3 multisignature wallet that locks "real money" as well! One warning though --- use "Alice", "Bob", and "Charlie"'s private keys at your own risk 😉
