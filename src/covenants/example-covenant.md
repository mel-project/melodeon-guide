Let's make a slightly more advanced covenant now. Suppose you want to store
some wealth in cold storage and you want to give a backup key to your friend in
case you lose your key. Basically, you want either of two pre-specified public
keys to be able to spend a coin.

To accomplish this we'll need to make use of the environment variables loaded
at runtime (see a refresher on [environment variables]()). We'll use
`env_spender_tx_hash()` to get the hash of the transaction spending the coin at
runtime.

We'll use the hash to verify the signature provided in the transaction
(remember transactions contain a list of signature, see the [structure of
a transaction]()). We get the actual transaction object with
`env_spender_tx()`.

We provide the two public keys which we want to be allowed to spend the coin.

Now for the logical parts. Notice spender.sigs is an unknown size array,
meaning in order to reference it we need to specify an `unsafe` operation. We
cast the sig of "unknown" size to a standard 32 bytes.

Finally, first try to verify the signature we loaded matches the first public
key specified. If it verifies, return 1. If not, try the second public key.
`pk2`.

```python
let txhash = env_spender_tx_hash(),
    spender = env_spender_tx(),
    pk1 = x"f2d6ba8084da23c40bd5bf01d477432d9370583116ddc6f362a60ca6cd0741e0",
    pk2 = x"140aac9f73036097d21c70cfd4c11032754b88461fdbdd9eaa4abe093dbde31b" in

let sig = vref(spender.sigs, 0) in sig &&
    if !ed25519_verify(txhash, pk1, sig):
        ed25519_verify(txhash, pk2, sig)
    else
        1
```

By default melorun provides an empty transaction to run a script in. In our
case the script expects the spender transaction to have a specific signature.
So we will specify our own environment in json and pass it to melorun as
a command-line input.

```bash
melorun example.melo -e tx.json
```

```json
{
    "spender_tx": {
        "kind":0,
        "inputs":[],
        "outputs":[],
        "fee":0,
        "covenants":[],
        "data":"",
        "sigs":[
            "7ee7e1c88eddd41bc7b2ac4248c148ed0f1f23dbf191d9d01d33c4dbde2386ff4c29534f392ac1f6315078939905b80b410ed288c80778864699a6891bb7e509",
            "0ebbf479b80ad2888df1ee12ad84a2d2c0ed9efbb367ea3d1f98d83abb2c3a350c1e6c8c69d32e5ee35f9030a628dc40d529ed194b9c3d2fcafe040d5a9ce209"
        ]
    },
    "environment": {
        "spender_index": 0
    }
}
```
