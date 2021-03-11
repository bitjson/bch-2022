# CHIP - 2021-02 - ADD NATIVE INTROSPECTION OPCODES

## Summary

This proposal updates the Bitcoin Cash scripting language with additional opcodes to natively provide details about the current transaction, such as the output amount and recipients.

> OWNERS: [Jason Dreyzehner](https://gist.github.com/bitjson), [Jonathan Silverblood](https://gitlab.com/monsterbitar)
>
> DISCUSSION: [Bitcoin Cash Research](https://bitcoincashresearch.org/t/native-introspection-chip-discussion/307), [Telegram](https://t.me/transactionintrospection)
>
> MILESTONES: **[Published](https://gitlab.com/GeneralProtocols/research/-/blob/master/CHIPs/May%202022,%20Native%20Introspection.md)**, Testnet (August), Specification (October), Accepted (November), Deployed (May 15th, 2022).

## Motivation and Benefits

In order for Bitcoin Cash to gain adoption as money, it needs to provide similar or better features than existing alternatives. One area where Bitcoin Cash can be uniquely positioned is as a cost-efficient programmable money. By improving the Bitcoin Cash script with additional opcodes to provide native introspection, we gain the following benefits:

- Reduces barrier to entry for new contract developers who no longer have to learn intimate details about transaction signing.

- Improves contract safety by eliminating the risk of a naively implemented workaround not doing proper verification.

- Allows for new usecases to be developed by providing more information on a transaction than the current workaround.

- Allows for larger and more complex contracts by removing at least 11 opcodes¹ and 90~300 bytes¹ used per transaction, compared to current workaround.

- Lowers the network bandwidth and storage costs for the growing number² of introspection transactions.

- Lowers network processing costs by removing one signature verification for the growing number² of introspection transactions.

  ¹ *An optimised preimage validation needs 8 opcodes/10 bytes `<pubkey> <sig> <preimage> 3DUP SHA256 ROT CHECKDATASIGVERIFY DROP <sighashflags> CAT SWAP CHECKSIGVERIFY` and extracting a single element from the middle (e.g. `hashOutputs`) needs 4 opcodes / 8 bytes (`<preimage> <splitindex> SPLIT NIP 32 SPLIT DROP`). A preimage is at least 153 bytes in size (which in some cases can be partially generated using ANYONECANPAY and NUM2BIN, bringing it down to ~90 bytes. This often cannot be used though). Often, however, the preimage of `hashOutputs` is also provided, increasing the used bytes by the size of the serialized outputs*. In practice, preimage decoding and verification might be less optimised.

  ² *There is currently ~100,000 such transactions, but introspection could be used for recurring transactions, such as rent, utilities and netflix subscriptions, which could have a dramatic impact on the network scalability.*

## Costs and Risks

To ensure that the outcome of this proposal is clearly beneficial to the long-term value of Bitcoin Cash the following costs have been taking into consideration and have been deemed acceptable:

### Implementation costs and risks

- Native introspection will use up 10~20 of the 40 currently unused² opcodes.

- Node software and other services that validate consensus would need to implement the new opcodes.

- A limited¹ number of libraries and developer tools that offer advanced scripting functionality would need to implement the feature.

  ¹ *SPV wallets, mining pools, exchanges and services does not have to be updated and will continue to function as normal.*

  ² *The opcodes in the `0xBD - 0xEF` range are currently unused (32 opcodes), according to [public documentation](https://documentation.cash/protocol/blockchain/script#operation-codes-opcodes). There are also the NOP1 and NOP4-NOP10 opcodes (8 opcodes) which can be used for softforking new features. Finally there is a handful of disabled opcodes that could potentially be used but that we leave out of this calculation for simplicity.

### Ongoing Costs and Risks

- Adding native introspection could increase complexity for any future technical changes to the Bitcoin Cash transaction format.

## Technical Description

> **NOTE**: This content of this section is still being determined. The data below is a list of possible things that could, in theory, be included in this proposal. Until a survey has been done among stakeholders as to which items have value, it is a good idea not to get attached to any particular item below.


Native Introspection is a set of new opcodes for the BCH virtual machine which provides direct access to elements of the virtual machine’s state during evaluation.
Each opcode provides information about the current transaction according to the table below.

Word | Value | Hex | Input | Output | Description
--- | --- | --- | --- | --- | ---
OP_OUTPOINTTXHASH | 189 | 0xBD |  |  | Push the outpoint transaction hash – the hash of the transaction which created the Unspent Transaction Output (UTXO) of the input being evaluated - to the stack in big-endian byte order.
OP_OUTPOINTINDEX | 190 | 0xBE |  |  | Push the outpoint index – the index of the Unspent Transaction Output (UTXO) of the input being evaluated – to the stack.
OP_INPUTINDEX | 191 | 0xBF |  |  | Push the index of the input being evaluated to the stack as a Script Number.
OP_INPUTSEQUENCENUMBER | 192 | 0xC0 | x | sequence | Pop the top item from the stack as an index (Script Number). Push the sequence number of the input at that index to the stack.
OP_INPUTBYTECODE | 193 | 0xC1 | x | bytecode | Pop the top item from the stack as an index (Script Number). Push the unlocking bytecode of the input at that index to the stack.
OP_UTXOBYTECODE | 194 | 0xC2 |  |  | Push the bytecode currently being evaluated to the stack. For standard scripts, this is the locking bytecode of the Unspent Transaction Output (UTXO) spent by the input, for P2SH scripts, the UTXO's redeem script is pushed.
OP_UTXOVALUE | 195 | 0xC3 |  |  | Push the value (as a Script Number) of the Unspent Transaction Output (UTXO) spent by the input being evaluated.
OP_OUTPUTBYTECODE | 196 | 0xC4 | x | lockscript | Pop the top item from the stack as an index (Script Number). Push the locking bytecode of the output at that index to the stack.
OP_OUTPUTVALUE | 197 | 0xC5 | x | outputvalue | Pop the top item from the stack as an index (Script Number). Push the value (in satoshis) of the output at that index to the stack as an 8 byte serialized integer. This can be interpreted as a Script Number by performing `OP_BIN2NUM` as long as the numerical value fits within the Script Number limits (currently <21 BCH).
OP_TXINPUTCOUNT | 198 | 0xC6 |  |  | Push the number of inputs of the current transaction as a Script Number
OP_TXOUTPUTCOUNT | 199 | 0xC7 |  |  |  Push the number of outputs of the current transaction as a Script Number
OP_TXLOCKTIME | 200 | 0xC8 |  |  | Push the locktime of the current transaction as a Script Number
OP_TXVERSION | 201 | 0xC9 |  |  | Push the version field of the current transaction as a Script Number
OP_NUM2VARINT | 202 | 0xCA | x | x varint | Pop the top item from the stack as a Script Number. Re-encode the number as a Bitcoin VarInt and push the result.
OP_VARINT2NUM | 203 | 0xCB | x | x scriptnum | Pop the top item from the stack as a Bitcoin VarInt. Re-encode the number as a Script Number and push the result.

### Aggregated values

*NOTE: The following is not part of jasons initial list, but might be interesting to add.*

Word | Value | Hex | Input | Output | Description
--- | --- | --- | --- | --- | ---
OP_TXINPUTVALUE | ? | 0x?? |  |  | Push the total number of satoshis in all inputs of the current transaction as a Script Number?
OP_TXOUTPUTVALUE | ? | 0x?? |  |  | Push the total number of satoshis in all outputs of the current transaction as a Script Number?
OP_TXINPUTSTACKITEM | ? | 0x?? | input item |  | Pop the item index and input index from the stack. Push the stack item with the item index pushed by input with index input index to the stack.

### Hashed values

*NOTE: The following is not part of jasons initial list, but might be interesting to add.*

Several state identifiers represent the hash of other state items (Transaction Outpoints Hash, Transaction Sequence Numbers Hash, Corresponding Output Hash, and Transaction Outputs Hash).
This allows scripts to avoid manually re-hashing the values (e.g. <2> OP_PUSHSTATE OP_HASH256).
This optimization reduces transaction sizes by eliminating the hashing opcode, incentivizes better performance, and makes performance optimizations easier for implementations.

In most cases, the virtual machine will be required to perform these hash functions during a signature checking operation (with a few exceptions, e.g. Corresponding Output Hash in an input which doesn't utilize "SIGHASH_SINGLE").
By allowing scripts to request the hashed result directly, scripts are incentivized to avoid harder-to-optimize constructions (e.g. `<2> OP_PUSHSTATE OP_SHA256 OP_SHA256`).

Word | Value | Hex | Input | Output | Description
--- | --- | --- | --- | --- | ---
Transaction Sequence Numbers Hash? | ? | 0x?? |  |  | ...
Corresponding Output Hash? | ? | 0x?? |  |  | ...
Transaction Outputs Hash? | ? | 0x?? |  |  | ...

### Templated version

*NOTE: The following is not part of jasons initial list, but might be interesting to add.*

At the cost of an additional opcode, it would be possible to push multiple values at the same time, resulting in small scripts.
This could be particulary useful if you need to use a piece of information more than one time, but in different places, as you could push the same data to both places with the same push.

Word | Value | Hex | Input | Output | Description
--- | --- | --- | --- | --- | ---
OP_PUSHSTATE | ? | 0x?? |  |  | Pop the top item from the stack as a state concatenation template. If each byte of the template is recognized, push each identified state value to the stack, otherwise, error.


## Implementations

- [Andrew stone](https://github.com/gandrewstone) have implemented similar features on his [next-chain testnet](http://nextchain.cash/).
- [Jason Dreyzehner](https://github.com/bitjson) has implemented similar features in his bitcoin VM library.

### Specification

*(Specification will be provided after a few remaining design choices have been finalized)*

### Test Cases

*(Test cases will be provided after a few remaining design choices have been finalized)*

## Evaluation of Alternatives

There has been similar proposals in the past that has been used to inform the design of this proposal:

- [Jason Dreyzehner](https://github.com/bitjson) created the [initial OP_PUSHSTATE](https://github.com/bitjson/op-pushstate) proposal. It was a good starting point and much of the work has been carried over to this proposal.

- [Tobias Rust](https://github.com/EyeOfPython) made a [multibyte version](https://github.com/slpdex/op-pushstate). _(.. add explanation of why that is not a better proposal ..)_

However, any delay or rejection of native introspection support may incur the following costs:

- Continued use of the inefficient workaround makes the blockchain larger than it needs to be and could have a negative impact on initial block download times and storage requirements.

- Requirement to use a difficult technical workaround in order to achieve transaction introspection could result in loss of opportunity as development costs and barrier to entry remains high.

- Complexity of the technical workaround in order to achieve transaction introspection could result in otherwise successful businesses having technical problems with their implementation that results in loss of profits and damages reputation of both the company and Bitcoin Cash as a network.

- Some applications and usecases are not possible with the workaround but depending on technical implementation details may be possible with native introspection, not serving these usecases on BCH might result in competitors building these products and gaining theses users instead.

## Security considerations

*(This depends on some specific design choices that has not yet been finalized)*

## List of major stakeholders

There is at least three major stakeholder groups:

### Companies and Organizations

There are some companies and organizations that are currently building products that utilize introspection:

- [General Protocols](https://generalprotocols.com/ "General Protocols") made [AnyHedge](https://anyhedge.com/ "AnyHedge"), a volatility risk-trading contract and are looking to build more non-custodial services and can enable more usecases with better introspection.

- [Tobias Ruck](https://twitter.com/TobiasRuck/ "Tobias Ruck") created [be.cash](https://be.cash/ "be.cash"), a refillable, offline wallet in the form of a credit card.

- [Casues Cash](https://causes.cash/ "Casues Cash") built an modified Mecenas to support recurring payments in USD.

- [Mistcoin](https://mistcoin.org/ "Mistcoin") has produced minable SLP tokens.

- [Flipstarter](https://flipstarter.cash/ "Flipstarter") was researching funding contracts as a way to improve usability, which may be possible with better introspection.

### Indepedent Developers

There is a number of independent developers who have an interest in developing tools and services that relies on introspection:

- [bitjson](https://gist.github.com/bitjson "bitjson") created [CashChannels](https://blog.bitjson.com/cashchannels-recurring-payments-for-bitcoin-cash-3b274fbfa6e2 "CashChannels"), reccuring payments for Bitcoin Cash.

- [haryu703](https://github.com/haryu703 "haryu703") created [Hamingja](https://github.com/SLPVH/hamingja "Hamingja"), a loyalty points system using non-tradable SLP token and is working on an SLP swap/trading contract.

- [Licho](https://github.com/KarolTrzeszczkowski "Licho") have created a [Last Will](https://github.com/KarolTrzeszczkowski/Electron-Cash-Last-Will-Plugin "Last Will") contract to manage inheritence, the [Mecenas](https://github.com/KarolTrzeszczkowski/Mecenas-recurring-payment-EC-plugin "Mecenas") contract for recurring payments and also considering to implement traditional games, like [NIM](https://en.m.wikipedia.org/wiki/Nim "NIM").

- [p0oker](https://twitter.com/p0oker "p0oker") created an [SLP Vending contract](https://github.com/p0o/yield-farming-bch-smart-contract "SLP Vending contract") that mints tokens on-demand and is building a BCH staking contract that mints tokens over time and an SLP exchange contract to sell NFTs.

- [Tobias Ruck](https://twitter.com/TobiasRuck/ "Tobias Ruck") is looking into a non-custodial on-chain gambling product.

- [James Cramer](https://twitter.com/James_Cramer "James Cramer") is experimenting with [SLP Mint Guard](https://github.com/simpleledger/Electron-Cash-SLP/blob/cashscript-dev/lib/cashscript/slp_mint_guard.cash "SLP Mint Guard") to protect minting batons, [SLP Vault](https://github.com/simpleledger/Electron-Cash-SLP/blob/cashscript-dev/lib/cashscript/slp_vault.cash "SLP Vault") to help reclaim unclaimed tokens, making [tokens with minting schedules](https://github.com/simpleledgerinc/slp-mint-contracts "tokens with minting schedules") and [SLP Dollars](https://github.com/simpleledgerinc/cashscript/blob/master/examples/slp_dollar.cash "SLP Dollars") that are tokens freezable by the issuer.

### Investors

There are people and organizations that have invested into the BCH token and ecosystem on the premise that it is expected to become peer to peer electronic cash for the world. Miners are a clear such group who have invested into the ecosystem. These stakeholders expects the token to get adopted in all the same places where money is traditionally used, which includes in financial services.

## Statements

...

### [General Protocols](https://generalprotocols.com/ "General Protocols")
> The CHIP requires further analysis and specification before it is possible to take a strong position. GP commits to supporting this CHIP with reasonable resources. GP predicts that the benefits to BCH value and network effect will greatly outweigh the costs. Regarding the multiple implementation options available, GP sees multiple good options and encourages the active CHIP participants to choose one with reasonable logic and zero polarization.

### [p0oker](https://twitter.com/p0oker "p0oker")
> I believe removing the current work arounds with the addition of native introspection OP Codes can improve the reliability of the smart contracts on Bitcoin Cash. Covenants are important and useful in many business use cases and it’s important to make them bulletproof!

### [Licho](https://github.com/KarolTrzeszczkowski "Licho")
> The covenant technology have proven to be a vibrant area of BCH development. It allows us to replicate more and more of traditional banking services in a non-custodial way. The native introspection seems to be the key ingredient of the future of permissionless money.

### Tom Zander — founder [Flowee](https://flowee.org/ "Flowee")
> This CHIP is super valuable and I fully support the direction this is going in and the ideas behind this chip. I will continue to monitor the progress with enthusiasm.

## Copyright Notice

Copyright (c) 2021 GeneralProtocols / Research

Permission is granted to copy, distribute and/or modify this document under the terms of the [MIT license](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/LICENSE).
