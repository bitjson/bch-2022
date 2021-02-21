# NATIVE INTROSPECTION - Proposal for May 15th, 2022

## Summary

This proposal updates the Bitcoin Cash scripting language with additional opcodes to natively provide details about the current transaction, such as the output amount and recipients.

> OWNERS: [Jason Dreyzehner](https://gist.github.com/bitjson)?, [Jonathan Silverblood](https://gitlab.com/monsterbitar)
> 
> DISCUSSION: [Telegram](https://t.me/transactionintrospection), Bitcoin Cash Research (soon)
> 
> MILESTONES: **[Published](https://gitlab.com/GeneralProtocols/research/-/blob/master/CHIPs/May%202022,%20Native%20Introspection.md)**, Testnet (August), Specification (October), Accepted (November), Deployed (May 15th, 2022).

## Motivation and Benefits

In order for Bitcoin Cash to gain adoption as money, it needs to provide similar or better features than existing alternatives. One area where Bitcoin Cash can be uniquely positioned is as a cost-efficient programmable money. By improving the Bitcoin Cash script with additional opcodes to provide native introspection, we gain the following benefits:

- Reduces barriers to entry for new developers that want to build smart contracts using transaction introspection.

- Improves contract safety by eliminating implementation risks that comes with the complexity of the current re-verification trick used as a workaround.

- Allows for new usecases to be developed as native introspection can provide more information on a transaction than the current re-verification trick.

- Allows for larger and more complex smart contracts to be developed as native introspection takes up less scripting space compared with current re-verification trick.

- For transactions that uses introspection, the transaction size can be reduced which lowers the cost on the network in terms of bandwidth and long-term storage.

- For transactions that uses introspection, the network processing cost would be reduced since there is no longer any need to do an additional signature verification.

## Implementation Costs and Risks

- Native introspection will use up some of the currently available opcodes. Depending on implementation detail, this cost might be small (single opcode, templated data) or it might be significant (every transaction data can be accessed through a unique relevant opcode).

- Since this would be a consensus change, all node software that validate consensus would need to implement the feature.

- A limited number of libraries and developer tools that offer advanced scripting functionality would need to implement the feature.

  *Wallets, mining pools, exchanges and services does not have to be updated and will continue to function as normal.*

## Ongoing Costs and Risks

- Adding native introspection could increase complexity for any future technical changes to the Bitcoin Cash transaction format.

## Technical Description

*(This section still needs to be drafted out. It will mostly look like the previous version listed in the alternatives, where there's one or more opcodes and a table of datapoints one could push to the stack. It's unclear if it will be templated or multiple opcodes, concatenated or not and a few other technical details still needs to be worked out.)*

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

- Continued use of the inefficient re-verification trick makes the blockchain larger than it needs to be and could have a negative impact on initial block download times and storage requirements.

- Requirement to use a difficult technical workaround in order to achieve transaction introspection could result in loss of opportunity as development costs and barrier to entry remains high.

- Complexity of the technical workaround in order to achieve transaction introspection could result in otherwise successful businesses having technical problems with their implementation that results in loss of profits and damages reputation of both the company and Bitcoin Cash as a network.

- Some applications and usecases are not possible with the re-verification trick but depending on technical implementation details may be possible with native introspection, not serving these usecases on BCH might result in competitors building these products and gaining theses users instead.

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

### Speculators

Some people have invested into BCH on the premise that it is expected to become peer to peer electronic cash for the world and who expects the token to get adopted in all the same places where money is traditionally used, which includes in financial services.

## Statements

TBD.


*This document is licensed under CC0.*
