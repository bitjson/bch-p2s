# CHIP-2024-12 P2S: Pay to Script

        Title: Pay to Script
        Type: Standards
        Layer: Consensus
        Maintainer: Jason Dreyzehner
        Status: Draft
        Initial Publication Date: 2024-12-12
        Latest Revision Date: 2024-09-05
        Version: 1.0.3

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

1. **Length check**: the locking bytecode of standard outputs must have a length less than or equal to `201`, the maximum length of currently standard bare multi-signature (BMS) outputs. See [Rationale: Selection of Maximum Locking Bytecode Length](rationale.md#selection-of-maximum-locking-bytecode-length).
2. **Data-carrier validation**: locking bytecode which exceeds the length check but matches the existing data-carrier pattern (`OP_RETURN`) is standard if accepted by the existing data-carrier validation (the cumulative 223-byte limit across all transaction outputs).

#### Retained Allowance for Multi-Signature Spends

Note that UTXO standardness checks must also retain the additional allowance for bare multi-signature patterns; these remain nonstandard to create beyond the existing 3-key limit, but UTXOs exceeding the limit remain spendable in standard transactions (matching the existing behavior for these cases).

### Token Commitment Length

The limit on maximum length of token commitments is raised from `40` bytes to `128` bytes. See [Rationale: Selection of Maximum Token Commitment Length](rationale.md#selection-of-maximum-token-commitment-length).

### Unlocking Bytecode Length

The limit on maximum standard input bytecode length (A.K.A. `MAX_TX_IN_SCRIPT_SIG_SIZE` – 1,650 bytes) is removed such that the maximum unlocking bytecode length is equal for both standard and consensus validation: 10,000 bytes (A.K.A. `MAX_SCRIPT_SIZE`). See [Rationale: Unification of Standard and Consensus Unlocking Bytecode Length](rationale.md#unification-of-standard-and-consensus-unlocking-bytecode-length).

## Rationale

- [Appendix: Rationale &rarr;](rationale.md#rationale)
  - [Relationship Between the Modified Limits](rationale.md#relationship-between-the-modified-limits)
  - [Selection of Maximum Locking Bytecode Length](rationale.md#selection-of-maximum-locking-bytecode-length)
  - [Selection of Maximum Token Commitment Length](rationale.md#selection-of-maximum-token-commitment-length)
  - [Unification of Standard and Consensus Unlocking Bytecode Length](rationale.md#unification-of-standard-and-consensus-unlocking-bytecode-length)
  - [Exclusion of P2S CashAddress Format](rationale.md#exclusion-of-p2s-cashaddress-format)
  - [Non-Impact on Data-Carrier Outputs (A.K.A. OP_RETURN Outputs)](rationale.md#non-impact-on-data-carrier-outputs-aka-op_return-outputs)
  - [Non-Reliance on Miscalculated 220-Byte Limit](#non-reliance-on-miscalculated-220-byte-limit)

## Evaluations of Alternatives

- [Appendix: Evaluations of Alternatives &rarr;](alternatives.md#evaluation-of-alternatives)
  - [Locking Bytecode Standardness Alternatives](alternatives.md#locking-bytecode-standardness-alternatives)
    - [Status Quo: Pattern Matching](alternatives.md#status-quo-pattern-matching)
    - [Alternative: Reduced Locking Bytecode Standardness Limit](alternatives.md#alternative-reduced-locking-bytecode-standardness-limit)
    - [Alternative: Increased Locking Bytecode Standardness Limit](alternatives.md#alternative-increased-locking-bytecode-standardness-limit)
      - [Alternative: 220-Byte Locking Bytecode Standardness Limit](alternatives.md#alternative-220-byte-locking-bytecode-standardness-limit)
      - [Alternative: 10,000-Byte Locking Bytecode Limit](alternatives.md#alternative-10000-byte-locking-bytecode-limit)
      - [Alternative: 100,000-Byte Locking Bytecode Limit](alternatives.md#alternative-100000-byte-locking-bytecode-limit)
  - [Token Commitment Length Alternatives](alternatives.md#token-commitment-length-alternatives)
    - [Status Quo: 40-Byte Token Commitments](alternatives.md#status-quo-40-byte-token-commitments)
    - [Alternative: 201-Byte Token Commitments](alternatives.md#alternative-201-byte-token-commitments)
    - [Alternative: 220-Byte Token Commitments](alternatives.md#alternative-220-byte-token-commitments)
    - [Alternative 10,000-Byte Token Commitments](alternatives.md#alternative-10000-byte-token-commitments)
    - [Alternative 100,000-Byte Token Commitments](alternatives.md#alternative-100000-byte-token-commitments)
  - [Unlocking Bytecode Length Alternatives](alternatives.md#unlocking-bytecode-length-alternatives)
    - [Status Quo: 1,650-Byte Unlocking Bytecode Standardness Limit](alternatives.md#status-quo-1650-byte-unlocking-bytecode-standardness-limit)
    - [Alternative: 100,000-Byte Unlocking Bytecode Limit](alternatives.md#alternative-100000-byte-unlocking-bytecode-limit)

## Risk Assessment

- [Appendix: Risk Assessment &rarr;](risk-assessment.md#risk-assessment)
  - [Risks \& Security Considerations](risk-assessment.md#risks--security-considerations)
    - [User Impact Risks](risk-assessment.md#user-impact-risks)
      - [Reduced or Equivalent Node Validation Costs](risk-assessment.md#reduced-or-equivalent-node-validation-costs)
      - [Increased or Equivalent Contract Capabilities](risk-assessment.md#increased-or-equivalent-contract-capabilities)
    - [Consensus Risks](risk-assessment.md#consensus-risks)
      - [Full-Transaction Test Vectors](risk-assessment.md#full-transaction-test-vectors)
      - [Additional Performance Benchmarks](risk-assessment.md#additional-performance-benchmarks)
      - [`Chipnet` Preview Activation](risk-assessment.md#chipnet-preview-activation)
    - [Denial-of-Service (DoS) Risks](risk-assessment.md#denial-of-service-dos-risks)
      - [Node Performance Safety Margin](risk-assessment.md#node-performance-safety-margin)
    - [Protocol Complexity Risks](risk-assessment.md#protocol-complexity-risks)
      - [Support for Post-Activation Simplification](risk-assessment.md#support-for-post-activation-simplification)
      - [Evaluation of Alternatives](risk-assessment.md#evaluation-of-alternatives)
  - [Upgrade Costs](risk-assessment.md#upgrade-costs)
    - [Node Upgrade Costs](risk-assessment.md#node-upgrade-costs)
    - [Ecosystem Upgrade Costs](risk-assessment.md#ecosystem-upgrade-costs)
  - [Maintenance Costs](risk-assessment.md#maintenance-costs)
    - [Node Maintenance Costs](risk-assessment.md#node-maintenance-costs)
    - [Ecosystem Maintenance Costs](risk-assessment.md#ecosystem-maintenance-costs)

## Test Vectors

This proposal includes [a suite of functional tests and benchmarks](./vmb_tests/) to verify the performance of all operations within virtual machine implementations.

## Implementations

Please see the following implementations for additional examples and test vectors:

- C++:
  - [Bitcoin Cash Node (BCHN)](https://bitcoincashnode.org/) – A professional, miner-friendly node that solves practical problems for Bitcoin Cash. [Merge Request !1937](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/merge_requests/1937).
- **JavaScript/TypeScript**:
  - [Libauth](https://github.com/bitauth/libauth) – An ultra-lightweight, zero-dependency JavaScript library for Bitcoin Cash. [Branch `next`](https://github.com/bitauth/libauth/tree/next).
  - [Bitauth IDE](https://github.com/bitauth/bitauth-ide) – An online IDE for bitcoin (cash) contracts. [Branch `next`](https://github.com/bitauth/bitauth-ide/tree/next).

## Stakeholder Responses & Statements

[Stakeholder Responses & Statements &rarr;](./stakeholders.md)

## Feedback & Reviews

- [Pay to Script CHIP Issues](https://github.com/bitjson/bch-p2s/issues)
- [`Relaxing output standardness` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/relaxing-output-standardness/1391)
- [`CHIP 2024-12 P2S: Pay to Script` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/chip-2024-12-p2s-pay-to-script/1451)

## Changelog

This section summarizes the evolution of this document.

- **v1.0.3 – 2025-09-05**
  - Update VMB tests and benchmarks
- **v1.0.2 – 2025-05-15**
  - Expand [Rationale](./rationale.md)
  - Add [Evaluation of Alternatives](./alternatives.md)
  - Add [Risk Assessment](./risk-assessment.md)
  - Scaffold [Stakeholder Responses & Statements](./stakeholders.md)
- **v1.0.1 – 2025-05-02**
  - Note that multisig standardness behavior is not modified ([#1](https://github.com/bitjson/bch-p2s/issues/1))
  - Commit latest test vectors
  - Link to BCHN implementation
- **v1.0.0 – 2024-12-12**
  - Initial publication

## Copyright

This document is placed in the public domain.
