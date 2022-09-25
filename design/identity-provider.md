# Identity Provider Design

The Signata Identity Provider (IdP) is an extension to Red Hat's keycloak. This means it provides a full web2-like authentication experience using existing identity protocols (OAuth), but the addresses of the identity are used in place of email addresses.

This means service providers (web apps, phone apps) can integrate Signata in the exact same way they integrate IdP providers like Google, Facebook, Twitter, etc., but using Signata identities instead.

<figure><img src="../.gitbook/assets/2.png" alt=""><figcaption></figcaption></figure>

Each website/app simply needs to be registered as a client Service Provider (SP) with a running Signata-integrated keycloak instance. This document won't cover the specifics of integrating keycloak as there are already plenty of guides provided by keycloak for doing so.

{% embed url="https://www.keycloak.org/docs/latest/securing_apps/index.html" %}

The key difference in result from using Signata-integrated keycloak is that an attribute like _mail_ won't be returned as a claim, as Signata doesn't know your email address. Instead, both your _identity_ and _delegate_ addresses will be returned during authentication, as well as the identifiers of any NFT rights you own that have been configured to be read by keycloak.

After authentication it is up to the website or app to determine how it will consume that information. The website/app doesn't require a web3 network connection unless is needs to make changes on-chain, as it can trust the assertions provided from the Signata IdP. If the website/app _does_ connect to web3, then it has the choice of either using the Signata IdP or just reading the connected wallet information (albeit requiring more client-side logic to derive all identity information about rights for the identity).
