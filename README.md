# Renegade Bug Bounty
- **Until TVL exceeds $25M**
    - Max Critical Payout: $100,000 USDC
    - High Severity Payout: $20,000 USDC
- **After TVL exceeds $25M**
    - Max Critical Payout: $250,000 USDC
    - High Severity Payout: $20,000 USDC
- [Read our Code4rena bug bounty guidelines for more details](https://docs.code4rena.com/awarding/judging-criteria/bounty-criteria)


## Background on Renegade
### What Is Renegade?
Renegade is a new type of decentralized exchange, an **on-chain dark pool**. 

Current DEXes are completely transparent: Anyone can see your balances and trade history. In contrast, Renegade gives users **universal trade privacy**. That means that trading activity is completely obscured from all third-parties, both before and after a trade is filled. No one except the trader can learn the details of her balances or trades.

Renegade is also **oracle-free**: distributing the matching engine means that prices may be agreed upon and authorized rather than imposed upon traders from an oracle. This allows Renegade's matching engine to build around a "bring your own price" model, which obviates the need for an oracle.

In all, Renegade provides users guaranteed best execution at the real-time Binance midpoint with full pre-trade and post-trade privacy. 

### How Does It Work?
Renegade makes two fundamental steps to achieve pre- and post- trade privacy: <br />
1. **Renegade decentralizes the order book** -- each user alone knows their balances and orders. The matching engine is then run on distributed state and its execution made input-private by encoding the matching engine logic in a secure two party computation, based on [SPDZ](https://eprint.iacr.org/2012/642.pdf). This gives the system pre-trade privacy; no party learns another's state as guaranteed by the 2PC. <br />
2. **Renegade Merklizes the exchange state** and stores the tree in a [Stylus](https://docs.arbitrum.io/stylus/stylus-gentle-introduction) smart contract. The leaves in this tree are opaque, blinded commitments to user "wallets" that contain balances, orders, etc. The smart contract is equipped with a [PlonK](https://eprint.iacr.org/2019/953.pdf) verifier, which allows the contract to enforce predicates representing match settlement, user state updates, etc on any commitments inserted into the Merkle tree. The commitments & zero-knowledge proofs attesting to their validity provide the exchange with a secure, post-trade private settlement layer.

Lastly, to take part in MPCs the user must be "online" at all times. This is an unrealistic assumption, so we introduce **relayers**; actors in the system that *manage* the state of user orders and shoulder the computational burden of the PlonK prover. Relayers can be grouped into clusters to enable horizontal scaling, with each relayer in a cluster having identical permissions to manage its share of the overall state.

**Note**: When a user delegates its wallet to a relayer, the relayer may only *match orders*, it may not update state: placing/cancelling orders, depositing/withdrawing balances, etc. This protection is encoded in a wallet's keychain, which is only partially shared with the relayer, and which the smart contract uses to authenticate state updates.

The following figure shows three relayer clusters interacting with the smart contract; including a "Public Gateway" managed by Renegade.
![](images/clusters.png)


## Further Technical Resources & Links
- **zkSecurity Audit** \([@zksecurityXYZ](https://x.com/zksecurityXYZ)\): An audit of our circuits and smart contracts with a great system overview enriched with helpful diagrams. [[Link](https://github.com/renegade-fi/renegade/blob/main/audits/zksecurity.pdf)]
- **Renegade Circuit Spec**: A specification of our system and the types, invariants, and constraints used in our circuits. [[Link](https://renegade-fi.notion.site/Updated-Circuit-Spec-24cf2047fdee4fac9212a9b5f1a8f3de)]
- **Renegade Docs**: Our system documentation, subject to change. [[Link](https://docs.renegade.fi/)]
- **Renegade Whitepaper**: The whitepaper our current instantiation is based off of. Note that the implemented system diverges from the whitepaper's description in significant ways. [[Link](https://whitepaper.renegade.fi/)]
- **Renegade Website**: [https://renegade.fi](https://renegade.fi)
- **Twitter**: [@renegade_fi](https://x.com/renegade_fi)
- **Discord** [https://discord.gg/renegade-fi](https://discord.gg/renegade-fi)

# Scope & Severity Criteria
### Severity: Safety versus Privacy
We delineate the critical versus high severity of a finding based on **safety** versus **privacy** respectively. That is, a critical finding must demonstrate high likelihood of being exploited and with impact that results in the theft or freezing of user funds.

A finding that does not place user funds at risk but demonstrates a high likelihood of being exploited with impact that results in the leaking of user balances or open orders will not be classified as critical, but may be classified as high.

## Scope
### Relayer Codebase

Note that at launch, MPC matches are disabled -- all state is managed by the Renegade cluster, which will rely solely on its internal matching engine. So findings in the context of the MPC matching engine are out of scope until MPC is enabled.

As such, the relayer codebase is scoped down to those sources relevant to the safety of user funds and the privacy of user state in a single-cluster setup. These are, in particular: the circuit types, circuits, gadgets, and the relayer's API server -- through which state is accessible.

As the system decentralizes, this scope will expand to include sources relevant in the multiparty context. 

Sources found here: [Github](https://github.com/renegade-fi/renegade)

| File | SLOC | Purpose |
|------|------|---------|
| circuit-types/src/merkle.rs | 38 | Merkle tree circuit types |
| circuit-types/src/order.rs | 193 | User order circuit types |
| circuit-types/src/srs.rs | 267 | The structured reference string used in our PlonK setup |
| circuit-types/src/transfers.rs | 81 | The transfer circuit type for deposits/withdrawals |
| circuit-types/src/lib.rs | 463 | Circuit type definitions |
| circuit-types/src/note.rs | 72 | The note type, used to pay fees out of a balance |
| circuit-types/src/fixed_point.rs | 841 | A circuit type for fixed point representation in finite fields |
| circuit-types/src/balance.rs | 87 | User balance circuit types |
| circuit-types/src/match.rs | 106 | The match tuple circuit types -- the result of a match |
| circuit-types/src/wallet.rs | 152 | The user's wallet circuit type |
| circuit-types/src/errors.rs | 77 | Errors for circuit types |
| circuit-types/src/elgamal.rs | 253 | ElGamal encryption types |
| circuit-types/src/keychain.rs | 396 | User keychain circuit types |
| circuit-types/src/traits.rs | 1132 | Traits for interacting with circuit types |
| circuits/src/zk_circuits/valid_fee_redemption.rs | 826 | The circuit for valid fee redemption |
| circuits/src/zk_circuits/valid_commitments.rs | 1007 | The circuit for valid commitments |
| circuits/src/zk_circuits/valid_wallet_update.rs | 1445 | The circuit for valid wallet update |
| circuits/src/zk_circuits/valid_reblind.rs | 628 | The circuit for valid reblind |
| circuits/src/zk_circuits/valid_wallet_create.rs | 299 | The circuit for valid wallet create |
| circuits/src/zk_circuits/proof_linking.rs | 753 | Proof linking implementation |
| circuits/src/zk_circuits/valid_offline_fee_settlement.rs | 828 | The circuit for valid offline fee settlement |
| circuits/src/zk_circuits/mod.rs | 391 | Module definitions for circuits |
| circuits/src/zk_circuits/valid_match_settle/multi_prover.rs | 479 | The multi-prover circuit for valid match settlement |
| circuits/src/zk_circuits/valid_match_settle/single_prover.rs | 456 | The single-prover circuit for valid match settlement |
| circuits/src/zk_circuits/valid_match_settle/mod.rs | 912 | Module definitions for match settlement circuits |
| circuits/src/zk_circuits/valid_relayer_fee_settlement.rs | 1085 | The circuit for validating relayer fee settlement |
| circuits/src/zk_gadgets/merkle.rs | 338 | Gadgets for Merkle tree operations |
| circuits/src/zk_gadgets/arithmetic.rs | 167 | Gadgets for arithmetic operations |
| circuits/src/zk_gadgets/wallet_operations.rs | 530 | Gadgets for wallet operations |
| circuits/src/zk_gadgets/bits.rs | 347 | Gadgets for bit operations |
| circuits/src/zk_gadgets/note.rs | 79 | Gadgets for note operations |
| circuits/src/zk_gadgets/fixed_point.rs | 298 | Gadgets for fixed-point arithmetic |
| circuits/src/zk_gadgets/mod.rs | 18 | Module definitions for zk gadgets |
| circuits/src/zk_gadgets/comparators.rs | 650 | Gadgets for boolean comparison operations |
| circuits/src/zk_gadgets/select.rs | 132 | Gadgets for selection operations |
| circuits/src/zk_gadgets/elgamal.rs | 197 | Gadgets for ElGamal encryption operations |
| circuits/src/zk_gadgets/poseidon.rs | 384 | Gadgets for Poseidon hash function operations |
| workers/api-server/src/websocket/price_report.rs | 114 | Handles websocket communication for price reporting |
| workers/api-server/src/websocket/handler.rs | 89 | Main websocket message handler |
| workers/api-server/src/websocket/task.rs | 123 | Manages websocket messages for relayer tasks |
| workers/api-server/src/websocket/wallet.rs | 74 | Handles wallet-related websocket operations |
| workers/api-server/src/websocket/order_status.rs | 64 | Manages order status updates via websocket |
| workers/api-server/src/auth/admin_auth.rs | 126 | Implements authentication for admin users |
| workers/api-server/src/auth/helpers.rs | 49 | Helper functions for authentication |
| workers/api-server/src/auth/mod.rs | 84 | Module definitions for authentication |
| workers/api-server/src/auth/wallet_auth.rs | 134 | Implements authentication for wallet users |
| workers/api-server/src/error.rs | 72 | Defines error types and handling for the API server |
| workers/api-server/src/lib.rs | 20 | Main library file for the API server |
| workers/api-server/src/worker.rs | 144 | The main event loop for the API server |
| workers/api-server/src/compliance/client.rs | 46 | Compliance client for the API server |
| workers/api-server/src/compliance/mod.rs | 5 | Module definitions for compliance-related functionality |
| workers/api-server/src/websocket.rs | 404 | Main websocket implementation and routing |
| workers/api-server/src/http/price_report.rs | 63 | Handles HTTP routes for price reporting |
| workers/api-server/src/http/task.rs | 156 | Manages HTTP routes for relayer tasks |
| workers/api-server/src/http/wallet.rs | 957 | Implements HTTP routes for wallet state |
| workers/api-server/src/http/network.rs | 148 | Handles HTTP routes for p2p network operations |
| workers/api-server/src/http/order_book.rs | 97 | Manages HTTP routes for network order book |
| workers/api-server/src/http/admin.rs | 475 | Implements HTTP routes for admin operations |
| workers/api-server/src/router.rs | 381 | Main router for HTTP and websocket routes |
| workers/api-server/src/http.rs | 530 | Main HTTP implementation and routing |
| **Total** | 20762 |  |

### Smart Contracts

Sources found here: [Github](https://github.com/renegade-fi/renegade-contracts)

| File | SLOC | Purpose |
|------|-------|-------------|
| contracts-common/src/types.rs | 522 | Types for smart contracts |
| contracts-common/src/constants.rs | 119 | Constants used in smart contracts |
| contracts-common/src/solidity.rs | 89 | Solidity type definitions |
| contracts-common/src/lib.rs | 14 | Library file |
| contracts-common/src/serde_def_types.rs | 166 | Serde foreign trait implementations |
| contracts-common/src/backends.rs | 71 | Arithmetic and hashing backends |
| contracts-common/src/custom_serde.rs | 511 | Custom serialization and deserialization |
| contracts-core/src/crypto/mod.rs | 4 | Module file |
| contracts-core/src/crypto/poseidon.rs | 10 | Shim code for Poseidon hash function|
| contracts-core/src/crypto/ecdsa.rs | 94 | ECDSA signature verification and related functions |
| contracts-core/src/lib.rs | 13 | Main library file for contracts-core |
| contracts-core/src/transcript/mod.rs | 290 | Implementation of Fiat-Shamir transformation and transcript |
| contracts-core/src/verifier/mod.rs | 1197 | Main implementation of PlonK verification|
| contracts-core/src/verifier/errors.rs | 37 | Error types and handling for the verifier module |
| contracts-stylus/src/contracts/merkle.rs | 353 | Implements Merkle tree operations for the darkpool |
| contracts-stylus/src/contracts/transfer_executor.rs | 152 | Handles token transfer execution in the darkpool |
| contracts-stylus/src/contracts/vkeys.rs | 55 | Manages verification keys for PlonK proofs |
| contracts-stylus/src/contracts/darkpool_core.rs | 790 | Core functionality of the darkpool contract |
| contracts-stylus/src/contracts/mod.rs | 28 | Module definitions for darkpool contracts |
| contracts-stylus/src/contracts/verifier.rs | 46 | Implements PlonK proof verification |
| contracts-stylus/src/contracts/darkpool.rs | 599 | Main darkpool contract implementation |
| contracts-stylus/src/lib.rs | 17 | Library file for the darkpool project |
| contracts-stylus/src/utils/constants.rs | 539 | Constants used across the darkpool contracts |
| contracts-stylus/src/utils/solidity.rs | 76 | Solidity type definitions |
| contracts-stylus/src/utils/helpers.rs | 254 | Helper functions for the darkpool contracts |
| contracts-stylus/src/utils/mod.rs | 7 | Module definitions for utility functions |
| contracts-stylus/src/utils/backends.rs | 156 | Arithmetic and hashing backends |
| **Total** | 6209 |  |

# Additional Context
### Publicly Known Issues
The following are publicly known issues that will not be considered valid findings, but may be helpful pointers as to where to look for vulnerabilities:

- Users may avoid paying the relayer a fee if they are able to "self match" at a zero (or low) price — effectively a transfer to another wallet. This drains a balance in one wallet, and allows the user to abandon the wallet with its non-zero fees.

    The user will still owe a fee on the transferred balance in the new wallet, but this may be less than what they owe on the abandoned wallet.

    Relayers choose prices themselves in the price agreement phase of the match, so naturally they will choose fair market prices to avoid this. However, if a user queues up a "self match" manually (without going through the relayer) they may still exploit this. 

### Trusted Roles
- **Owner**: The owner of the darkpool may pause and unpause the system, set the protocol fee and fee encryption key, and upgrade any contract in the system. [[Source Code](https://github.com/renegade-fi/renegade-contracts/blob/main/contracts-stylus/src/contracts/darkpool.rs#L45)]
- **Relayer**: A relayer manages user state -- shopping around orders and proving state transition predicates in zero-knowledge. The relayer should *never* be able to place orders, deposit/withdraw balances, etc without the user's authorization via their privately held keychain.

    The relayer is trusted to faithfully execute orders at the real time Binance midpoint. As such, the relayer selecting a malicious price is _not_ classified as theft of funds. 
    
    **Note** that while the relayer is trusted with price agreement, user orders have a `worst_cast_price` field that backstops the price that the relayer chooses.

### Protocol Invariants
See the [Smart Contract Invariants](https://github.com/renegade-fi/renegade-contracts/blob/main/docs/specification.md#protocol-invariants) for a detailed discussion of the invariants in the system.

### Miscellaneous
Employees of Renegade, and employees' family members are ineligible for bounties.


