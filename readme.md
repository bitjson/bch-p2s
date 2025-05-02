# CHIP-2024-12 P2S: Pay to Script

        Title: Pay to Script
        Type: Standards
        Layer: Consensus
        Maintainer: Jason Dreyzehner
        Status: Draft
        Initial Publication Date: 2024-12-12
        Latest Revision Date: 2024-12-12
        Version: 1.0.0

## Summary

This proposal makes Pay to Script (P2S) outputs standard and increases the length limits on token commitments and standard unlocking bytecode.

These changes improve wallet ecosystem safety, simplify contract design, and reduce transaction sizes for many vault, multi-party covenant, and decentralized financial applications.

## Motivation & Benefits

- **Improve wallet ecosystem safety** - Many kinds of contracts should not by "randomly payable" by naive wallets, but because P2SH contracts always have payable addresses, it's easy for confused users to mistakenly send funds to unrecoverable locations. This proposal gives contract authors a new primitive that is both more byte efficient and safer for end users.

- **Simplify contracts** - This proposal avoids the need for intermediate construction of P2SH contracts in a variety of use cases, simplifying inspection and modification of the active bytecode, and avoiding wasted bytes and hashing – both in packing (validating the P2SH address within the covenant) and unpacking (at spending time) the P2SH contract. This reduces the overhead of most covenant systems by at least 34 bytes per output.

## Deployment

Deployment of this specification is proposed for the May 2026 upgrade.

- Activation is proposed for `1763208000` MTP, (`2025-11-15T12:00:00.000Z`) on `chipnet`.
- Activation is proposed for `1778846400` MTP, (`2026-05-15T12:00:00.000Z`) on the BCH network (`mainnet`), `testnet3`, `testnet4`, and `scalenet`.

## Technical Specification

Standard output validation is relaxed to allow Pay to Script (P2S) outputs, and the limits on maximum token commitment length and maximum standard unlocking bytecode length are increased.

### Locking Bytecode Length

The existing output standardness validation requiring spendable outputs to match a known pattern (<abbr title="Pay to Public Key">P2PK</abbr>, <abbr title="Pay to Public Key Hash">P2PKH</abbr>, <abbr title="Pay to Script Hash (20 bytes)">P2SH20</abbr>, <abbr title="Pay to Script Hash (32 bytes)">P2SH32</abbr>, <abbr title="Data-Carrier Outputs (A.K.A. OP_RETURN Outputs)">OP_RETURN</abbr>, or <abbr title="Bare Multi-Signature">BMS</abbr>) is replaced by:

