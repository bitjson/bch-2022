# CHIP - 2021-03 - BIGGER SCRIPT INTEGERS

## Summary

Currently, math operations within Bitcoin Cash's Script VM are limited to 32-bit signed integers. This proposal adds support for bigger integers to accommodate high-value smart contracts.

> OWNERS: [Rosco Kalis](https://github.com/rkalis), [Jonathan Silverblood](https://gitlab.com/monsterbitar)
>
> DISCUSSION: [Bitcoin Cash Research](TODO), [Telegram](https://t.me/joinchat/SeRFuFmba7tWOcKh)
>
> MILESTONES: **Draft**, Published (March), Testnet (August), Specification (October), Accepted (November), Deployed (May 15th, 2022).

## Motivation and Benefits

Currently, arithmetic inside Script is limited to signed 32 bit integers. There are no significant systems in other domains that natively have these strict limitations. Covenant contracts often perform arithmetic on a UTXO's balance or other arithmetic to determine transaction outputs. Because of these limitations, covenant contracts are effectively limited to balances and outputs of ~21 BCH (at current prices, ~$10k).

Some **contract developers** (like General Protocols / AnyHedge) attempt to work around these limitations by applying convoluted math tricks that allow them to circumvent these limits at the cost of added complexity, slight math errors and an increased risk of smart contract bugs. And even with those workarounds this limitation is not fully mitigated¹.

Other **contract developers** do not address these limits in their contract implementation (like Licho / Mecenas & Last Will), but instead make sure not to send more than ~21 BCH to these contracts.

**Contract users** suffer from these limitations in different ways. The workarounds cause slight math errors¹, meaning that payouts of these contracts can differ from their expectations. They also may not be able to create contracts of the size that they would like, instead having to create multiple smaller contracts. In a worst case if the **contract user** is not aware of these limits (and no "emergency hatch" is implemented in the contract), it is possible for their money to get permanently stuck inside a contract if they deposit more than the limit.

While BCH uses 8 "decimals", some SLP tokens (like SPICE) use more decimals. This means that using the same 32 bit numbers, even less value can be represented, making the existing limits even more restrictive when working with SLP covenants.

Increasing these limits would allow **contract developers** to create contracts that can hold larger amounts of value. They will also be able to create contracts that are simpler to reason about and are more accurate, because they do not need to use convoluted and inaccurate workarounds. This in turn lowers the barrier to entry for new **contract developers** to enter the market and write more valuable (DeFi) contracts. This increases the utility of BCH and can cause the price of BCH to rise, benefiting **BCH investors**.

**Contract users** will be able to deposit larger sums into contracts. They do not need to worry about their funds getting stuck *because of these limits*. They will also get more accurate results/payouts out of these contracts.

The mentioned workarounds take up quite a bit of opcodes², so removing the need for these workarounds lowers transaction sizes for the affected contracts. This benefits **contract developers** by freeing up space for other smart contract functionality, **contract users** by lowering transaction fees, and **node operators** by reducing storage costs for these transactions.

There are no benefits to the other groups of stakeholders.

    ¹ TODO: Link to Karol's math analysis

    ² In case of AnyHedge, the workaround needs ~60 opcodes and ~100 bytes

## Costs and Risks

### Implementation costs and risks

**Node developers** will need to implement the new functionality in all nodes to stay in consensus. BCH has at least six different node implementations (BCHN, BU, BCHD, Flowee, Verde, Knuth) in three different languages (C++, Go, Java) and not all languages have the same native support for bigger integers.

How big these changes are depend on the technical details of the proposal, but these changes are likely to be significant. Depending on the technical details it is possible that new supporting opcodes need to be introduced.

Some Bitcoin Cash libraries have support for Script execution. **Developers of these libraries** have to go through the same process nodes need to go through in terms of implementing new functionality. There are at least five such libraries (bitcore-lib-cash, libauth, bitcoincashj, python-bitcoincash, iguana) in four different languages (JavaScript, Java, Python, Rust).

Other Bitcoin Cash libraries do not have support for Script execution, but do support Script (de)serialisation and other related operations. These libraries usually have checks in place for things such as integer sizes. **Developers of these libraries** need to update these checks and some other minor operations to account for bigger integer support.

Depending on how backwards compatibility will be dealt with, it is possible that **contract developers or users** need to take some action.

There are no implementation costs to the other groups of stakeholders.

### Ongoing Costs and Risks

**Node developers** and **library developers** need to make significant changes to the Script VM. This adds complexity and extra work for any future upgrades that these developers might want to implement on the Script VM specifically.

Much of the rationale behind this update is based on being able to handle larger amounts of BCH or SLP tokens inside covenant contracts. Should BCH move to fractional satoshis (or fractoshis) in the future, larger numbers will be required to represent the same amount of value. So depending on technical details of this proposal, it could increase or decrease the complexity of moving to fractoshis in the future.

Regardless of technical details, bigger integer operations are more computationally expensive, but how much more does depend on technical details. Processing transactions that use these bigger integers (a small % of all transactions) will become more computationally expensive. So **node operators** and **miners** will spend marginally more resources to process these transactions.

If block and transaction processing turns out to be bottlenecked by the presence of big integers, this can slightly impact miner profitability and push a marginal amount of hashrate off of BCH to other SHA256 chains. However, it is highly unlikely that script processing of big integers will be the bottleneck for block or transaction processing.

But because these operations are still more computationally expensive, it is important to make sure that the implementation does not open up any DOS vectors that can impact **node operators** or **miners**.

There are no ongoing or future costs to the other groups of stakeholders.

## Technical Description

*(Technical description will be provided after a technical proposal has been selected)*

## Implementations

- [Andrew stone](https://github.com/gandrewstone) has specified and implemented a [BigNum proposal on his next-chain testnet](http://nextchain.cash/bignum).
- [Tobias Ruck](https://github.com/EyeOfPython) has specified a proposal for [128 bit integers](https://gist.github.com/rkalis/eabdbba283f3807c0e38bd677672b6ae).

### Specification

*(Specification will be provided after a technical proposal has been selected)*

### Test Cases

*(Test cases will be provided after a technical proposal has been selected)*

## Evaluation of Alternatives

*(Evaluation of alternatives will be provided after a technical proposal has been selected)*

## Security considerations

*(Evaluation of alternatives will be provided after a technical proposal has been selected)*

## List of major stakeholders

There are at least five major stakeholder groups:

### Node Developers

There are six different node teams that need to update their code.

- [Bitcoin Cash Node](https://bitcoincashnode.org/)
- [Bitcoin Unlimited](https://www.bitcoinunlimited.info/)
- [BCHD](https://bchd.cash/)
- [Flowee](https://flowee.org/)
- [Bitcoin Verde](https://bitcoinverde.org/)
- [Knuth](https://kth.cash/)

### Library Developers

There are five different libraries that need to update their code.

- [BitPay](https://bitpay.com/) developed the [bitcore-lib-cash](https://github.com/bitpay/bitcore/tree/master/packages/bitcore-lib-cash) library, which supports Script execution.
- [Jason Dreyzehner](https://github.com/bitjson) developed the [libauth](https://github.com/bitauth/libauth) library, which supports Script execution.
- [Pokkst](https://github.com/pokkst) developed the [bitcoincashj](https://github.com/pokkst/bitcoincashj) library, which supports Script execution.
- [Dagur Valberg Johansson](https://github.com/dagurval/) developed the [python-bitcoincash](https://github.com/dagurval/python-bitcoincash) library, which supports Script execution.
- [Tobias Ruck](https://github.com/eyeofpython) developed the [Iguana](https://github.com/be-cash/iguana) library, which supports Script execution.

### Contract Developers

There are some organizations and independent developers that are currently building products that are limited by the current integer size limits.

- [General Protocols](https://generalprotocols.com/) created [AnyHedge](https://anyhedge.com/), a volatility risk-trading contract. AnyHedge contracts are limited to ~$15k and have a slight math error due to workarounds.
- [Licho](https://github.com/KarolTrzeszczkowski) created a [Last Will](https://github.com/KarolTrzeszczkowski/Electron-Cash-Last-Will-Plugin) contract to manage inheritance and the [Mecenas](https://github.com/KarolTrzeszczkowski/Mecenas-recurring-payment-EC-plugin) contract for recurring payments. These contracts are limited to ~21 BCH (~$10k).
- [Shomari Prince](https://github.com/nyusternie) created [Causes Cash](https://causes.cash/), which includes a modified Mecenas to support recurring payments in USD. These contracts are limited to ~21 BCH (~$10k).
- [James Cramer](https://github.com/jcramer/) created experimental [SLP Mint Guard](https://github.com/simpleledger/Electron-Cash-SLP/blob/cashscript-dev/lib/cashscript/slp_mint_guard.cash) contracts and [tokens with minting schedules](https://github.com/simpleledgerinc/slp-mint-contracts). These contracts are currently not limited, but to execute the proposed roadmap they will be very limited as they will need to perform arithmetic on SLP token amounts (which can have more decimals and lower USD values than BCH).
- [p0oker](https://github.com/p0o/) created an [SLP vending contract](https://github.com/p0o/yield-farming-bch-smart-contract) that mints tokens on-demand and is building a BCH staking contract that mints tokens over time and an SLP exchange contract to sell NFTs. Similar to James Cramer's contracts, these contracts will be very limited as they need to work with SLP amounts.
- [Jason Dreyzehner](https://github.com/bitjson) created [CashChannels](https://blog.bitjson.com/cashchannels-recurring-payments-for-bitcoin-cash-3b274fbfa6e2), recurring payments for Bitcoin Cash. These channels are limited to ~21 BCH (~$10k).

### Node Operators & Miners

TBD

### Investors

There are people and organizations that have invested into the BCH token and ecosystem on the premise that it is expected to become peer to peer electronic cash for the world. Miners are a clear such group who have invested into the ecosystem. These stakeholders expect the token to get adopted in all the same places where money is traditionally used, which includes in financial services.

## Statements

### Node Developers

TBD

### Library Developers

TBD

### Contract Developers

#### General Protocols
Developing workarounds to this limitation has cost General Protocols a large amount of time and money. This is still ongoing as the added complexity makes further smart contract changes more difficult. Because of the required code to address this, there are also other contract features that do not fit within the contract bytecode size limits.

#### Others: TBD

### Node Operators & Miners

TBD

### Investors

TBD

## Copyright Notice

Copyright (c) 2021 GeneralProtocols / Research

Permission is granted to copy, distribute and/or modify this document under the terms of the [MIT license](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/LICENSE).
