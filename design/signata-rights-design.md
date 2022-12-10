# Signata Rights Design

## Overview

1. Signata Rights can be defined by any wallet that wants to create a collection of rights.
2. It is recommended that rights be issued through an intermediary smart contract.
3. Smart contracts enforce rules on the issuance of rights, such as payment requirements and time restrictions.
4. Rights are typically only issued to registered Signata Identities, but can also be issued as "unbound" rights.
5. Signata Rights have two optional modifiers: Transferable and Revocable. These give minters and users greater control over their associated rights.

## About Signata Rights

Signata Rights are unique, non-fungible tokens (NFTs) that are created using the ERC721Schema extension to the ERC721 token standard. These tokens are minted to represent a collection of rights and are always issued in relation to a specific schema. ERC721Schema is a way of representing complex rights on EVM blockchains.

Access control is rarely defined for just one person; it is typically defined for a role or group. Signata rights allow you to mint roles or groups as schemas and mint membership using relationships to these schemas. This makes it possible to manage access control on the blockchain in a flexible and scalable way.

Signata identities do not have expiration dates. If rights need to expire after a certain time period, it is best to track this information within the issuing contract for each right so that it can be read by access control systems. Alternatively, you can use the revocable modifier to periodically revoke identities. This allows you to specify a time period for which an identity is valid, after which it can no longer be used.

## Issuing Rights

Signata Rights are not controlled or managed by Signata; they can be defined by any wallet that wants to create a collection of rights. It is recommended that rights be issued through an intermediary smart contract rather than from a personal wallet. Smart contracts serve as the authority for rights, enforcing rules on their issuance, such as payment requirements, exclusivity, quantity limits, and time restrictions.

Rights are typically only issued to addresses that are registered as Signata Identities. However, smart contracts do not need to be registered as identities, and rights can also be issued as "unbound" rights. These unbound rights have the same properties as normal rights, but they lose some useful abilities for migration with on-chain identities.

Signata Rights have two optional modifiers: Transferable and Revocable. Rights with the Transferable modifier can be transferred between Signata Identities or smart contracts, while rights with the Revocable modifier can be revoked by the minter if access needs to be removed for a specific identity. These modifiers give minters and users greater control over the rights associated with their Signata Tokens.

## Verifying Right Holders

Signata rights are minted and can be viewed on the SignataRights contract to determine if a particular identity holds them. Instead of having to track each individual right that has been issued, integrations simply need to verify if an identity holds a token with a specific schema identifier. This provides strong assurance that the identity holds the correct rights, as only one address or contract can mint tokens for a given schema.