1. **Length check**: the locking bytecode of standard outputs must have a length less than or equal to `201`, the maximum length of currently standard bare multi-signature (BMS) outputs. See [Rationale: Selection of Maximum Locking Bytecode Length](#selection-of-maximum-locking-bytecode-length).
2. **Data-carrier validation**: locking bytecode which exceeds the length check but matches the existing data-carrier pattern (`OP_RETURN`) is standard if accepted by the existing data-carrier validation (the cumulative 223-byte limit across all transaction outputs).

Note that UTXO standardness checks must also retain the additional allowance for bare multi-signature patterns; these remain nonstandard to create beyond the existing 3-key limit, but UTXOs exceeding the limit remain spendable in standard transactions (matching the existing behavior for these cases).

### Token Commitment Length

The limit on maximum length of token commitments is raised from `40` bytes to `128` bytes. See [Rationale: Selection of Maximum Token Commitment Length](#selection-of-maximum-token-commitment-length).

### Unlocking Bytecode Length

The limit on maximum standard input bytecode length (A.K.A. `MAX_TX_IN_SCRIPT_SIG_SIZE` – 1,650 bytes) is removed such that the maximum unlocking bytecode length is equal for both standard and consensus validation: 10,000 bytes (A.K.A. `MAX_SCRIPT_SIZE`). See [Rationale: Unification of Standard and Consensus Unlocking Bytecode Length](#unification-of-standard-and-consensus-unlocking-bytecode-length).

## Rationale

This section documents design decisions made in this specification.

### Selection of Maximum Locking Bytecode Length

This proposal simplifies validation of output standardness by eliminating the existing, complex, bytecode pattern-matching behavior and replacing it primarily by a length check: standard outputs following activation of this proposal must have bytecode no longer than the longest of the currently-standard patterns.

The longest standard locking bytecode under existing validation rules is an M-of-3 Bare Multi-Signature (BMS) output with uncompressed public keys, requiring 201 bytes.<sup>1</sup>

1. See VMB test ID `20d42l`.

### Selection of Maximum Token Commitment Length

In the same way that this proposal enables contracts to avoid wrapping contract code in an otherwise unnecessary hash (wasting the byte length of the hash plus stack manipulation bytecode across both setup and usage transactions), this proposal also extends the maximum allowable token commitment length from `40` bytes to `128` bytes, avoiding the same variety of waste within token commitments.

The `128` byte limit is selected to remain below the existing data-carrier limit (`220` bytes), while extending the usefulness of the commitment field (without added waste or complexity from a hash-unwrapping step) for notable use cases: bilinear pairing-based accumulators (e.g. `BLS12-381` KZG commitments require ~48 bytes compressed or ~96 bytes uncompressed), two 64-byte Schnorr signatures, four 32-byte (OP_HASH256, birthday-collision resistant) hashes, or six 20-byte (OP_HASH160) hashes.

### Unification of Standard and Consensus Unlocking Bytecode Length

This proposal eliminates the separate standardness limit for input bytecode length (A.K.A. `MAX_TX_IN_SCRIPT_SIG_SIZE`; 1,650 bytes) such that the existing consensus limit (A.K.A. `MAX_SCRIPT_SIZE`; 10,000 bytes) is enforced for both transaction relay and block validation.

Following the [VM Limits CHIP](https://github.com/bitjson/bch-vm-limits), contract length is no longer relevant to worst-case transaction or block validation performance. As standard transactions can include many inputs – up to the maximum standard transaction byte length (A.K.A. `MAX_STANDARD_TX_SIZE`; 100,000 bytes) – a lower per-input standardness limit offers no additional safety to the network while inconveniencing applications with larger contiguous data requirements.

For example, many zero-knowledge and post-quantum cryptographic systems require proofs larger than 1,650 bytes; with a lower per-input standardness limit, these proofs would need to be broken apart into multiple inputs and/or outputs, requiring the development of unusual standards, significant waste in data manipulation bytecode, and unnecessary contract complexity.

### Exclusion of P2S CashAddress Format

While it is technically possible to define an arbitrary-length CashAddress format for conveying Pay-to-Script "addresses" (though not initially designed for variable-length payloads), **this proposal intentionally excludes such a format to improve wallet ecosystem safety and compatibility**.

Pay to Script Hash (P2SH) already provides a safe, well-established pattern for sharing user-payable contract addresses. Notably, data usage within transactions (and associated mining fee costs) are fixed across all P2SH20 and P2SH32 addresses(respectively), with the receiver responsible for the potentially-variable fees required to spend from P2SH addresses.

On the other hand, Pay to Script (P2S) "addresses" would create significant new risks of loss for many users: it is not guaranteed (or even commonly expected) that funds unexpectedly payed to a P2S contract will remain recoverable. As P2S contracts tend to be most useful in vault, multi-party covenant, and decentralized financial applications, there will often be no counterparty to whom refund requests could even be made. **The non-existence of P2S addresses is instead a useful feature**: developers can intentionally avoid using P2SH in contexts where end-users should not "manually" send payments (e.g. by copying the P2SH address from a block explorer or wallet history), reducing the chance of losses or poor user experiences due to user error.

Note that payment and wallet protocols can still be designed to enable cross-wallet payments to P2S contracts; the exclusion of a simplified "address" format serves only to reduce the risk of losses due to mismatches between user expectations and technical realities.

While block explorer and other visualization software may represent P2S contracts by rendering their encoded bytecode in hexadecimal or another format, **typical wallets should never support payments to arbitrary/unknown P2S contracts via such representations**. Instead, wallets should utilize a template system, collaborative signing protocol, or other high-level scheme to communicate and authenticate contracts prior to locking funds in new outputs using those contracts.

## Implementations

Please see the following implementations for additional examples and test vectors:

- **JavaScript/TypeScript**:
  - [Libauth](https://github.com/bitauth/libauth) – An ultra-lightweight, zero-dependency JavaScript library for Bitcoin Cash. [Branch `next`](https://github.com/bitauth/libauth/tree/next).
  - [Bitauth IDE](https://github.com/bitauth/bitauth-ide) – An online IDE for bitcoin (cash) contracts. [Branch `next`](https://github.com/bitauth/bitauth-ide/tree/next).

## Feedback & Reviews

- [Pay to Script CHIP Issues](https://github.com/bitjson/bch-p2s/issues)
- [`CHIP 2024-12 P2S: Pay to Script` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/chip-2024-12-p2s-pay-to-script/1451)

## Changelog

This section summarizes the evolution of this document.

- **v1.0.0 – 2024-12-12**
  - Initial publication

## Copyright

This document is placed in the public domain.
