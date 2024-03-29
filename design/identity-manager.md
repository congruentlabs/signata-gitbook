# Identity Manager Design

{% hint style="warning" %}
Note that the examples in this documentation are provided in JavaScript and are derived from the signata-web source code. The examples are one way to achieve the desired results for on-chain interactions, but are not the only way to do so.
{% endhint %}

## Links

{% embed url="https://my.signata.net/" %}
Production Website
{% endembed %}

{% embed url="https://github.com/congruentlabs/signata-web" %}
Website Source Code
{% endembed %}

## Overview

The Signata Identity Manager is a dedicated application for the creation and management of web3 identities within an organization. It serves as the equivalent of an enterprise identity manager, but for self-sovereign web3 identities. With the Signata Identity Manager, users can create and manage all of their web3 identities within a single interface and prepare them for use with Signata Identity Providers. Signata uses blockchain technology as decentralized identity registers, ensuring that every Signata identity can be consumed within smart contracts and that users retain control over their identity state.

<figure><img src="../.gitbook/assets/1.png" alt=""><figcaption></figcaption></figure>

## Identity Definition

An "Standard" identity is created with 3 keys, an **Identity** key, a **Delegate** key, and a **Security** key.

### About Key Lifecycles

Each time a private key is used for any publicly-available information (such as digitally signed records), the risk of key material compromise will increase through known-plaintext attacks. Most Public Key Infrastructure systems enforce key life limits (typically 1 to 2 years) on an active key, rolling over to new keys before the key could theoretically become compromised.

Signata will not enforce key lifespan lengths for identities, however the use of the 3-key identity lifecycle model will help to mitigate any risk of exposure through prolonged use of an identity, and also allow for simple rollover to a new set of keys if a user suspects any compromise.

The **recommended approach** to handling keys with Signata identities is to use 3 separate BIP-39 seeds for deriving the 3 keys needed to create each identity.

### Identity Key

The identity key is the core identity identifier. This key is intentionally "useless" in the context of lifecycle management with Signata. That is, it cannot be used to perform lifecycle event management of itself, just to ensure the key material is not used extraneously. It will bear all rights assigned to it, and be used for proving ownership, but all lifecycle events are instead handled by the Delegate Key.

### Delegate Key

The delegate key is the lifecycle management key for a Signata identity. It is used to lock, unlock, destroy, and rollover identities after they are created.

It is not recommended that these keys be derived from the same seed as the identity key, otherwise a seed compromise will immediately compromise this key as well.

### Security Key

The security key is a fallback key, specifically designed to provide recovery mechanisms in the event of a compromise. In ideal circumstances this key should never be needed to be used, it will only be used for important lifecycle events.

It is not recommended that these keys be derived from the same seed as the identity key, otherwise a seed compromise will immediately compromise this key as well.

## Identity Storage

Signata stores no identity data on its servers. All identity records you create are AES256 password-encrypted and stored on IPFS. The actual record stored on IPFS is the entire array object of user identities converted to a JSON string and encrypted.

An IPFS Name key is generated for new user accounts and stored on Signata servers. This Name key is used to provide a consistent link to the same IPFS record, and replace the link with updated records as users make changes.

Signata servers retain the IPFS file name, as well as the account wallet address that created it. This way the Signata Identity Manager doesn't need to retain that information to recover user data on load.

All modifications to identity data stored in IPFS require a signature from the wallet that is pinned to the account. This ensures no unauthorised user can overwrite stored user data. Whilst overwritten data would not be lost, it would be treated as a denial of service attack against users.

## Identity Lifecycle Management

{% hint style="info" %}
The standard identity contract requires unique counters for events such as locking, unlocking, and rolling over. These counters ensure that signed mutations cannot be replayed after use.

The create and destroy functions can only be executed once for each identity, and as such do not require counters to resist replay attacks.
{% endhint %}

### Domain Separator

The domain separator is used to constrain signatures generated for Signata to just the Signata dApp and resist phishing attacks. The EIP-712 standard explains this in detail:

{% embed url="https://eips.ethereum.org/EIPS/eip-712" %}

Deriving the Domain Separator should be done for each chain to make sure you're capturing the correct values that are specified in the contract, as well as the specific chainId you're connected to.

```jsx
const domainSeparator = ethers.utils
  .keccak256(
    EIP712DOMAINTYPE_DIGEST
      + NAME_DIGEST.slice(2)
      + VERSION_DIGEST.slice(2)
      + chainId.toString(16).padStart(64, '0')
      + idContract.address.slice(2).padStart(64, '0')
      + SALT.slice(2),
  )
  .toString('hex');
```

### Standard Identity::Create

An identity is registered within the `SignataIdentity` contract using the following process:

