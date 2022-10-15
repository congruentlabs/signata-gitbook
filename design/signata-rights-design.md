# Signata Rights Design

## Overview

Signata Rights are Non-Fungible Tokens (NFTs) created through an extension to the ERC721 token standard called ERC721Schema. Schema NFTs are minted to represent a collection of rights, and all rights are issued linked to schemas.

Access control is rarely defined on for an individual, it's defined as a role or group. With Signata rights, roles or groups are minted as schemas and membership is minted with a relationship to those schemas.

ERC721Schemas are not controlled or managed by Signata, they can be defined by any wallet wishing to create a collection of rights. It's not recommended to issue rights from a personal wallet but instead to issue the rights through an intermediary smart contract. Smart contracts act as the authorities for rights, enforcing rules on issuance of the rights (such as payment, exclusive access, quantity, and time limitations).

Rights normally can only be issued to addresses registered as Signata Identities. Smart contracts do not need to be registered as identities, and rights can also be issued as "unbound" rights. Unbound rights are exactly the same as normal rights, but lose some useful abilities for migration with on-chain identities.

There are 2 optional modifiers for rights: **Transferrable**, and **Revocable**. Transferrable rights can be moved between Signata Identities (or smart contracts), and Revocable rights can be revoked by the minter if it needs to remove access for an identity.

There are no time constraints (like expiration dates) on identities. If rights need to expire after a timespan it is best to track that information within the issuing contract for each right so it can be read by access control systems, or use the revocable modifier and revoke identities on a periodic schedule.

## Soulbound Rights



## Issuing Rights



## 'Verifying Right Holders





