# Signata DAO

Guides on using the DAO can be found here:

{% content-ref url="../guides/dao.md" %}
[dao.md](../guides/dao.md)
{% endcontent-ref %}

## Overview

The Signata Decentralized Autonomous Organization (DAO) is controlled by the Signata DAO (dSATA) token. This token is entirely separate from the SATA token as the SATA token was launched without the necessary components to be used with an on-chain DAO (namely snapshotting token balances and delegation). In addition, adding the ability to vote with SATA would have added additional gas costs for every use of the token, impacting its utility.

The Signata DAO is a derivative of OpenZeppelin's standardization of Compound's Governor contracts:

{% embed url="https://docs.openzeppelin.com/contracts/4.x/api/governance" %}

Tally.xyz is used as a DAO interface for the community as it supports OpenZeppelin implementations. Tally is just a website that lets you interact with the DAO contract, it's not hosting the DAO itself.

{% embed url="https://www.tally.xyz/governance/eip155:1:0x3D3255D21654B9a8325DfE6353ac6B37352Eb80B" %}

Websites like snapshot.org could have been used for handling the DAO for Signata instead of using the dSATA token, however there are a few issues with using snapshot:

* The SATA token is deployed on multiple chains. As more chains are added and supply burned on the Ethereum chain, the power of individual votes changes. People can bridge tokens to increase or decrease voting power and potentially manipulate votes.
* SATA tokens on chains other than Ethereum can never participate in votes.
* Snapshot DAOs are records of signatures only and cannot execute changes on the chain. Someone separate always needs to step in and execute the changes for the community if a vote passes.

Snapshot has been used for a "meta" vote for the DAO - a vote that did not require modification of any on-chain information. The community may wish to use that again in the future for any votes of a similar nature.

Proposal management is handled on GitHub as it provides an open and free platform for the creation and discussion of voting topics for the DAO. A number of proposal managers have been voted on by the community to moderate and protect the DAO from any malicious attacks.

{% embed url="https://github.com/congruentlabs/signata-dao" %}

## Tokenomics

The original launch of SATA had various allocations of SATA tokens. With the move to the Signata DAO, these allocations were abandoned in favor of handing over complete control of the tokens to the DAO, guaranteeing the protocol remains decentralized.

There is no team or investor allocation of tokens. There was no presale. All circulating supply has entered circulation through the initial airdrops, staking contracts, or exchange liquidity.

![SATA Token Supply Allocations (as of 2022-08-08)](<../.gitbook/assets/SATA Tokenomics.png>)

Some tokens are still held in timelocked vaults on the Ethereum chain. As this supply unlocks over time it will be transferred across to the DAO.

The launch supply was fixed at 100,000,000 SATA tokens. As additional networks have had SATA tokens deployed to them the amount of supply minted on them has been burned (sent to 0xdead) on Ethereum to preserve the 100M total supply. The DAO can vote to change the total supply if they wish, however the Ethereum, Avalanche C-Chain, Fantom, and Metis tokens do not support minting so additional supply would need to be minted on BSC or an alternative chain.

The BSC SATA token is an upgradeable token contract, purely to allow for enabling minting/burning roles as required by the completion of a DAO vote.

The dSATA launch supply was fixed at 50,000,000 dSATA tokens. This was to have half match the approximate 25,000,000 SATA in circulation at the time, and 25,000,000 being made available for SATA holders to exchange SATA for dSATA (exchange utility for voting rights) during a temporary migration window.

dSATA has a 1% swap tax on Uniswap swaps. This is intended to fund liquidity growth for the SATA/wETH pair on Uniswap. The DAO may vote to modify this tax percentage or purpose.

![dSATA Token Supply Allocations (as of 2022-08-08)](<../.gitbook/assets/SATA Tokenomics (1).png>)

## NFT-Based Governance

Earlier in the design phase for Signata, plans were put in place to modify voting rights based on NFT rights issued to particular users. This capability is dependent on constructing the means to issue these rights to their respective owners, the ability for them to be claimed by their owners, and for a governance contract to be built to recognise these rights.

The original design for this was for using the SATA token itself to hold these rights and vote. However, because the DAO is instead controlled by dSATA those plans no longer make sense as the tokenomics are different for dSATA. It is still intended to develop this capability, but the parameters for how the governor contract will recognize these rights will be subject to a DAO vote for acceptance.

