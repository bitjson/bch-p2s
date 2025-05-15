# Evaluation of Alternatives

Potential alternatives to this proposal are reviewed below.

<details>

<summary><strong>Table of Contents</strong></summary>

- [Evaluation of Alternatives](#evaluation-of-alternatives)
  - [Locking Bytecode Standardness Alternatives](#locking-bytecode-standardness-alternatives)
    - [Status Quo: Pattern Matching](#status-quo-pattern-matching)
    - [Alternative: Reduced Locking Bytecode Standardness Limit](#alternative-reduced-locking-bytecode-standardness-limit)
    - [Alternative: Increased Locking Bytecode Standardness Limit](#alternative-increased-locking-bytecode-standardness-limit)
      - [Alternative: 220-Byte Locking Bytecode Standardness Limit](#alternative-220-byte-locking-bytecode-standardness-limit)
      - [Alternative: 10,000-Byte Locking Bytecode Limit](#alternative-10000-byte-locking-bytecode-limit)
      - [Alternative: 100,000-Byte Locking Bytecode Limit](#alternative-100000-byte-locking-bytecode-limit)
  - [Token Commitment Length Alternatives](#token-commitment-length-alternatives)
    - [Status Quo: 40-Byte Token Commitments](#status-quo-40-byte-token-commitments)
    - [Alternative: 201-Byte Token Commitments](#alternative-201-byte-token-commitments)
    - [Alternative: 220-Byte Token Commitments](#alternative-220-byte-token-commitments)
    - [Alternative 10,000-Byte Token Commitments](#alternative-10000-byte-token-commitments)
    - [Alternative 100,000-Byte Token Commitments](#alternative-100000-byte-token-commitments)
  - [Unlocking Bytecode Length Alternatives](#unlocking-bytecode-length-alternatives)
    - [Status Quo: 1,650-Byte Unlocking Bytecode Standardness Limit](#status-quo-1650-byte-unlocking-bytecode-standardness-limit)
    - [Alternative: 100,000-Byte Unlocking Bytecode Limit](#alternative-100000-byte-unlocking-bytecode-limit)

</details>

## Locking Bytecode Standardness Alternatives

This proposal simplifies standard validation of locking bytecode to eliminate most of the existing pattern matching, replacing it with a simple length check. See [Technical Specification: Locking Bytecode Length](./readme.md#locking-bytecode-length).

### Status Quo: Pattern Matching

The existing, pattern-matching standardness validation for locking bytecode – particularly the requirement that all custom contracts use the Pay-to-Script-Hash (P2SH) pattern – unnecessarily complicates contract systems, exposing users to additional risks (especially of [erroneous payments to P2SH addresses](./rationale.md#exclusion-of-p2s-cashaddress-format)) and increased usage costs (by requiring [larger or additional transactions](./rationale.md#relationship-between-the-modified-limits)).

The P2SH wrapping requirement also makes a variety of downstream systems more expensive and/or less private by requiring exhaustive tracking and storage of P2SH preimages, even for fully public covenant systems (where the preimage was already exposed by the previous transaction). This complicates light client support for contract systems, often requiring significant local storage and/or trusted servers, additional network connectivity, and more complex wallet backup and recovery solutions.

### Alternative: Reduced Locking Bytecode Standardness Limit

This proposal establishes the simplified length-based limit at precisely the existing maximum-length standard locking bytecode: 201 bytes (see [Rationale: Selection of Maximum Locking Bytecode Length](./rationale.md#selection-of-maximum-locking-bytecode-length)). This maximizes flexibility for contract developments while avoiding any new risk to memory or bandwidth usage.

Alternatively, this proposal could establish a new limit below the current maximum length. However, a lower limit would not be protective in any way, and would significantly increase the complexity of standard validation by requiring more of the existing pattern matching behavior to be retained. This would also have a marginal negative impact on performance, as the requisite behavior is exercised in the common case: every output validation.

### Alternative: Increased Locking Bytecode Standardness Limit

As another alternative, this proposal could establish the simplified limit at a length longer than 201 bytes. Potential options include 220 bytes (the current maximum [Data-Carrier Output Length](./rationale.md#non-impact-on-data-carrier-outputs-aka-op_return-outputs)), 10,000 bytes (the current bytecode length limit, A.K.A. `MAX_SCRIPT_SIZE`), and 100,000 bytes (the maximum standard transaction byte length, A.K.A. `MAX_STANDARD_TX_SIZE`).

#### Alternative: 220-Byte Locking Bytecode Standardness Limit

It may seem that this proposal could further simplify validation by established the locking bytecode standardness limit at `220` bytes, eliminating the [special case for data-carrier validation](./readme.md#locking-bytecode-length). However, data-carrier outputs are excluded from dust limits (data-carrier outputs may have a BCH value of `0`, while all other outputs must have a minimum value determined by their encoded length), so increasing the standard length limit does not eliminate the need for data-carrier validation. (Note also the additional [allowance for multi-signature spends](./readme.md#retained-allowance-for-multi-signature-spends) which must be retained regardless of the maximum standard locking bytecode length.)

Additionally, because this alternative would represent an increase from today's practical limit of `201` bytes, it could also plausibly entail new risks to memory or bandwidth usage, requiring additional analysis and review. Specifically, because the existing practical limit is shorter than the data-carrier limit (at least for contiguous data), a change which makes them equivalent could plausibly impact [costs and incentives surrounding some "data storage" use cases](./rationale.md#non-impact-on-data-carrier-outputs-aka-op_return-outputs).

Finally, even if the scope of this proposal were expanded to potentially impact "data storage" use cases, it must be noted that the `220` byte limit is [the result of a miscalculation](./rationale.md#non-reliance-on-miscalculated-220-byte-limit). At minimum, an expanded alternative proposal attempting to merge the data-carrier and standard locking bytecode length limits would need to calculate a corrected limit based on the original rationale (if the limit were to be retained at all). In practice, these two limits are simply poor candidates for a merge: standard locking bytecode length has potential implications for validation performance and UTXO set growth, while data-carrier limits have little potential for impact in those areas.

#### Alternative: 10,000-Byte Locking Bytecode Limit

Alternatively, this proposal could remove the locking bytecode standardness limit such that the maximum locking bytecode length is equal for both standard and consensus validation: 10,000 bytes (A.K.A. `MAX_SCRIPT_SIZE`). This would entail many of the benefits of unifying standard and consensus unlocking bytecode length (see [Rationale: Unification of Standard and Consensus Unlocking Bytecode Length](rationale.md#unification-of-standard-and-consensus-unlocking-bytecode-length)).

However, any proposed increase in the maximum standard locking bytecode length could arguably impact the costs and incentives surrounding "data storage" behaviors, UTXO set growth, and other node operation costs. (To justify the additional scope/review cost, such a proposal should likely continue to at least a [100,000-Byte Locking Bytecode Limit](#alternative-100000-byte-locking-bytecode-limit).) To minimize scope, this proposal focuses primarily on resolving inconsistencies within the existing practical limits and, where possible, leaving increases to future proposals.

#### Alternative: 100,000-Byte Locking Bytecode Limit

Alternatively, this proposal could both remove the locking bytecode standardness limit and raise the consensus limit to 100,000 bytes, the current per-transaction limit (minus minimal overhead) for cumulative Pay-to-Script-Hash (P2SH) redeem bytecode; this limit follows from the limit on maximum standard transaction length (A.K.A. `MAX_STANDARD_TX_SIZE`).

This alternative would significantly improve the efficiency of many zero-knowledge and post-quantum cryptographic systems (particularly if coupled with [read-only inputs](https://github.com/bitjson/bch-txv5#read-only-inputs)), where signatures alone can easily extend into the kilobytes, and deduplication of contract bytecode could significantly reduce overall network storage and bandwidth usage. However, as [described in `Alternative: 10,000-Byte Locking Bytecode Limit`](#alternative-10000-byte-locking-bytecode-limit), this proposal minimizes scope by avoiding increases to existing practical limits where possible.

## Token Commitment Length Alternatives

This proposal increases the maximum length of token commitments from 40 bytes to 128 bytes. See [Technical Specification: Token Commitment Length](./readme.md#token-commitment-length) and [Rationale: Selection of Maximum Locking Bytecode Length](./rationale.md#selection-of-maximum-token-commitment-length).

### Status Quo: 40-Byte Token Commitments

The existing 40-byte limit on token commitment length was [partially justified](https://github.com/cashtokens/cashtokens/issues/23) by the inflexibility of the existing pattern-matching standardness validation for locking bytecode: because commitments could carry arbitrary data (and locking bytecode patterns were restricted), it was plausible that longer contiguous commitments could impact the [costs and incentives surrounding some "data storage" use cases](./rationale.md#non-impact-on-data-carrier-outputs-aka-op_return-outputs). However, this proposal's relaxation of locking bytecode standardness obviates these theoretical risks: if standard locking bytecode can carry up to ~200 bytes of contiguous, arbitrary data, token commitments are no longer plausibly competitive for such uses. (Note also that token commitments also require greater encoding overhead, so for cost reasons, applications are more likely to rely on token commitments only for use cases requiring token commitment functionality rather than simple data publication.)

Further, the 40-byte token commitment length limit requires the same kind of [unnecessary, hash-wrapping behavior](./rationale.md#relationship-between-the-modified-limits), limiting the possible designs available to contract authors, complicating contract systems and applications, and increasing transaction costs (with intermediate transactions and/or wasted bytes).

As 1) the CashTokens upgrade was [designed to ensure future upgradability of token commitment lengths](https://cashtokens.org/docs/spec/chip#token-prefix-validation), 2) the original rationale for the 40-byte limit is obviated by this upgrade, and 3) the current 40-byte limit unnecessary limits applications and increases transaction costs, this proposal considers a minimal increase to be prudent (see [Rationale: Selection of Maximum Token Commitment Length](./rationale.md#selection-of-maximum-token-commitment-length)).

### Alternative: 201-Byte Token Commitments

Alternatively, this proposal could increase the maximum length of token commitments to `201` bytes, matching the simplified [locking bytecode standardness limit](./readme.md#locking-bytecode-length). This would further increase the efficiency and flexibility available to contract authors; however, matching or closely approaching the locking bytecode standardness limit could reintroduce [the same uncertainties addressed by the status quo limit](#status-quo-40-byte-token-commitments) regarding both ["data storage" costs and incentives](./rationale.md#non-impact-on-data-carrier-outputs-aka-op_return-outputs) and UTXO set growth. Instead, this proposal reduces scope to simplify review: the maximum token commitment length is increased to a limit that remains well below the other methods of contiguous data publication (see [Rationale: Selection of Maximum Token Commitment Length](./rationale.md#selection-of-maximum-token-commitment-length)).

### Alternative: 220-Byte Token Commitments

Alternatively, this proposal could increase the maximum length of token commitments to `220` bytes, matching the current maximum [Data-Carrier Output Length](./rationale.md#non-impact-on-data-carrier-outputs-aka-op_return-outputs). This alternative would further increase the efficiency and flexibility available to contract authors, but it would incur both the increased scope/review cost described [in `Alternative: 201-Byte Token Commitments`](#alternative-201-byte-token-commitments) and misalign the token commitment length limit with the [miscalculated 220-byte data-carrier limit](./rationale.md#non-reliance-on-miscalculated-220-byte-limit). (See also: [Alternative: 220-Byte Locking Bytecode Standardness Limit](#alternative-220-byte-locking-bytecode-standardness-limit).)

### Alternative 10,000-Byte Token Commitments

Alternatively, this proposal could increase the maximum length of token commitments to 10,000 bytes, matching the current consensus locking/unlocking/redeem bytecode length (A.K.A. `MAX_SCRIPT_SIZE`).

This approach would maximize efficiency and flexibility for contract design by removing the protocol bias favoring locking bytecode. (Because commitments enable contract systems to [extract variable data from contract bytecode](https://github.com/cashtokens/cashtokens/issues/23#issuecomment-1181218075), often significantly improving the efficiency of the contract system as a whole.)

However, a significant increase in maximum token commitment length would increase the scope and review cost of this proposal (as [described in `Alternative: 201-Byte Token Commitments`](#alternative-201-byte-token-commitments)).
(To justify the additional scope/review cost, such a proposal should likely continue to at least a [100,000-Byte Token Commitment Length Limit](#alternative-100000-byte-token-commitments).) Instead, this proposal minimizes scope, leaving significant increases for future proposals.

### Alternative 100,000-Byte Token Commitments

Alternatively, this proposal could increase the maximum length of token commitments to 100,000 bytes, matching the approximate practical per-transaction, cumulative limit on token commitment data (minus overhead, across multiple inputs/outputs). Such an increase should likely be paired with an equivalent increase in locking/unlocking/redeem bytecode length limits (see: [Alternative: 100,000-Byte Locking Bytecode Limit](#alternative-100000-byte-locking-bytecode-limit) and [Alternative: 100,000-Byte Unlocking Bytecode Limit](#alternative-100000-byte-unlocking-bytecode-limit)) to avoid biasing the protocol toward or against [extracting variable data from contract bytecode](https://github.com/cashtokens/cashtokens/issues/23#issuecomment-1181218075).

However, as [described in `Alternative: 201-Byte Token Commitments`](#alternative-201-byte-token-commitments), this proposal minimizes scope and simplifies review by avoiding such a significant, single-output limit increase.

## Unlocking Bytecode Length Alternatives

This proposal removes the additional standardness limit on unlocking bytecode length such that unlocking bytecode length is limited at 10,000 bytes for both standard and consensus (block) validation. See [Technical Specification: Unlocking Bytecode Length](./readme.md#unlocking-bytecode-length) and [Rationale: Unification of Standard and Consensus Unlocking Bytecode Length](./rationale.md#unification-of-standard-and-consensus-unlocking-bytecode-length).

### Status Quo: 1,650-Byte Unlocking Bytecode Standardness Limit

Following the [VM Limits CHIP](https://github.com/bitjson/bch-vm-limits), the existing per-input, 1,650-byte limit on unlocking bytecode length unnecessarily limits contract systems (requiring overhead-incurring workarounds) at no gain to the network: the additional standardness limit does not improve worst case computation/validation costs, storage, or bandwidth usage, and the additional overhead required in contract systems has a marginal negative impact on storage and bandwidth usage.

This proposal simply removes the standardness limit, unifying the field's standard validation with the existing consensus validation. See [Rationale: Unification of Standard and Consensus Unlocking Bytecode Length](./rationale.md#unification-of-standard-and-consensus-unlocking-bytecode-length).

### Alternative: 100,000-Byte Unlocking Bytecode Limit

Alternatively, this proposal could both remove the unlocking bytecode standardness limit and raise the consensus limit to 100,000 bytes, the current per-transaction cumulative limit (minus minimal overhead); this limit follows from the limit on maximum standard transaction length (A.K.A. `MAX_STANDARD_TX_SIZE`).

This alternative would significantly improve the efficiency of many zero-knowledge and post-quantum cryptographic systems (particularly if coupled with [read-only inputs](https://github.com/bitjson/bch-txv5#read-only-inputs)), where signatures alone can easily extend into the kilobytes, and deduplication of contract bytecode could significantly reduce overall network storage and bandwidth usage. However, as [described in `Alternative: 100,000-Byte Locking Bytecode Limit`](#alternative-100000-byte-locking-bytecode-limit), this proposal minimizes scope by avoiding increases to existing practical limits where possible.
