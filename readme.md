# CHIP-2021-08-ZCE: Zero-Confirmation Escrows

        Title: Zero-Confirmation Escrows
        Type: Standards
        Layer: Peer Services
        Authors: Jason Dreyzehner, Marty Alcala
        Status: Draft
        Specification Version: 1.0.0
        Initial Publication Date: 2021-08-18
        Latest Revision Date: 2021-08-18

## Summary

This proposal introduces a set of Zero-Confirmation Escrow (ZCE) contracts, protocols, and transaction relay and mining polices which improve the speed and security of unconfirmed transactions on Bitcoin Cash.

## Deployment

Deployment of this specification does not require network coordination, though sufficient deployment is required before ZCE-secured transactions can be safely accepted as final.

ZCE transaction relay & mining policies are recommended for deployment in the November 2021 upgrade.

## Background & Motivation

The Bitcoin Cash ecosystem makes significant use of **zero-confirmation transactions** – transactions which have not yet been included in a block.

As BCH block sizes are limited well-beyond typical network throughput, practically all transactions are included in the next block after their initial broadcast. Additionally, a widely-deployed "first-seen" network relay rule ensures that honest nodes refuse to relay or mine transactions which conflict with any already-seen transactions. These properties make unconfirmed BCH transactions reliable enough for many applications, provided precautions are taken to mitigate the risk of payment fraud.

### Existing Risk-Reduction Strategies

**Payment fraud** on BCH is most likely to occur before a transaction has been included in a block<sup>1</sup>. When the payee receives a zero-confirmation transaction which they expect to be confirmed in the next block, they release the products/services to the payer. However, when the next block is mined, the payee finds that a conflicting transaction has been included rather than the expected transaction; **the payee never receives the expected payment despite having already released the products/services**.

Depending on the specific application, payees can utilize a number of existing strategies to reduce the risk of payment fraud.

<small>

1. While it is also possible for payments to be fraudulently reversed after inclusion in a widely-accepted block, this type of payment fraud is both far more expensive and disincentivized by the protocol – see the [original whitepaper](./bitcoin.pdf) for details.

</small>

#### Transaction Conflict Monitoring

By listening for conflicting transactions on the network for 5 to 10 seconds prior to considering a payment final, payees can significantly reduce the risk of a payment being fraudulently reversed<sup>2</sup>. To monitor the network, payment service providers maintain sets of widely-connected nodes which are more likely to receive a broadcast of a conflicting payment.