* A `SHA3 Hash` is generated of the truncation of the `TXTYPE_CREATE_DIGEST`, the `Delegate Address`, and the `Security Address`.&#x20;
* The `SHA3 Hash` is digitally signed by the `Identity Private Key`.
* The `SHA3 Hash`, the `Identity Address`, the `Delegate Address`, and the `Security Address` are sent to `create()`. The sender can be any address.
* A `Create` event is emitted from the contract upon successful creation.

All three keys _must_ be distinct, and they cannot already be in use with another identity (that is, you cannot re-use any of the keys more than once).

The identity address must be provided with the signature to validate that the signature is actually signed by the same address.

`0x1901` represents EIP-191 signed data, with `01` being the version number.

```
const TXTYPE_CREATE_DIGEST = '0x469a26f6afcc5806677c064ceb4b952f409123d7e70ab1fd0a51e86205b9937b';
```

```jsx
const inputHash = ethers.utils.keccak256(
  `${TXTYPE_CREATE_DIGEST}${delegateWallet.address
    .slice(2)
    .padStart(64, '0')}${securityWallet.address
    .slice(2)
    .padStart(64, '0')}`,
);
const hashToSign = ethers.utils.keccak256(
  `0x1901${DOMAIN_SEPARATOR.slice(2)}${inputHash.slice(2)}`,
);
const { r, s, v } = new ethers.utils.SigningKey(
  identityWallet.privateKey,
).signDigest(hashToSign);

createSend(
  v,
  r,
  s,
  identityWallet.address,
  delegateWallet.address,
  securityWallet.address,
);
```

Alternatively, you can generate the signature components required in JS and then use Remix or Etherscan to make the chain call:

```javascript
const web3 = require('web3');
const Util = require('ethereumjs-util');

const CHAINID = 4; // replace this depending on which chain you're using. 4 is Rinkeby
var IDENTITY_CONTRACT_ADDRESS = '0b24e28a4b7fed6d59d3bd06af586f02fddfa6385'; // also replace this for the identity contract on the chain you're using.
const EIP712DOMAINTYPE_DIGEST = '0xd87cd6ef79d4e2b95e15ce8abf732db51ec771f1ca2edccf22a46c729ac56472';
const NAME_DIGEST = '0xfc8e166e81add347414f67a8064c94523802ae76625708af4cddc107b656844f';
const VERSION_DIGEST = '0xc89efdaa54c0f20c7adf612882df0950f5a951637e0307cdcb4c672f298b8bc6';
const SALT = '0x233cdb81615d25013bb0519fbe69c16ddc77f9fa6a9395bd2aecfdfc1c0896e3';
const TXTYPE_CREATE_DIGEST = '0x469a26f6afcc5806677c064ceb4b952f409123d7e70ab1fd0a51e86205b9937b';   
const TXTYPE_ROLLOVER_DIGEST = '0x3925a5eeb744076e798ef9df4a1d3e1d70bcca2f478f6df9e6f0496d7de53e1e';
const TXTYPE_UNLOCK_DIGEST = '0xd814812ff462bae7ba452aadd08061fe1b4bda9916c0c4a84c25a78985670a7b';
const TXTYPE_DESTROY_DIGEST = '0x21459c8977584463672e32d031e5caf426140890a0f0d2172da41491b70ef9f5';

const DOMAIN_SEPARATOR = web3.utils.sha3(
  EIP712DOMAINTYPE_DIGEST 
    + NAME_DIGEST.slice(2) 
    + VERSION_DIGEST.slice(2) 
    + CHAINID.toString('16').padStart(64, '0') 
    + IDENTITY_CONTRACT_ADDRESS.slice(2).padStart(64, '0') 
    + SALT.slice(2), 
  {encoding: 'hex'}
);

const inputHash = web3.utils.sha3(
  TXTYPE_CREATE_DIGEST
    + '0xce95dade44e7307baa616c77ef446915633dd9ab'.slice(2).padStart(64, '0') // replace the address with your delegate key
    + '0xc34504f0195f00914a1a3b5adf142b015f174125'.slice(2).padStart(64, '0'), // replace the address with your security key
  {encoding: 'hex'}
);

const hashToSign = web3.utils.sha3(
  '0x19' + '01' + DOMAIN_SEPARATOR.slice(2) + inputHash.slice(2), // 0x1901 means 
  {encoding: 'hex'}
);

const { r, s, v } = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from('0x0'.slice(2), 'hex') // replace 0x0 with the bytes of your identity private key
);

console.log(r.toString('hex'));
console.log(s.toString('hex'));
console.log(v);
```

![](<../.gitbook/assets/image (22).png>)

### Standard Identity::Lock

To prevent an identity being modified or used, such as in the event you believe it has been compromised, you can flag the identity as "locked". You will not be able to use the identity whilst it is locked, but the delegate can still perform a rollover to ensure any compromise can be mitigated.

