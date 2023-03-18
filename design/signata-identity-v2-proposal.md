# Signata Identity V2 Proposal

## Identity Register

The Signata Identity register is an essential piece of the identity lifecycle, especially in relation to Soubound NFTs, as it provides the overarching lifecycle management of an identity as a whole in addition to any rights/soubound NFTs that may be held by it.

### Identity Creation

The first iteration of the Signata Identity register provided a robust model for establishing an mutating an identity on-chain, but suffered from a few drawbacks:

1. The 3-key approach of "identity" (id\_key), "delegate" (delegate\_key), and "security" (security\_key) keys is too complex for non-technical users to understand.
2. The id\_key key being different from the delegate\_key key, but the delegate\_key key being (typically) the connected web3 wallet was a confusing model for users.
3. The need to generate and manage these three keys from a custodial perspective was difficult, as the users would need to use a dedicated dApp rather than being able to be integrated with other dApps.
4. Some mutation events required digital signatures to be created from the Delegate wallet, which would throw warnings in wallet software that it was a high risk signature to perform (due to a superseded implementation of eth\_sign vs. personal\_sign)
5. Delegation could only be against a single address, and not support delegation to services or other trusted individuals.

The proposal for Version 2 is to modify the identity register as follows:

* Consolidate id\_key and delegate\_key together to be represented as a single id\_key.
* Establish a delegation mapping against the id\_key record, and on creation assign id\_key as it's own delegate.
* Allow the id\_key to add or remove delegations to itself. The id\_key will be unable to remove itself as a delegation.
* Lock, Unlock, Destroy, and Migrate events can be performed by any delegated address.
* An address can be assigned as delegate for multiple id\_keys, allowing service providers to centrally manage identities on-chain.
* The security\_key will be removed to reduce complexity. The security\_key was originally serving the purpose of an alternative means of identity mutation overrides, but this functionality can be inherited through delegation to a separate address.

The creation of the id\_key will not be possible by a delegate - it must always originate from the transaction signer to ensure only the key holder consents to the identity creation. This does not exclude a service provider for creating an identity on-behalf of a user, but they will also be required to generate and retain the private key material (and provide that to the user) to retain full control of the identity.

### Identity Mutation

The identity mutation events defined in Signata V1 will still be retained. These are:

1. Lock
2. Unlock
3. Destroy
4. Migrate

Whilst these states do not remove the on-chain representation of the identity, they still provide tools for service providers to be able to provide alternative control flows for the active state of an identity.

For the id\_key, it will retain the ability to perform all mutation events. For delegate addresses, the rights of which mutation event they can perform will be defined by the id\_key (for example, they may be permitted to lock and unlock but not destory or migrate the identity).

## Soulbound Rights

Signata Rights were an early implementation of what later became known as Soulbound NFTs. This implementation was founded on enterprise identity lifecycle management concepts with the custody of the identity instead held by the creator (as opposed to a 3rd party).

Functionally, these rights still serve the majority of Soulbound NFT requirements, but require updates to meet the additional requirements to become "soulbound-complete" in their utility.

### Mutual consent



### Rights expiration

An additional attribute will be assigned to rights called expiration\_timestamp. This will provide the optional ability to define when a right is no longer active. This will be useful for rights such as "proof of KYC" where regulatory requirements require an individual to complete KYC processes periodically. The ability to update this attribute will be held by the schema administrator of the right.

### Rights attributes

Each Signata NFT bears a "schema" definition, which is the same definition held by every right issued underneath that schema.

The schema owner, upon issuing an NFT right, will be additionally able to define custom data against individual rights. This can be in the form of a URI to an externally hosted definition that provides additional metadata attached to the right, or data to be stored and consumable on-chain.

## Migration Strategy

For identities and rights issued against V1 contracts, migration contracts will be provided for converting them into V2 rights. These contracts will be provided for Signata-issued rights, but any 3rd party rights issuers will be required to deploy their own migration contracts if desired.



