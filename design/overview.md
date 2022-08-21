# Overview

Developers of online services spend countless hours of effort in the construction of systems to collect private information to identify users, and even more effort in maintenance of these systems to remain compliant with local and international laws and regulations. Taking payment for services only adds to this burden, as organizations must pay and rescind control to 3rd party service providers to deliver secure and fraud-resistant services for the collection and management of payment information.

The Signata Project is building the means to create the next generation of identities that are kept anonymous from service providers, as well as a framework for a decentralized on-and-off-chain solution for the identification, authorization, and lifecycle management for these identities. The project provides the ability for users to self-assert identities onto chains via smart contracts, as well as the ability for service providers to validate and maintain known anonymous identities via off-chain solutions.

## Introduction

The online identity management world is in a constant state of flux. Centralized identity providers (such as Google, Facebook, and Okta) are attempting to lead the charge with centralized authentication services for simplified management, and the era of the password is looking to quickly become obsolete. However, centralized identity management requires users to rescind all control of their identity and access to the service providers that manage their identities, instead of retaining the control over their individual authentication capabilities and identity assertions. These centralized providers typically fund themselves by building unprecedented tracking data on individuals, observing the use of identities within their services and outside through an ever-growing network of online tracking systems.

The Signata project has introduced both an ERC-20 token as well as a number of dApps and services for identity management. The token is used to interact with a platform of smart contract-based decentralized identity services - both as core internal capabilities for the product, but additionally as on-and-off-chain anonymity preserving systems that external applications can integrate and consume to build an identity ecosystem unbound by central authorities.

Existing blockchain identity management systems are often designed and built adjunct to the chain - Signata is instead delivering a platform that provides a better bridge from Web 2 Identity Management into Web 3, leveraging the blockchain as the core registry & state mutation solution. Signata provides:

* Anonymous but cryptographically trusted identification of individuals,
* Decentralized assertion of content, and
* Secured payment for services or interactions on the chain.

Service Providers are able to securely authenticate and authorize users with anonymous credentials by combining on-chain verification of data produced by the credential holder (assuring ownership of the credential), on-chain verification of authorizations, and off-chain verification of information held within the service itself (ensuring the credential receives appropriate authorization).

This capability will allow for service providers to authenticate users, collect payments, and provide access control to systems without knowing any identifiable information about the user - unless they want to collect that information themselves and the user consents to the collection of the information.

## Self-Asserted Identity Registries

Individuals can create an unlimited number of identities to serve their needs. Each identity is comprised of 3 separate keys - an identity anchor key, a delegation key, and a security key. The anchor key holds and proves ownership of any rights held by the individual. The delegation key serves as the controlling key for the identity. The security key serves as an emergency authorization for the identity should the delegate key become compromised.

The design intent for Signata identities is to limit the cryptographic use of identity key material broadcast to the public. The more that key material is used, the greater the risk of compromise through known-plaintext attacks, especially as identity keys are primarily used to sign public data.

Each chain that Signata contracts are deployed to will serve as an identity registry. After an individual creates the key material required for their identity, they can submit that material to the chain for registration. After registration, smart contracts and other dApps can interrogate the chain for identity state information of the individual.

## **Decentralized Identity Providers**

Identities registered on a chain provide some value to smart contract developers, but for authentication purposes provide little value. Interrogating an identity register does not prove ownership of an identity, and so identity providers must request individuals to assert proofs to complete authentication and authorization processes.

Standardized authentication frameworks such as OpenID Connect and OAuth have established a strong identity layer for the internet, consolidating identities to singular sources for simplified sign-on. Signata provides a "bridge" of these existing frameworks to apply them to web3 identities, intercepting authentication processes to validate proofs of key ownership and then issue trusted tokens for Service Providers to consume.

The power of Signata is that the authentication process for Service Providers remains almost entirely the same - users are redirected to the Signata Identity Provider, connect their web3 wallet, and return back to the Service Provider bearing a token asserting their identity information. The core difference is only their wallet address and on-chain rights are asserted, not personal information such as their email address or name.

Service Providers may choose to forego Single Sign-On with Signata and just connect directly to a user's wallet, however they must still validate that the user controls their keys of their wallet by virtue of requesting the wallet to cryptographically sign some piece of data. Service Providers can utilize Signata for a trusted 3rd party cryptographic validation, saving development effort.

## Decentralized Rights Management

Rights, at their core, are just assertions of state or attributes. In the context of identity, rights can be assertions of attributes like Nationality or Age, or be assertions of state like "has paid for service". A right is something issued by one party and assigned to another.

For smart contract systems, Non-Fungible Tokens can serve as the base structure of a right. With Signata, rights are defined as ERC721-compliant Non-Fungible Tokens. All rights originate from a register, and authorities (such as Service Providers) can define a right schema on the chain of which they distribute rights to Identities as needed.

For the collection of payment for rights, Service Providers can simply utilize exchange contracts that receive payment in an asset and mint a Signata right in response. As an individual receives these rights, they are automatically passed to Service Providers during authentication for the Service Provider to perform authorization validation on.

## Governance

The Signata ERC20 token is not the governance token for the protocol. As the token was originally created without implementing the ERC20Votes or ERC20Snapshot standards the token cannot be used with a Compound-based Decentralized Autonomous Organization (DAO). Without those essential extensions the governor contract would be at risk of receiving duplicate votes on proposals.

The addition of ERC20Votes and ERC20Snapshot to a token increases the network fees required during every transfer of the token. As the Signata token is a utility token minimizing the fees for its use is a superior outcome for adoption of the protocol, and through typical use would result in the voting power of holders constantly fluctuating. The Signata token is also deployed on multiple networks, and with a Compound-based DAO governor contract on Ethereum the voting rights of holders on other networks would not apply.

Governance for Signata is handled separately through a separate Signata DAO ERC20 token. The governance token is used exclusively for governing the protocol and should not be used transactionally for utility purposes (the only transactions being to transfer voting power). The utility token is used exclusively for using the systems within the protocol.