* A `SHA3 Hash` is generated of the truncation of the `TXTYPE_LOCK_DIGEST`, and the `_identityLockCount`.
* The `SHA3 Hash` is digitally signed by the `Delegate Private Key`.
* The `SHA3 Hash` and the `Identity Address` are sent to `lock()`. The sender can be any address.
* A `Lock` event is emitted from the contract upon successful creation.

{% code lineNumbers="true" %}
```jsx
const inputHash = ethers.utils.keccak256(
  `${TXTYPE_LOCK_DIGEST}${identityLockCount
    .toHexString()
    .slice(2)
    .padStart(64, '0')}`,
);
const hashToSign = ethers.utils.keccak256(
  `0x1901${DOMAIN_SEPARATOR.slice(2)}${inputHash.slice(2)}`,
);
const { r, s, v } = new ethers.utils.SigningKey(
  delegateWallet.privateKey,
).signDigest(hashToSign);

lockSend(identityWallet.address, v, r, s);
```
{% endcode %}

### Standard Identity::Unlock

To unlock an identity, the `delegate` and `security` keys are needed to digitally sign the authorization of unlocking.

* A `SHA3 Hash` is generated of the truncation of the `TXTYPE_UNLOCK_DIGEST`, and the `_identityLockCount`.
* The `SHA3 Hash` is digitally signed by the `Security Private Key`.
* The `SHA3 Hash` and the `Identity Address` are sent to `unlock()`. The sender can be any address.
* An `Unlock` event is emitted from the contract upon successful creation.

{% code lineNumbers="true" %}
```jsx
const inputHash = ethers.utils.keccak256(
  `${TXTYPE_UNLOCK_DIGEST}${identityLockCount
    .toHexString()
    .slice(2)
    .padStart(64, '0')}`,
);
const hashToSign = ethers.utils.keccak256(
  `0x1901${DOMAIN_SEPARATOR.slice(2)}${inputHash.slice(2)}`,
);
const { r, s, v } = new ethers.utils.SigningKey(
  securityWallet.privateKey,
).signDigest(hashToSign);

unlockSend(identityWallet.address, v, r, s);
```
{% endcode %}

### Standard Identity::Rollover

Much like the destruction event, rollovers also need signatures from both the `delegate` and `security` keys. Rollover will transfer control of an identity address to new delegate and security keys, the identity itself will remain the same.

After this mutation has been made the old delegate and security keys can no longer be used.

```jsx
const inputHash = ethers.utils.keccak256(
  `${TXTYPE_ROLLOVER_DIGEST}${newDelegate.address
    .slice(2)
    .padStart(64, '0')}${newSecurity.address
    .slice(2)
    .padStart(64, '0')}${identityRolloverCount
    .toHexString()
    .slice(2)
    .padStart(64, '0')}`,
);
const hashToSign = ethers.utils.keccak256(
  `0x1901${DOMAIN_SEPARATOR.slice(2)}${inputHash.slice(2)}`,
);
const {
  r: delegateR,
  s: delegateS,
  v: delegateV,
} = new ethers.utils.SigningKey(delegateWallet.privateKey).signDigest(
  hashToSign,
);
const {
  r: securityR,
  s: securityS,
  v: securityV,
} = new ethers.utils.SigningKey(securityWallet.privateKey).signDigest(
  hashToSign,
);

rolloverSend(
  identityWallet.address,
  delegateV,
  delegateR,
  delegateS,
  securityV,
  securityR,
  securityS,
  newDelegate,
  newSecurity,
);
```

### Standard Identity::Destroy

Once an identity is no longer required, it can be destroyed to prevent further use. Identity destruction cannot be undone, it is permanent. If you wish to disable an identity temporarily, just lock it instead.

Identity destruction requires both the `delegate` and the `security` keys to digitally sign the authorization of destruction.

Also bear in mind that this is a blockchain and all the history of the identity will still remain on the chain after destruction. Destruction in this case simply means to prevent the key from ever being used again in the Signata identity ecosystem, and identity providers must honour the mutation that has been made.

```jsx
const inputHash = ethers.utils.keccak256(
  `${TXTYPE_DESTROY_DIGEST}`,
);
const hashToSign = ethers.utils.keccak256(
  `0x1901${DOMAIN_SEPARATOR.slice(2)}${inputHash.slice(2)}`,
);
const {
  r: delegateR,
  s: delegateS,
  v: delegateV,
} = new ethers.utils.SigningKey(delegateWallet.privateKey).signDigest(
  hashToSign,
);
const {
  r: securityR,
  s: securityS,
  v: securityV,
} = new ethers.utils.SigningKey(securityWallet.privateKey).signDigest(
  hashToSign,
);

destroySend(
  identityWallet.address,
  delegateV,
  delegateR,
  delegateS,
  securityV,
  securityR,
  securityS,
);
```