A new protocol message type, the [Double Spend Proof (DSP)](https://documentation.cash/protocol/network/messages/dsproof-beta), is also being widely-tested and deployed to help propagate knowledge of conflicting transactions (without assisting in the propagation of the conflicting transactions themselves).

<small>

2. [Testing on BCH in 2018](https://www.youtube.com/watch?v=TIt96gFh4vw) indicated that attack success probability falls below 1% after only 2 seconds, and less than 0.1% at 3 seconds. Even for sophisticated attacks with a high-fee replacement transaction delivered by a network of well-connected nodes, after a 5 second delay, attacks had only a 5% success probability.

</small>

#### Transaction Volume Limits

Payees can also limit fraud risk by placing upper limits on the individual payment value and total outstanding transaction volume contained in zero-confirmation transactions. If a payee's total exposure remains below the expected cost of maintaining attack infrastructure (well-connected nodes, rented mining power, reputational risk to colluding mining entities, etc.), attacks must be coordinated and simultaneously executed across many payees to remain profitable.

#### External Transaction Reversal

In many cases, payees can reduce or eliminate payment fraud risk by maintaining reversibility until unconfirmed transactions are included in a block. For example, some digital subscriptions can be canceled within a few minutes or hours with little cost to the business.

This strategy can also be employed by payees who deliver goods after a delay, such as e-commerce merchants – orders paid by fraudulently-reversed payments can be canceled before fulfillment or delivery.

#### External Dispute Resolution

Finally, external dispute resolution systems can both dissuade attacks and help to recuperate lost funds and stolen merchandise.

By requiring and validating customer identity information, merchants can ensure they have methods of recourse in the case of a successful attack. Businesses with ongoing relationships can suspend accounts and ban customers which attempt to defraud them. If an attacker is successful, payees can pursue restitution via debt collection, credit, arbitration, and/or state-run legal systems.

## Benefits

**Zero-Confirmation Escrows** (ZCEs) enable instant, secure BCH payments, particularly in point-of-sale, ATM, and vending applications where payers have no prior or ongoing relationship with the payee.

### Instant Payments

Because non-ZCE payments can only be considered reasonably secure after a short delay for network monitoring, secure in-person BCH payment systems often seem slow. Even delays of 5-10 seconds make a payment experience non-competitive with centralized digital payment methods (e.g. tap-to-pay devices or credit cards).

Payment acceptance systems which hide this delay by immediately displaying a success state risk bad merchant experiences (higher fraud rates, particularly against new or inadequately-trained employees) and higher employee training costs.

**Zero-Confirmation Escrows eliminate this network monitoring delay**, allowing payees to instantly validate and accept ZCE-secured transactions as final. Because ZCEs can be validated by a local node, **ZCE-secured payments can provide faster user experiences than centralized payment systems** (which require a round-trip communication between the point-of-sale and a bank or centralized payment processor).

### Long-Term Security

The presently-low risk of accepting zero-confirmation transactions is predicated on a common but unenforceable norm: the "first-seen" transaction mining policy. Because miners which do not follow this policy cannot be conclusively identified and punished by other network actors – and BCH is not the only SHA256-based chain on which miners can operate – the current behavior can only be considered stable while the "defraudable" volume of zero-confirmation commerce is less than the expected value of each block coinbase<sup>3</sup>.

As block subsidies continue to be replaced by transaction fees and zero-confirmation commerce volume increases, the expected profitability of attack infrastructure may rise significantly. Unscrupulous mining pools could augment their income by offering zero-confirmation transaction reversal services which circumvent network monitoring strategies and occasionally succeed at reversing transactions approximately 10 minutes later.

ZCEs remain incentive-secure against this category of attacks, even as zero-confirmation commerce volume increases. With adequate escrow values, **ZCE-secured transactions can be accepted with the same finality as 1-confirmation transactions**.

<small>

1. Even "hostile" miners have a temporary incentive to maintain the value of BCH (including as a fast, secure payment system): each block coinbase can only be moved after 100 additional blocks have been mined beyond it. As such, a rational miner with no other attachment to BCH would only find it desirable to offer "fraud-as-a-service" if the expected returns (possibly paid in another currency) were greater than the expected depreciation of any BCH mined (plus the implementation and maintenance cost of attack infrastructure).

</small>

### Improved Privacy

By relying heavily on [external dispute resolution](#external-dispute-resolution) for zero-confirmation payment fraud, BCH currently incentivizes many businesses to implement identity verification in order to accept zero-confirmation transactions.

Many exchanges, payment processors, ATM operators, and other services have begun collecting customer identity information, even in markets where legal and regulatory regimes do not require such behavior. This seems to indicate that these businesses find the risk-mitigation (and other added value) of collecting identity information outweighs financial privacy-related losses.

**ZCEs offer instant finality and an automatic response in the case of fraud**. By avoiding the need to collect identity information, ZCEs can improve customer privacy and reduce business risks due to data breaches.

### Reduced Infrastructure Costs

Proper network monitoring infrastructure is non-trivial to setup and maintain, making safe, self-hosted, zero-confirmation transaction acceptance systems more expensive and less practical for many businesses. Because monitoring services must be widely-connected, they also place a higher bandwidth burden on the network than nodes with fewer connections. While [DSPs](https://documentation.cash/protocol/network/messages/dsproof-beta) reduce the cost of this infrastructure, they do not eliminate the need for active network monitoring after a transaction is received.

ZCEs completely eliminate the need for network monitoring after a transaction is received. **ZCE-secured transactions can be validated by a sporadically-connected local node**, and ZCE security does not rely upon merchants hearing the broadcast of a DSP.

### Business Certainty

For many businesses, assessing the risk of payment fraud against zero-confirmation BCH transactions is non-trivial. Payment fraud remains extremely rare, likely because the expected returns remain small; a large actor rolling out widespread support for zero-confirmation transactions could trigger the development of [attack networks](#full-value-escrow-requirement). This uncertainty is a barrier to development and wider adoption of zero-confirmation payment applications.

The fraud-risk of ZCE-secured transactions can be easily assessed. Even when colluding with a miner, an attacker must risk significant resources with a low probability of success, and at-risk businesses can easily [tune their escrow requirements](#accepting-zce-secured-payments) to match their own risk profile.

## Costs & Risk Mitigation

The following costs and risks have been assessed.

### Increased Transaction Sizes

ZCE-secured transactions include an additional output which must be reclaimed in a later transaction. Together, these add an average overhead of 295 bytes<sup>1</sup>.

While this slightly increases the cost of transactions using ZCEs (by ~$0.001 in 2020 USD), many users are likely willing to pay such a premium to avoid a ~10 minute delay for transaction confirmations. Even for users making hundreds of payments a month, this additional cost should amount to less than $1 per year (2020 USD). Many users are likely willing to pay this price even to avoid 5-10 second network monitoring delays with merchants who currently accept higher-risk, non-escrowed payments.

While ZCEs are likely to increase the total blockchain space consumed by point-of-sale and other zero-confirmation transactions, for comparison, the additional cost of ZCEs is similar to the overhead of multisignature wallet security or proper use of [CashFusion](https://cashfusion.org/).

<details>
<summary>Calculations</summary>

1. ZCE-secured transaction includes a ZCE P2SH output (32 bytes), an additional reclaim transaction requires X bytes: ZCE reclaim input overhead (36-byte outpoint, 4-byte sequence number, 1-byte bytecode length), ZCE reclaim input unlocking bytecode (100 bytes), ZCE reclaim input redeem bytecode (63 bytes for single input, 71 bytes for 1-level merkle tree, 78 bytes for 2-level merkle tree, 85 bytes for 3-level merkle tree, ..., 177 bytes for 16-level merkle tree), other reclaim transaction overhead (4-byte version, ~2-byte input/output counts, 34-byte P2PKH output, 4-byte locktime). Total without redeem bytecode: `32 + 36 + 4 + 1 + 100 + 4 + 2 + 34 + 4 ~= 217`. Total for 1 input: `217 + 63 = 280`, 2 inputs: `217 + 71 = 288`, 4 inputs: `217 + 78 = 295`, 8 inputs: `217 + 85 = 302`, 65,536 inputs: `217 + 177 = 394`. As of May 2021, average input count of transactions is ~3; average expected overhead is 295 bytes. Minimum overhead is 280 bytes; maximum overhead is 394 bytes.

</details>

### Modification to Transaction Acceptance/Relay

Modifications to transaction acceptance and relay policies can create differences in mempools (the store of not-yet-confirmed transactions) between various node versions and implementations. Because existing users of non-escrowed, zero-confirmation transactions rely on mempool consistency and network monitoring for transaction security, these changes can increase the fraud-risk of existing zero-confirmation transaction use cases<sup>1</sup>.

**Mitigation**: this specification has been carefully designed to [avoid modifications which impact existing users of zero-confirmation transactions](#transaction-replacement-period).

<small>

1. A discussion of this fraud vector is documented in [Double Spending in Bitcoin](https://medium.com/@octskyward/double-spending-in-bitcoin-be0f1d1e8008) by Mike Hearn.

</small>

### Node Implementation Complexity

Because ZCEs require [specialized transaction relay and mining policies](#zce-relay--mining-policies), they may increase the cost and complexity of node implementations.

**Mitigations**: notably, this proposal does not introduce new consensus rules, reducing the possibility that ZCEs contribute to technical debt. While ZCEs may be highly-valuable to the current network, future node implementations can choose to deprecate support for ZCEs if a superior solution arises and ZCE use becomes negligible.

## Technical Specification

A standard Zero-Confirmation Escrow (ZCE) contract type and validation strategy is specified, ZCE-specific transaction relay and mining policies are recommended, and a ZCE extension to the JSON Payment Protocol (v2) is defined.

### Technical Summary

Zero-Confirmation Escrows are a standardized contract with which a payer can guarantee that a payment will not be invalidated by a double-spend, allowing the payment to be accepted with confidence similar to a single confirmation (as if the transaction were included in the most recently mined block).

To make a ZCE-secured payment, the payer computes the proper P2SH ZCE output for the inputs used in a transaction, allocating at least a minimum value (as required by the payee) to the output. In a following transaction, the payer immediately reclaims the ZCE output back to their wallet, exposing the P2SH redeem bytecode as a valid ZCE contract. Because ZCE-secured transactions use a modified transaction relay and mining policy, the ZCE-secured transaction can be safely accepted immediately without a delay for network monitoring.

If a payer attempts to invalidate a ZCE-secured transaction by double-spending any of its inputs, a miner can claim the ZCE using information available in either the double-spending transaction itself or a [Double Spend Proof (DSP)](https://documentation.cash/protocol/network/messages/dsproof-beta). To claim the ZCE, the miner must confirm the ZCE-secured transaction, ensuring the payee also receives the expected funds<sup>1</sup>.

<small>

1. In practice, this result only occurs when an attacker is betrayed by a miner claiming to offer "fraud-as-a-service". The most likely attack on ZCE-accepting merchants is a follow-up ZCE transaction which costs the attacker more than the payment (burned to miners) but prevents the merchant from receiving their expected funds. Merchants with such well-funded adversaries can mitigate this attack by [increasing escrow requirements](#accepting-zce-secured-payments) and/or [monitoring for 5 seconds](#monitoring-for-replacement-transactions).

</small>

### ZCE Contract & Validation

A **Zero-Confirmation Escrow Contract** is defined by the following Pay-To-Script-Hash (P2SH) contract template:

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK [...repeat to tree depth]
    OP_[tree depth] OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160 [...repeat to tree depth]

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

The `reclaim` key is controlled by the payer creating the ZCE-secured transaction, and the `root_hash` is the transaction's [ZCE Root Hash](#zce-root-hash).

A [ZCE authentication template](https://ide.bitauth.com/import-gist/104baff7503d6a7ad619ad814153b059) is also available.

<details>

<summary>Exhaustive Specification</summary>

The following 17 contracts are considered valid ZCE contracts. For a ZCE to be valid, it must use the minimum-size contract capable of covering the required number of public keys.

**ZCE Contract (1 Public Key)**

This contract is a special case: if only one public key is covered, no merkle tree is used.

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_DUP
    OP_HASH160
    <$(<input_public_key> OP_HASH160)>
    OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (2 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK
    OP_1 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (3-4 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK
    OP_2 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (5-8 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_3 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (9-16 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_4 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (17-32 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_5 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (33-64 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_6 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (65-128 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_7 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (129-256 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_8 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (257-512 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_9 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (513-1,024 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_10 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (1,025-2,048 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_11 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (2,049-4,096 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_12 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (4,097-8,192 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_13 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (8,193-16,384 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_14 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (16,385-32,768 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_15 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

**ZCE Contract (32,769-65,536 Public Keys)**

```
OP_DUP OP_HASH160 <$(<reclaim.public_key> OP_HASH160)> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK OP_TOALTSTACK
    OP_16 OP_PICK OP_HASH160

    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160
    OP_FROMALTSTACK OP_IF OP_SWAP OP_ENDIF OP_CAT OP_HASH160

    <root_hash> OP_EQUALVERIFY

    OP_OVER OP_4 OP_PICK OP_EQUAL
    OP_NOT OP_VERIFY

    OP_DUP
    OP_TOALTSTACK
    OP_CHECKDATASIGVERIFY
    OP_FROMALTSTACK
    OP_CHECKDATASIG
OP_ENDIF
```

</details>

#### ZCE Root Hash

Every ZCE-Secured Transaction has exactly one (`1`) valid **ZCE Root Hash**, the root hash of a `HASH160` merkle tree containing all public keys which must be covered by the ZCE (See [ZCE-Secured Transactions](#zce-secured-transactions)).

For a ZCE to be considered valid, the merkle tree must be no deeper than `16` levels (allowing ZCE outputs to cover up to `65,536` public keys). Each public key in the tree must be unique and appear in lexicographical order. Unused tree leaves must appear after all covered public keys, and each unused leaf must be set to `0`.

<details>
 <summary>Root Hash Test Vectors</summary>

The following public key sets (ordered lexicographically) produce the indicated root hash. For each test vector, a script is provided to demonstrate correct merkle tree construction. (Note, for ZCE outputs covering only one public key, no merkle tree is required.)

| Root Hash                                    | Public Keys                                                                                                                                                                                                                                                                                                                                                            | Generation Script                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `0xe01b06b9dce4b03b169c1e9c9d59d7907b2b6e5b` | `0x020000000000000000000000000000000000000000000000000000000000000001`                                                                                                                                                                                                                                                                                                 | `<0x020000000000000000000000000000000000000000000000000000000000000001> OP_HASH160`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `0xaa458d0b6785d76baf316e75e6e49fc63b8c2090` | `0x020000000000000000000000000000000000000000000000000000000000000001`, `0x020000000000000000000000000000000000000000000000000000000000000002`                                                                                                                                                                                                                         | `<0x020000000000000000000000000000000000000000000000000000000000000001> OP_HASH160 <0x020000000000000000000000000000000000000000000000000000000000000002> OP_HASH160 OP_CAT OP_HASH160`                                                                                                                                                                                                                                                                                                                                                                                                                |
| `0xfd3369ef5802a6c314cfdb4bb33e2b6a2a2a435c` | `0x020000000000000000000000000000000000000000000000000000000000000001`, `0x020000000000000000000000000000000000000000000000000000000000000002`, `0x030000000000000000000000000000000000000000000000000000000000000001`                                                                                                                                                 | `<0x020000000000000000000000000000000000000000000000000000000000000001> OP_HASH160 <0x020000000000000000000000000000000000000000000000000000000000000002> OP_HASH160 OP_CAT OP_HASH160 <0x030000000000000000000000000000000000000000000000000000000000000001> OP_HASH160 <0> OP_HASH160 OP_CAT OP_HASH160 OP_CAT OP_HASH160`                                                                                                                                                                                                                                                                           |
| `0x1b4a64581e793a5a74e7cffa4872f48f48c9954f` | `0x020000000000000000000000000000000000000000000000000000000000000001`, `0x020000000000000000000000000000000000000000000000000000000000000002`, `0x030000000000000000000000000000000000000000000000000000000000000001`, `0x030000000000000000000000000000000000000000000000000000000000000002`                                                                         | `<0x020000000000000000000000000000000000000000000000000000000000000001> OP_HASH160 <0x020000000000000000000000000000000000000000000000000000000000000002> OP_HASH160 OP_CAT OP_HASH160 <0x030000000000000000000000000000000000000000000000000000000000000001> OP_HASH160 <0x030000000000000000000000000000000000000000000000000000000000000002> OP_HASH160 OP_CAT OP_HASH160 OP_CAT OP_HASH160`                                                                                                                                                                                                        |
| `0x37056d304b99c97ef1ca848ccd247412314a7d2f` | `0x020000000000000000000000000000000000000000000000000000000000000001`, `0x020000000000000000000000000000000000000000000000000000000000000002`, `0x030000000000000000000000000000000000000000000000000000000000000001`, `0x030000000000000000000000000000000000000000000000000000000000000002`, `0x030000000000000000000000000000000000000000000000000000000000000003` | `<0x020000000000000000000000000000000000000000000000000000000000000001> OP_HASH160 <0x020000000000000000000000000000000000000000000000000000000000000002> OP_HASH160 OP_CAT OP_HASH160 <0x030000000000000000000000000000000000000000000000000000000000000001> OP_HASH160 <0x030000000000000000000000000000000000000000000000000000000000000002> OP_HASH160 OP_CAT OP_HASH160 OP_CAT OP_HASH160 <0x030000000000000000000000000000000000000000000000000000000000000003> OP_HASH160 <0> OP_HASH160 OP_CAT OP_HASH160 <0> OP_HASH160 <0> OP_HASH160 OP_CAT OP_HASH160 OP_CAT OP_HASH160 OP_CAT OP_HASH160` |

</details>

#### ZCE-Secured Transactions

A **ZCE-Secured Transaction** adheres to the following validation requirements:

1. All transaction inputs are [ZCE-Secured Transaction Inputs](#zce-secured-transaction-inputs).
2. At least one output pays to the required P2SH ZCE contract:
   1. The correct [ZCE Root Hash](#zce-root-hash) is used, including the public key of every transaction input.
   2. The minimum key-count ZCE contract is used for the total count of public keys covered.
   3. The ZCE output has a value greater than or equal to the transaction's mining fee.

#### ZCE-Secured Transaction Inputs

All inputs in a **ZCE-Secured Transaction** must adhere to the following validation requirements:

1. Spends from a Pay-To-Public-Key-Hash (P2PKH) Unspent Transaction Output (UTXO) which either:
   1. has at least one (`1`) confirmation (the previous transaction has been included in a block), or
   2. is an output of a [ZCE-Secured Transaction](#zce-secured-transactions).
2. All signatures (in inputs) use only the `SIGHASH_ALL` (`0x01`) and `SIGHASH_FORKID` (`0x40`) [signing serialization algorithm](https://github.com/bitcoincashorg/bitcoincash.org/blob/3e2e6da8c38dab7ba12149d327bc4b259aaad684/spec/uahf-technical-spec.md) flags (inputs using `SIGHASH_NONE`, `SIGHASH_SINGLE`, or `SIGHASH_ANYONECANPAY` could evade enforcement of the ZCE contract).

### Wallet UTXO Selection

Wallets creating ZCE-secured transactions must implement the following UTXO selection criteria:

1. All selected UTXOs must use P2PKH contracts.
2. Each selected UTXOs must use a unique public key<sup>1</sup> which:
   1. has never previously produced a signature (e.g. made past transactions, including in other cryptocurrency networks), and
   2. will not produce another signature until the created ZCE-secured transaction has received 11 confirmations<sup>2</sup>.
3. All selected UTXOs must be either:
   1. confirmed (included in a block), or
   2. an output of a previous ZCE-secured transaction in which the ZCE output met the minimum escrow requirement of the current transaction.
4. The combined value of all selected UTXOs must cover the payment amount, the `instantAcceptanceEscrow` value, and double the required miner fee<sup>3</sup>.

<small>

1. The ZCE contract allows anyone (typically, the miner) to spend the ZCE output if the spender can prove two different messages have been signed by the same public key. If a ZCE includes two UTXOs which pay to the same public key hash (A.K.A. address), the ZCE can be immediately claimed using only the signatures provided in the transaction's inputs. ([Future upgrades](#eliminating-one-utxo-per-address-limitation) could enable ZCEs to safely use multiple UTXOs per address.)
2. This prevents the ZCE from being inadvertently lost by broadcasting later transactions with signatures which also match the ZCE's miner claim criteria. Additionally, because ZCEs offer bounties for miners which are often much larger than typical transaction fees, at least 11 confirmations are necessary to avoid incentivizing blockchain rewrites: if a large enough volume of ZCE-secured transactions have had their miner claim criteria quickly revealed, it can become profitable for miners to attempt deep block reorganizations to claim ZCEs. This window of opportunity closes after the 10-block rolling checkpoint is reached, preventing deep block-reorganizations from being accepted by the network.
3. To fulfill a payment request's [`instantAcceptanceEscrow` requirement](#payment-request), the ZCE output must be greater than or equal to the `instantAcceptanceEscrow` value in satoshis plus the transaction's miner fee.

</small>

### ZCE Extension to JSON Payment Protocol

The [JSON Payment Protocol (v2)](https://github.com/bitpay/jsonPaymentProtocol/blob/24014a7d29a0fac8a47523843aaafd5b7e07b901/v2/specification.md) is extended to enable ZCE-secured payments to be requested and returned.

#### Payment Request

In the BCH response to [Payment Requests](https://github.com/bitpay/jsonPaymentProtocol/blob/24014a7d29a0fac8a47523843aaafd5b7e07b901/v2/specification.md#payment-request), a new, optional `instantAcceptanceEscrow` field is added to the expected object provided in `instructions`. E.g.:

```json
{
  "time": "2019-06-13T18:35:19.138Z",
  "expires": "2019-06-13T18:50:19.138Z",
  "memo": "Payment request from Sandy's Snowmobiles for invoice TiXuyEmcJRCcinoFoY3Cym",
  "paymentUrl": "https://sandy.snow/i/TiXuyEmcJRCcinoFoY3Cym",
  "paymentId": "TiXuyEmcJRCcinoFoY3Cym",
  "chain": "BCH",
  "network": "testnet",
  "instructions": [
    {
      "type": "transaction",
      "requiredFeeRate": 1,
      "instantAcceptanceEscrow": 193300,
      "outputs": [
        {
          "amount": 193300,
          "address": "qpy4tkvn4z6xc7p42wvvx0m7kde9t6n5pvp4p2qq9y"
        }
      ]
    }
  ]
}
```

To fulfill the `instantAcceptanceEscrow` requirement, the wallet must construct a ZCE-secured transaction in which the value of the ZCE output is at least the value of **`instantAcceptanceEscrow` in satoshis plus the transaction's miner fee**. For example, to pay a request with an `instantAcceptanceEscrow` of `193300` with a 300 byte ZCE-secured transaction (at a fee rate of 1 satoshi/byte), the ZCE output must be at least `193600`.

> Note, a ZCE output value of only `instantAcceptanceEscrow` in satoshis cannot be safely accepted due to relay rules: if a previously broadcasted ZCE-secured transaction conflicts with the ZCE-secured transaction sent to the payee, the transaction sent to the payee would not be accepted by the network as a valid replacement. The ZCE output value must be at least `instantAcceptanceEscrow` plus the transaction's miner fee to guarantee network acceptance.

##### Chaining ZCE-Secured Transactions

Additionally, if P2PKH outputs of previous ZCE-secured transactions are used to fund the transaction fulfilling this payment request, **each funding ZCE-secured transaction must have used a ZCE output value greater than or equal to the value of `instantAcceptanceEscrow` in satoshis plus the fulfilling transaction's miner fee**.

#### Payment

In the BCH [Payment message](https://github.com/bitpay/jsonPaymentProtocol/blob/24014a7d29a0fac8a47523843aaafd5b7e07b901/v2/specification.md#payment), a new optional `escrowReclaimTx` field is added to the transaction object. E.g.:

```json
{
  "chain": "BCH",
  "currency": "BCH",
  "transactions": [
    {
      "tx": "0100000002de2120db373f867ed09b82b30a83e1e97a89dccbed4f4a2533def0d7273f324d00000000644140f00447e5d24a705ae95f1cbdb107fa07bc7539c51b1472dabd53d7c02dedbf2d2881e3c68eaced81d54c66630ea3cdf8b31a040e4b925b349bd4cf49b8b8db4121025bf454369c7b98325e3e7f33fe0eec8672bb83ae2d7147bd00a5b8eb781d6ca7ffffffff271ed8a25b659313af73a05c1b933da09e33870124f21b2263746cb2c854b0f101000000644109a2085e07f7bde9c5512e0b12049d9e2f22a9a0fb01e7966497a720969ddf5aaff23bad90acfac87e7be9f43ad5bc56b800e48f1517999f5947f058215b5ef6412103f0ae4b3c55f804139c91f57361462f1f9f4e883f540133d1268289af43c4676bffffffff0320150000000000001976a914f0de267cec0695d276a610977f485a4e6d733cd088aca02300000000000017a91489d8a237024b0e92beaa5b0c2b63d9b4cdb4e8388780c80200000000001976a9144955d993a8b46c78355398c33f7eb37255ea740b88ac00000000",
      "escrowReclaimTx": "01000000014ae1d7198c7b3b9534b608ddfde695e1c09fa8bc67e69cc9b167d87a0ea6a0ed01000000ac41170549c22b1634f78d6fb8b2382d1574492b9d7c69bea33dcf65509efc5ba28f9f52dacf7eb2ba3d6bda5169269c8b7ad6c9e021d6b032db224a0ee1feff8b55412102a2b27e8068d9721626f53d438e16d2962d01d649822beeaa77139b5f6c5b64bd4776a914c236e243a4e2746c226c42f3476267fb75a275138763ac676b5179a96c637c687ea914e10b88419957dfd6bbfe71e89addd6c42b1b5da488785479879169766bbb6cba68ffffffff019f220000000000001976a914a88a59cf3b991600ba09c3d5ebb1673e3c0c907588ac00000000"
    }
  ]
}
```

The `escrowReclaimTx` field must contain a transaction in which the ZCE output is spent (i.e. by the payer reclaiming the funds to their own wallet).

The `escrowReclaimTx` field is required for a payee to instantly accept a ZCE-secured transaction – by broadcasting the reclaim transaction at the same time as the ZCE-secured transaction, the network is alerted to the existence and `reclaim` public key of the ZCE output. If not provided, the payee should not accept the transaction as a ZCE-secured payment.

### ZCE Relay & Mining Policies

To be considered **ZCE-ready**, node software must implement specialized transaction relay/mining policies for ZCE-secured transactions.

1. When broadcasting a ZCE-secured transaction, implementations should first broadcast the ZCE reclaim transaction<sup>1</sup>.
2. During validation of newly-received transactions, if any inputs reclaim a ZCE output and the transaction providing the UTXO is not yet confirmed<sup>2</sup>, check if the parent transaction is a properly-formed, [ZCE-secured transaction](#zce-secured-transactions).
3. If a ZCE-secured transaction is received within 5 seconds<sup>3</sup> of a conflicting, non-ZCE-secured transaction, the ZCE-secured transaction must replace the first-seen transaction<sup>4</sup>. (Implementations must rebroadcast and/or add the replacement transaction to the candidate block.)
4. If a ZCE-secured transaction is received within 5 seconds of a conflicting, ZCE-secured transaction, and the second transaction contains a ZCE output with a greater value than the first-seen ZCE output plus the transaction fee of the second transaction<sup>5</sup>, the first-seen transaction is replaced. (Implementations must rebroadcast and/or add the replacement transaction to the candidate block.)
5. If a mining node either accepts a ZCE-secured replacement transaction or receives a double spend proof for a known ZCE-secured transaction, the node should create a transaction claiming the ZCE to its mining wallet<sup>6</sup>.

<details>
<summary>Notes</summary>

1. Even if the receiving peer has already accepted a transaction which conflicts with the ZCE-secured transaction, this ordering allows for the peer to efficiently identify that the parent transaction is ZCE-secured.
2. It is only valuable to inspect transactions which are currently in the mempool for ZCE compatibility, as these transactions require modified relay/mining behaviors. Historical transactions which have been confirmed in a block do not require any further treatment in this context.
3. Implementations should maintain a rolling list of transactions which were received within the past 5 seconds. The 5 second period is based on [existing usage and real-world testing](#5-second-transaction-replacement-period).
4. In this case, the ZCE has been violated by an already existing transaction and can be immediately claimed by a miner.
5. To prevent Denial of Service attacks, ZCE transaction replacement is limited using transaction fees. By requiring the ZCE output to grow by at least the value of the transaction's fee, this policy limits the excess bandwidth/validation resources which can be wasted on sequences of ZCE replacements. As ZCE outputs of replaced transactions will always be claimed by the miner, this strategy imposes the same cost on Denial of Service attacks which would otherwise be imposed on transactions which are ultimately mined.
6. Demonstrations of `Miner Claim` constructions are available in the [ZCE authentication template](https://ide.bitauth.com/import-gist/104baff7503d6a7ad619ad814153b059).

</details>

### Accepting ZCE-Secured Payments

ZCE-secured transactions offer instant, incentive-secure payment finality.

To accept ZCE-secured transactions, businesses should monitor the network using a fully-validating node. While ZCEs eliminate the need to delay purchases to [monitor the network for conflicting transactions](#transaction-conflict-monitoring), if the monitoring node has already received a transaction which conflicts with a ZCE-secured transaction at the time of payment, the purchase must be delayed until one of the transactions is confirmed<sup>1</sup>. (Note, any conflict with a ZCE-secured transaction implies deliberate, attempted fraud.)

ZCE-secured transactions should be accepted using the [ZCE Extension to the JSON Payment Protocol](#zce-extension-to-json-payment-protocol-v2).

When setting an [`instantAcceptanceEscrow`](#payment-request) value, businesses should require an escrow value which is **at least equal to the payment amount**. Businesses with exceptional vandalism risk (i.e. well-funded adversaries willing to burn funds to vandalize the business<sup>2</sup>) should set `instantAcceptanceEscrow` to a value larger than 100% of the payment.

Upon receipt of the [`Payment` message](#payment), the payee must:

1. Validate `escrowReclaimTx`:
   1. at least one input must spend the ZCE output of the ZCE-secured transaction, and
   2. the redeem bytecode must match the expected ZCE contract for the ZCE-secured transaction's input count.
   3. Extract the `reclaim` public key from the ZCE input.
2. Validate the ZCE-secured transaction:
   1. Validate that all inputs meet the requirements of [ZCE-Secured Transaction Inputs](#zce-secured-transaction-inputs).
   2. Construct the expected [`ZCE Root Hash`](#zce-root-hash) given all input public keys.
   3. Given the `reclaim` public key found in the `escrowReclaimTx`, the expected `ZCE Root Hash`, confirm the ZCE output pays to the expected P2SH contract (using the minimum-sized ZCE contract for the transaction's input count).
3. Attempt to broadcast both transactions.

If the broadcast is successful, the payment can be considered ZCE-secured.

<small>

1. Even if the conflicting transaction were broadcasted less than 5 seconds before the ZCE-secured transaction was received, it could not be guaranteed that the entire network would hear the ZCE-secured transaction in time to replace the conflicting transaction, and the double-spend could be successful. As such, payees should not ignore this attempted fraud.
2. While ZCEs [generally prevent attackers from profiting](#full-value-escrow-requirement) via zero-confirmation fraud, a well-funded attacker can vandalize a business by simultaneously broadcasting a conflicting ZCE-secured transaction – paying back to the attacker – with an equal or greater escrow value than the transaction sent to the business. This attack is guaranteed to lose the attacker at least the value of the payment (due to miners claiming the ZCE), and may be detected by the business before the double-spent payment is honored (the attack can always be [detected within 5 seconds](#monitoring-for-replacement-transactions)). The attack also risks a 200% loss to the attacker if a miner ultimately accepts the legitimate transaction (in which case, the business still receives their expected payment).

</small>

## Rationale

This section documents design decisions made in this specification.

### Transaction Replacement Period

The [5 second transaction replacement period](#zce-relay--mining-policies) is chosen based on [existing usage and real-world testing](#transaction-conflict-monitoring). Because 1) all mining nodes appear to hear transactions in less than 3 seconds, 2) the network can be relied upon to accept ZCE-secured transaction replacements for around 5 seconds, and 3) payees delay any transactions with a previously-seen conflict, the replacement period offers immediate confidence in any valid ZCE-secured transaction.

Additionally, existing payment processors and point-of-sale packages typically monitor the network for 5 to 10 seconds before accepting a zero-confirmation transaction. By only allowing replacement within a 5 second window, the security of existing systems is not reduced.

### Full-Value Escrow Requirement

This specification recommends that payees require ZCEs of at least equivalent value to the payment being secured ("full-value escrows"). While escrow values below 100% of the payment ("partial-value escrows") coupled with [briefly monitoring for replacement transactions](#monitoring-for-replacement-transactions) could offer some finality, the fraud-risk of such a solution is difficult to measure and changes with network conditions<sup>1</sup>.

Partial-value and non-escrowed zero-confirmation commerce incentivizes the development of payment fraud infrastructure. Though honest network nodes refuse to rebroadcast conflicting transactions (favoring the "first-seen" transaction, even if that transaction contains a lower fee), it is trivial for an attacker to directly share double-spending transactions with a miner willing to accept the later transaction.

Because mining is highly competitive, additional income from this sort of "fraud-as-a-service" agreement<sup>2</sup> could become a serious competitive advantage, particularly as transaction fees continue their growth as a proportion of block rewards.

By requiring full-value escrows, this proposal introduces significant friction between buyers and sellers of fraud-as-a-service mining: attackers are motivated to pay less than 100% of the purchase price, but any discount offered by a fraud-assisting miner would earn the miner less than their profit to betray the attacker (by mining the ZCE-secured transaction). Though such miners could successfully operate on reputation, the development and growth of such pools remains regulated by exit scam risk (attackers risk 200% of each payment for the chance at a discount)<sup>3</sup>.

Finally, it should be noted that the full-value escrow requirement does not interfere with typical wallet usage: it is uncommon for buyers to spend more than half of a wallet or account balance in any single point-of-sale transaction. Larger expenditures (e.g. rent, mortgage, utility bills, service invoices, insurance premiums, etc.) tend to allow for larger payment windows than retail settings and can wait for transactions to be confirmed in a block rather than employing ZCEs.

<small>

1. Beyond purchase context (disposability of stolen goods, length of customer interaction, customer demographics, the customer-business relationship, and presence of external fraud-mitigation measures), fraud risk for partial-value escrows changes with network conditions like global volume of zero-confirmation commerce, global average escrow percentage, SHA256 dominance of BCH, ratio of anonymous to publicly identified hash power, and ratio of block subsidy to transaction fees. These factors influence the cost and profitability of fraud-as-a-service mining operations.
2. Fraud-as-a-service mining has been [detected in the past](https://web.archive.org/web/20210817210029/https://bitcointalk.org/index.php?topic=327767.0&all=&__cf_chl_jschl_tk__=pmd_rc8NuOrHI5eVXxF7rTfrF4cpufLxBcP_Bg_zE5s1W24-1629233986-0-gqNtZGzNAfujcnBszQcR). Given a large enough base of partial-value and non-escrowed zero-confirmation commerce, such operations are likely to become more widespread, harder to detect, and easier for criminals to use.
3. Strategies which attempt to reduce this exit scam risk also appear to be vulnerable to betrayal. While attackers can't prevent miners from seeing and mining the ZCE-secured transaction broadcasted to the network, an adjacent contract could be designed to punish miners which betray the attacker by mining the ZCE-secured transaction. However, any contract which can punish the fraud-assisting miner by proving that a ZCE-secured transaction has been mined (e.g. in a block header with correct lineage and adequate proof of work) can also be used to betray the fraud-assisting miner by colluding with a different miner.

</small>

### Monitoring for Replacement Transactions

Because ZCE-secured transactions can be replaced for up to 5 seconds, the theoretical cost of fraud rises 5 seconds after an initial ZCE-secured transaction is broadcasted.

For example: an attacker makes a $5 purchase which is secured by a $5 ZCE. Within 5 seconds of the initial broadcast, the attacker can broadcast a conflicting ZCE-secured transaction secured by a $5.01 ZCE which pays funds back to their own wallet. (This fraud costs the attacker slightly more than the original $5 payment, but the attacker may have non-monetary motives.)

After 5 seconds, when ZCE replacement over the network is no longer possible, the maximum cost of fraud to the attacker rises to $10 (the original payment + ZCE) – while fraud-as-a-service mining pools could offer a chance at a discount from the $5 payment, the attacker always risks $10 by sharing a double spend with a mining pool.

This property offers another simple strategy for improving ZCE transaction security: payees could monitor the network for ~5 seconds prior to releasing goods, ensuring attackers are always forced to pay the higher cost. This mode is excluded from the specification because it re-introduces a waiting period for users, harming user experience. Security is likely better improved by increasing `instantAcceptanceEscrow`, but some rare use cases may both require additional security and find the additional delay preferable to increasing escrow values.

### Limitation on Immediate Reuse of Escrowed Funds

While other (P2PKH) outputs of ZCE-secured transactions can be immediately reused in later ZCE-secured transactions, using funds locked in the ZCE contracts itself before it is confirmed in a block would weaken the security of later ZCE-secured transactions.

For example, Alice sends a $5 ZCE-secured transaction to Bob with a $5 ZCE. She immediately reclaims the ZCE, then uses it to fund a $2 transaction to Charlie with a $2 ZCE. If Alice attempts to double-spend the transaction sent to Bob, the miner of the next block may claim the ZCE output of Bob's transaction. While Bob would still be paid $5, the payment to Charlie is invalidated without recourse.

While the ZCE protocol could be extended to allow for immediate reuse of escrowed funds – e.g. by requiring later ZCE-secured transactions to also commit to public keys used in parent transactions – such schemes are significantly harder to reason about, add combinatorial complexity to validation (for both network nodes and payees), and would only offer slightly higher liquidity for most ZCE use cases. Instead, this proposal only allows ZCE-secured transactions to be funded using confirmed outputs or the change outputs of other ZCE-secured transactions.

In practice, this limitation is both simple and unlikely to impact regular usage: up to half of the confirmed funds in a wallet can be used for any number of ZCE-secured transactions within a single block period (~10 minutes), and the other half always remains immediately spendable in non-ZCE-secured transactions.

## Areas for Research

The following topics have been excluded from this specification pending further research and development. Future specifications may incorporate these features.

### Eliminating One-UTXO-Per-Address Limitation

ZCE-secured transactions can only safely spend [one UTXO per address](#wallet-utxo-selection). Because the [signing serialization algorithm](https://github.com/bitcoincashorg/bitcoincash.org/blob/3e2e6da8c38dab7ba12149d327bc4b259aaad684/spec/replay-protected-sighash.md) produces a different preimage for each transaction input, if multiple inputs share a public key, the ZCE can be claimed using only the signatures present in the ZCE-secured transaction's inputs.

If a future upgrade enables [PMv3's Detached Signatures](https://github.com/bitjson/pmv3#detached-signatures), this ZCE limitation can be eliminated: a wallet could include any number of UTXOs which share the same public key, and all UTXOs can be unlocked using a single detached signature.

### Reducing ZCE Overhead

In normal operation, between [63 and 177 bytes of the ZCE protocol overhead](#increased-transaction-sizes) is dedicated to sharing the unexecuted code paths of ZCE contracts (the miner claim clauses).

Merklized Abstract Syntax Trees ([MAST](https://bitcoinops.org/en/topics/mast/)), [Taproot on BCH](https://gist.github.com/markblundeberg/94650e69ebf056215de6ad1a716de559), or other similar upgrades would allow this overhead to be removed from normal operation, reducing average overhead from ~295 bytes to ~217 bytes.

### Miner Enforcement of ZCE Security

As specified, ZCE-secured transactions remain vulnerable to [some types of miner collusion](#full-value-escrow-requirement) (with a probability of success equal to the colluding miner's portion of network hash power).

If a notable "fraud-as-a-service" miner were ever detected on the network, an additional mining policy could be implemented to solidify ZCE security: miners could ignore blocks which fail to claim sufficiently-aged ZCEs beyond some limit.

Because all miners are expected to eventually hear all transactions, blocks which fail to claim a significant sum of value from ZCEs of sufficient age can be assumed to originate from a miner engaged in zero-confirmation payment fraud. (A miner forgoing significant on-chain profits indicates that they are being paid a larger sum off-chain to modify their behavior.)

To ameliorate this fraud, honest miners can profitably provide a valuable service: ignore the offending block, claiming the ZCEs themselves in the next block. If all honest miners expect this behavior (and reasonable timing and value limits are established), the network can be expected to successfully drop the offending transactions.

The other miners can expect to make a profit at the fraudsters' expense, and **any users who were defrauded will automatically receive the payment they originally expected**.

## Implementations

Please see the following reference implementations for additional examples and test vectors:

- [Bitcore PR #3240 - ZCE](https://github.com/bitpay/bitcore/pull/3240)

**_(seeking contributors for node implementation patches)_**

## Evaluation of Alternatives

No alternative proposals are currently active, but several past proposals have informed the design of this proposal:

- [Double-spending Prevention for Bitcoin zero-confirmation transactions](https://eprint.iacr.org/2017/394.pdf) – a scheme which forces the user to reveal a private key if they attempt to double-spend (though requires a previous setup transaction).
- [Zero-Confirmation Forfeits (ZCF)](https://gist.github.com/awemany/619a5722d129dec25abf5de211d971bd) – a proof-of-concept which inspired the contract construction and payment flow of Zero-Confirmation Escrows.

Other potential alternatives which have been proposed for networks like Bitcoin Cash include:

- [Lightning Network](https://en.wikipedia.org/wiki/Lightning_Network) – a payment channel network which offers similar finality as ZCE-secured transactions (but requires previous setup transactions, channel liquidity between payer and payee, and a constantly-online service to monitor for fraud).
- [Avalanche](https://www.avalabs.org/whitepapers) – a secondary consensus layer in which a set of recent miners settles disputes between conflicting transactions. (Though no public proposal exists for applying the Avalanche protocol to a bitcoin-like Proof-of-Work network.)

## Stakeholders & Statements

_(TODO, pull requests welcome)_

## Feedback & Reviews

- [`CHIP 2021-06 Zero-Confirmation Escrows` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/)

## Changelog

This section summarizes the evolution of this document.

- **v1.0.0 – 2021-8-18** ([`053927b5`](https://github.com/bitjson/bch-zce/blob/053927b57b5c7fe9f70e2dd77242e84c507d3502/readme.md))
  - Initial publication

## Copyright

This document is placed in the public domain.
