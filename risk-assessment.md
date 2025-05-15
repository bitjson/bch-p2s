# Risk Assessment

The following security considerations, potential risks, and costs have been reviewed to verify the safety and advisability of this proposal.

<details>

<summary><strong>Table of Contents</strong></summary>

- [Risk Assessment](#risk-assessment)
  - [Risks \& Security Considerations](#risks--security-considerations)
    - [User Impact Risks](#user-impact-risks)
      - [Reduced or Equivalent Node Validation Costs](#reduced-or-equivalent-node-validation-costs)
      - [Increased or Equivalent Contract Capabilities](#increased-or-equivalent-contract-capabilities)
    - [Consensus Risks](#consensus-risks)
      - [Full-Transaction Test Vectors](#full-transaction-test-vectors)
      - [Additional Performance Benchmarks](#additional-performance-benchmarks)
      - [`Chipnet` Preview Activation](#chipnet-preview-activation)
    - [Denial-of-Service (DoS) Risks](#denial-of-service-dos-risks)
      - [Node Performance Safety Margin](#node-performance-safety-margin)
    - [Protocol Complexity Risks](#protocol-complexity-risks)
      - [Support for Post-Activation Simplification](#support-for-post-activation-simplification)
      - [Evaluation of Alternatives](#evaluation-of-alternatives)
  - [Upgrade Costs](#upgrade-costs)
    - [Node Upgrade Costs](#node-upgrade-costs)
    - [Ecosystem Upgrade Costs](#ecosystem-upgrade-costs)
  - [Maintenance Costs](#maintenance-costs)
    - [Node Maintenance Costs](#node-maintenance-costs)
    - [Ecosystem Maintenance Costs](#ecosystem-maintenance-costs)

</details>

## Risks & Security Considerations

This section reviews the foreseeable security implications of the proposed changes to the Bitcoin Cash network. Key technical considerations include user impact risks, consensus risks, denial-of-service (DoS) risks, and risks to protocol complexity or maintenance burden of newly introduced behavior.

### User Impact Risks

All upgrade proposals must carefully analyze proposed changes for potential impacts to existing Bitcoin Cash users and use cases. Virtual Machine (VM) upgrades can impact node operators and blockchain indexers (and therefore payment processors, exchanges, and other businesses), software development libraries, wallets, decentralized applications, and a wide range of pre-signed transactions, contract systems, and transaction-settled protocols.

This proposal is designed to preserve backwards-compatibility along a variety of dimensions, minimizing user impact risks:

#### Reduced or Equivalent Node Validation Costs

By avoiding modifications to performance-relevant limits, this proposal minimizes the risk of increase to worst-case validation performance, preventing any increase in node operation costs. Specifically:

- **Locking Bytecode Length** remains limited at the maximum currently-standard length (of bare multi-signature outputs), avoiding any new risk to memory or bandwidth usage. At the same time, locking bytecode standardness validation is simplified, reducing implementation complexity and marginally reducing validation cost in the common case.
- **Token Commitment Length** remains limited below the [existing data-carrier limit](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#op_return-data-carrier-outputs), avoiding any new risks to bandwidth usage arising from the availability of a longer contiguous, commonly-indexed field for publication of arbitrary data. Additionally, the 88-byte cumulative increase in the maximum length of a single standard transaction output (i.e. maximum locking bytecode length plus maximum encoded token prefix length) has no impact on the efficiency of [Data-Carrying Transactions](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#op_return-data-carrier-outputs), no impact on the per-byte cost of increasing UTXO storage requirements ("dust" validation rules are not modified), and no impact on overall ["data storage" costs and incentives](./rationale.md#non-impact-on-data-carrier-outputs-aka-op_return-outputs).
- **Unlocking Bytecode Length** in standard validation is no longer limited differently than during consensus (block) validation, simplifying implementation and reducing potential for unexpected behavior (both in node implementations and on-chain applications). As before, the `100,000` byte limit on maximum standard transaction length (A.K.A. `MAX_STANDARD_TX_SIZE`) remains the primary limit on per-transaction inclusion of unlocking bytecode, avoiding any new risk to memory or bandwidth usage.

Critically, the relaxation of output standardness – i.e. Pay-to-Script (P2S) – was accounted for in the development of [CHIP-2021-05 VM Limits](https://github.com/bitjson/bch-vm-limits). From [VM Limits Risk Assessment: Consideration of Possible Future Changes](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#consideration-of-possible-future-changes):

> - **Relaxation of output standardness** – The re-targeted limits eliminate differences in worst-case performance between Pay-to-Script-Hash (P2SH) and non-P2SH contracts, simplifying potential future upgrades in which [output standardness is relaxed](https://bitcoincashresearch.org/t/relaxing-output-standardness/1391) to allow some custom, non-P2SH contracts within standard validation (e.g. such that output standardness could be determined only by UTXO length rather than by parsed UTXO contents). See [Rationale: Use of Explicitly-Defined Density Limits](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#use-of-explicitly-defined-density-limits) and [Rationale: Use of Input Length-Based Densities](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#use-of-input-length-based-densities),

Further, this risk mitigation strategy has been empirically verified across multiple implementations using a wide range of [cross-implementation functional tests and performance benchmarks](./vmb_tests/).

#### Increased or Equivalent Contract Capabilities

Because this proposal increases or removes limits without modifying other behaviors, existing contract systems, decentralized applications, pre-signed transactions, and transaction-settled protocols are not impacted. Upgrades are only required for applications wishing to take advantage of new privacy, auditability, or fee efficiency capabilities.

### Consensus Risks

All network consensus upgrade proposals must account for consensus risks arising from incorrect or inconsistent implementation of consensus-critical changes. For Virtual Machine (VM) upgrades, consensus risks primarily apply to node implementations and other software which performs VM evaluation as part of transaction validation.

This proposal mitigates consensus risks via 1) an extensive set of full-transaction test vectors, 2) a new cross-implementation performance testing methodology, and 3) a 6-month early activation on `chipnet`.

#### Full-Transaction Test Vectors

To minimize the risk of inconsistencies between implementations, this proposal includes [over 30,000 cross-implementation functional tests and performance benchmarks](./vmb_tests/) covering both the contents of the upgrade and of significant portions of pre-existing VM behavior.

In developing these test vectors, a workflow for test vector development has been established between [Libauth](https://github.com/bitauth/libauth) (a debugging-focused JavaScript implementation) and [Bitcoin Cash Node](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/) (C++) – the implementation most commonly relied on by Bitcoin Cash miners. In effect, this development brings the added reliability of [N-version programming](https://en.wikipedia.org/wiki/N-version_programming) to Bitcoin Cash's VM test vector development.

#### Additional Performance Benchmarks

This proposal adds [additional performance benchmarks](./vmb_tests/) thoroughly covering P2S and the modified limits to help detect performance irregularities and regressions in specific node implementations and VM-evaluating software.

#### `Chipnet` Preview Activation

Finally, like all Bitcoin Cash Improvement Proposals (CHIPs), this proposal schedules activation on `chipnet` 6-months before activation on `mainnet`.

While many development teams and ecosystem stakeholders have reviewed this proposal prior to the November lock-in, only a smaller subset of highly-interested projects and parties (e.g. contract developers, decentralized application development teams, and node implementations) can speculatively allocate development time for complete integration testing with existing software systems.

By scheduling a 6-month early activation on `chipnet`, this proposal minimizes the cost of widest-possible integration testing – in a partially adversarial environment – in which software defects can be discovered and corrected prior to `mainnet` activation.

### Denial-of-Service (DoS) Risks

All network consensus upgrade proposals which alter VM behavior carry risks related to Denial-of-Service (DoS) attacks. In particular, modifications to VM behavior could 1) exacerbate the worst-case performance of transaction or block validation for both expensive-but-valid cases and excessively-invalid cases, and/or 2) decrease the cost or increase the practicality of attempting a particular VM-related DOS attack.

#### Node Performance Safety Margin

This proposal maintains Bitcoin Cash's existing, `10x` to `100x` margin of safety for transaction and block validation performance. See [Limits CHIP Risk Assessment: Expanded Node Performance Safety Margin](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#expanded-node-performance-safety-margin).

### Protocol Complexity Risks

All upgrade proposals must carefully analyze proposed changes for both immediate and potential future impacts on overall protocol complexity. This proposal has been reviewed to ensure that all changes are 1) minimal, 2) necessary, and 3) avoid creating technical debt, even if future upgrades were to further expand the VM's capabilities.

#### Support for Post-Activation Simplification

This proposal allows for backwards-compatible implementation. As a result, implementations may choose to remove activation code following activation.

#### Evaluation of Alternatives

This proposal includes a thorough [Evaluation of Alternatives](./alternatives.md) assessing each option's impact on protocol complexity.

## Upgrade Costs

This section reviews the costs of implementing the proposed changes.

### Node Upgrade Costs

This proposal affects consensus – all fully-validating node software must implement these Virtual Machine (VM) changes to remain in consensus. To minimize the cost of validating node implementation upgrades, this proposal includes a wide range of [cross-implementation functional tests and performance benchmarks](./vmb_tests/).

### Ecosystem Upgrade Costs

Upgrade costs for the wider ecosystem – wallets, blockchain indexers, mining, mining-monitoring, and mining-administrative software, etc. are limited:

- This proposal **does not create new user-facing primitives** (e.g. [CashTokens, 2023](https://cashtokens.org/)) which might demand significant changes or additional features in wallets, block explorers, or other ecosystem software.

  - In particular, note that this proposal excludes a generalized address format for Pay-To-Script (P2S) addresses. See: [Exclusion of P2S CashAddress Format](./rationale.md).

- This proposal **does not notably impact user or miner-visible metrics** (e.g. [Adaptive BlockSize Limit Algorithm, 2024](https://gitlab.com/0353F40E/ebaa#chip-2023-04-adaptive-blocksize-limit-algorithm-for-bitcoin-cash)) which might demand significant changes in software like block explorers, mining calculators, and proprietary mining software and tooling.

  - In particular, note that many P2S outputs have been valid since the beginning of the chain (2009) – any software which attempts to somehow display all outputs must already include support for displaying P2S outputs. Likewise, any software which could be negatively impacted by this proposal (e.g. a crash or display bug) is necessarily vulnerable to the same issues today in valid but non-standard transactions.

While all Bitcoin Cash software systems must inevitably be upgraded to follow all consensus upgrades (e.g. a company must annually upgrade to the latest version of their preferred full node implementation), ecosystem upgrade costs for this proposal are essentially confined to specialized software, tooling, and documentation for contract developers. Further, as the fundamental aim of this proposal is to expand the power and flexibility of Bitcoin Cash's VM for these stakeholders, significant portions of these costs were already speculatively paid, in hopes of later activation, during the creation and review of this proposal.

## Maintenance Costs

All network upgrade proposals must evaluate their foreseeable impact on maintenance costs. Virtual Machine (VM) upgrades can increase the risks of [consensus divergence](#consensus-risks), [performance issues](#denial-of-service-dos-risks), and increased [implementation complexity](#protocol-complexity-risks). This section reviews foreseeable ongoing costs following activation of the proposed changes.

### Node Maintenance Costs

This proposal minimizes any negative impact on node maintenance costs:

- **Equivalent worst-case validation cost** – By design, this proposal avoids any increase in worst-case validation cost. This reduces ongoing node maintenance costs by reducing the potential impact of implementation-specific performance issues.

- **New functional tests and performance benchmarks** – This proposal improves the long-term maintainability of implementations by contributing thorough [cross-implementation functional tests and performance benchmarks](./vmb_tests/) to help root out existing or future performance issues in specific implementations.

- **Potential increase in project maintenance resources** – By increasing the power and flexibility of Bitcoin Cash's VM for contract development, this proposal has the potential to attract further resources to node and VM implementations, both to 1) enhance performance of implementations primarily aimed at mining and/or business infrastructure, and 2) enhance the feature sets of alternative implementations aimed at contract development, decentralized application design, and/or specialized wallet management.

### Ecosystem Maintenance Costs

As with [ecosystem upgrade costs](#ecosystem-upgrade-costs), maintenance costs for the wider ecosystem – wallets, blockchain indexers, mining, mining-monitoring, and mining-administrative software, etc. are limited: this proposal **does not create new user-facing primitives**, and it **does not notably impact user or miner-visible metrics**. See [Ecosystem Upgrade Costs](#ecosystem-upgrade-costs).

Features of this proposal which are relevant to [node maintenance costs](#node-maintenance-costs) are also relevant to the rest of the ecosystem: 1) decreased or equivalent worst-case validation cost, 2) new functional tests and performance benchmarks, and a 3) potential increase in project maintenance resources.
