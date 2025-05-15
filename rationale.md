# Rationale

This section documents design decisions made in this specification.

<details>

<summary><strong>Table of Contents</strong></summary>

- [Rationale](#rationale)
  - [Relationship Between the Modified Limits](#relationship-between-the-modified-limits)
  - [Selection of Maximum Locking Bytecode Length](#selection-of-maximum-locking-bytecode-length)
  - [Selection of Maximum Token Commitment Length](#selection-of-maximum-token-commitment-length)
  - [Unification of Standard and Consensus Unlocking Bytecode Length](#unification-of-standard-and-consensus-unlocking-bytecode-length)
  - [Exclusion of P2S CashAddress Format](#exclusion-of-p2s-cashaddress-format)
  - [Non-Impact on Data-Carrier Outputs (A.K.A. OP_RETURN Outputs)](#non-impact-on-data-carrier-outputs-aka-op_return-outputs)
  - [Non-Reliance on Miscalculated 220-Byte Limit](#non-reliance-on-miscalculated-220-byte-limit)

</details>

### Relationship Between the Modified Limits

This proposal modifies 3 limits: [Locking Bytecode Length](./readme.md#locking-bytecode-length), [Token Commitment Length](./readme.md#token-commitment-length), and [Unlocking Bytecode Length](./readme.md#unlocking-bytecode-length). The currently-inconsistent behavior of these limits each produce the same unintended effect for contract authors: they force one or more unnecessary hashes to be placed into other part(s) of the transaction where they don’t logically belong, wasting storage/bandwidth for no gain to either the user or the wider network.

This proposal recommends the most conservative change to each constant to make these limits logically consistent and eliminate some of today’s most common sources of waste in transaction sizes and contract system design.

### Selection of Maximum Locking Bytecode Length

This proposal simplifies validation of output standardness by eliminating the existing, complex, bytecode pattern-matching behavior and replacing it primarily by a length check: standard outputs following activation of this proposal must have bytecode no longer than the longest of the currently-standard patterns.

The longest standard locking bytecode under existing validation rules is an M-of-3 Bare Multi-Signature (BMS) output with uncompressed public keys, requiring 201 bytes.<sup>1</sup>

1. See VMB test ID `20d42l`.

### Selection of Maximum Token Commitment Length

In the same way that this proposal enables contracts to avoid wrapping contract code in an otherwise unnecessary hash (wasting the byte length of the hash plus stack manipulation bytecode across both setup and usage transactions), this proposal also extends the maximum allowable token commitment length from `40` bytes to `128` bytes, avoiding the same variety of waste within token commitments.

The `128` byte limit is the largest power of 2 below the existing data-carrier limit (`220` bytes), extending the usefulness of the commitment field (without added waste or complexity from a hash-unwrapping step) for notable use cases: bilinear pairing-based accumulators (e.g. `BLS12-381` KZG commitments require ~48 bytes compressed or ~96 bytes uncompressed), two 64-byte Schnorr signatures, four 32-byte (OP_HASH256, birthday-collision resistant) hashes, or six 20-byte (OP_HASH160) hashes.

### Unification of Standard and Consensus Unlocking Bytecode Length

This proposal eliminates the separate standardness limit for input bytecode length (A.K.A. `MAX_TX_IN_SCRIPT_SIG_SIZE`; 1,650 bytes) such that the existing consensus limit (A.K.A. `MAX_SCRIPT_SIZE`; 10,000 bytes) is enforced for both transaction relay and block validation.

Following the [VM Limits CHIP](https://github.com/bitjson/bch-vm-limits), contract length is no longer relevant to worst-case transaction or block validation performance. As standard transactions can include many inputs – up to the maximum standard transaction byte length (A.K.A. `MAX_STANDARD_TX_SIZE`; 100,000 bytes) – a lower per-input standardness limit offers no additional safety to the network while inconveniencing applications with larger contiguous data requirements.

For example, many zero-knowledge and post-quantum cryptographic systems require proofs larger than 1,650 bytes; with a lower per-input standardness limit, these proofs would need to be broken apart into multiple inputs and/or outputs, requiring the development of unusual standards, significant waste in data manipulation bytecode, and unnecessary contract complexity.

### Exclusion of P2S CashAddress Format

While it is technically possible to define an arbitrary-length CashAddress format for conveying Pay-to-Script "addresses" (though not initially designed for variable-length payloads), **this proposal intentionally excludes such a format to improve wallet ecosystem safety and compatibility**.

Pay to Script Hash (P2SH) already provides a safe, well-established pattern for sharing user-payable contract addresses. Notably, data usage within transactions (and associated mining fee costs) are fixed across all P2SH20 and P2SH32 addresses(respectively), with the receiver responsible for the potentially-variable fees required to spend from P2SH addresses.

On the other hand, Pay to Script (P2S) "addresses" would create significant new risks of loss for many users: it is not guaranteed (or even commonly expected) that funds unexpectedly paid to a P2S contract will remain recoverable. As P2S contracts tend to be most useful in vault, multi-party covenant, and decentralized financial applications, there will often be no counterparty to whom refund requests could even be made. **The non-existence of P2S addresses is instead a useful feature**: developers can intentionally avoid using P2SH in contexts where end-users should not "manually" send payments (e.g. by copying the P2SH address from a block explorer or wallet history), reducing the chance of losses or poor user experiences due to user error.

Note that payment and wallet protocols can still be designed to enable cross-wallet payments to P2S contracts; the exclusion of a simplified "address" format serves only to reduce the risk of losses due to mismatches between user expectations and technical realities.

While block explorer and other visualization software may represent P2S contracts by rendering their encoded bytecode in hexadecimal or another format, **typical wallets should never support payments to arbitrary/unknown P2S contracts via such representations**. Instead, wallets should utilize a template system, collaborative signing protocol, or other high-level scheme to communicate and authenticate contracts prior to locking funds in new outputs using those contracts.

### Non-Impact on Data-Carrier Outputs (A.K.A. OP_RETURN Outputs)

This proposal carefully avoids impact to the costs and incentives of "on-chain data storage" use cases. Because the existing 220-byte content limit on [`OP_RETURN` Data Carrier Outputs](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#op_return-data-carrier-outputs) was originally selected [due to a miscalculation](#non-reliance-on-miscalculated-220-byte-limit), there is [insufficient consensus among stakeholders](https://bitcoincashresearch.org/t/chip-2024-12-p2s-pay-to-script/1451/3?u=bitjson) as to whether or not the limit should exist, and future proposals can separately increase or remove this limit, this proposal considers changes impacting the status quo to be out of scope.

### Non-Reliance on Miscalculated 220-Byte Limit

The 220-byte limit was originally established to, "disincentivize the use of other methods to embed data into the chain" ([commit `cbf44109`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/cbf4410912f6512e481f15270329683d4d4378d4)). However, this rationale was based on [earlier analysis](https://github.com/bitcoin/bitcoin/issues/12033) which omitted [more efficient strategies](https://bitcoincashresearch.org/t/raising-the-520-byte-push-limit-201-operation-limit/282/8?u=bitjson) for recording arbitrary data in standard transactions. As such, this proposal avoids modifying or establishing any new limits based on the incorrect `220` byte constant; future proposals can separately address this topic.
