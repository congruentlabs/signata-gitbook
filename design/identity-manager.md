# Identity Manager

{% hint style="warning" %}
Note that the examples in this documentation are provided in JavaScript and are derived from the sata-contracts-v1 repository unit tests. If you wish to see more comprehensive details on how to interact with Signata identities, then for now the unit tests are the best option.

Elliptic curve signatures in the examples below are created using `ethereumjs-util`. Any time you see `Util.ecsign(...`, it will have come from the following import:

```jsx
const Util = require('ethereumjs-util');
```

Cryptographic signing in other languages can vary considerably.
{% endhint %}

## Links

{% embed url="https://my.signata.net/" %}
Production Website
{% endembed %}

{% embed url="https://github.com/congruentlabs/signata-web" %}
Website Source Code
{% endembed %}

## Overview

Enterprise Identity Managers are dedicated applications for the creation and management of all identities with an organisation. The Signata Identity Manager is the equivalent to this, but serving self-sovereign web3 identities. Users can create and manage all their own web3 identities within a single interface, and prepare them for use with the Signata Identity Provider.

## Identity Types

### Standard Identity

An "Standard" identity is created with 3 keys, an **Identity** key, a **Delegate** key, and a **Security** key.

#### About Key Lifecycles

Each time a private key is used for any publicly-available information (such as digitally signed records), the risk of key material compromise will increase through known-plaintext attacks. Most Public Key Infrastructure systems enforce key life limits (typically 1 to 2 years) on an active key, rolling over to new keys before the key could theoretically become compromised.

Signata will not enforce key lifespan lengths for identities, however the use of the 3-key identity lifecycle model will help to mitigate any risk of exposure through prolonged use of an identity, and also allow for simple rollover to a new set of keys if a user suspects any compromise.

The **recommended approach** to handling keys with Signata identities is to use 3 separate seeds for deriving the 3 keys needed to create each identity. Simply derive the 0th key from each seed for the first identity, then the 1st key from each seed for the second identity, and so on. In the unit test suite you will see we have derived all keys from the same seed - this was done purely to keep the unit tests simple but is not the recommended approach for production use.

#### Identity Key

The identity key is the core identity identifier. This key is intentionally "useless" in the context of lifecycle management with Signata. That is, it cannot be used to perform lifecycle event management of itself, just to ensure the key material is not used extraneously. It will bear all rights assigned to it, and be used for proving ownership, but all lifecycle events are instead handled by the Delegate Key.

#### Delegate Key

The delegate key is the lifecycle management key for a Signata identity. It is used to lock, unlock, destroy, and rollover identities after they are created.

It is not recommended that these keys be derived from the same seed as the identity key, otherwise a seed compromise will immediately compromise this key as well.

#### Security Key

The security key is a fallback key, specifically designed to provide recovery mechanisms in the event of a compromise. In ideal circumstances this key should never be needed to be used, it will only be used for important lifecycle events.

It is not recommended that these keys be derived from the same seed as the identity key, otherwise a seed compromise will immediately compromise this key as well.

#### Domain Separator

The domain separator is used to constrain signatures generated for Signata to just the Signata dApp and resist phishing attacks. The EIP-712 standard explains this in detail:

{% embed url="https://eips.ethereum.org/EIPS/eip-712" %}

```jsx
const CHAINID = 1; // CHAINID is determined from the network the contract is deployed on, it's set as the ETH mainnet ID here for the example

const IDENTITY_CONTRACT_ADDRESS = '0x6B47e26A52a9B5B467b98142E382c081eA97B0fc'; // again, this is an example. This is just the ETH mainnet ID contract address.

// these consts are fixed for all networks:
const EIP712DOMAINTYPE_DIGEST = '0xd87cd6ef79d4e2b95e15ce8abf732db51ec771f1ca2edccf22a46c729ac56472';
const NAME_DIGEST = '0xfc8e166e81add347414f67a8064c94523802ae76625708af4cddc107b656844f';
const VERSION_DIGEST = '0xc89efdaa54c0f20c7adf612882df0950f5a951637e0307cdcb4c672f298b8bc6';
const SALT = '0x233cdb81615d25013bb0519fbe69c16ddc77f9fa6a9395bd2aecfdfc1c0896e3';

// deriving the DOMAIN_SEPARATOR for the contract. You can derive this at any time after the contract has been deployed.
const DOMAIN_SEPARATOR = web3.utils.sha3(
  EIP712DOMAINTYPE_DIGEST
    + NAME_DIGEST.slice(2)
    + VERSION_DIGEST.slice(2)
    + CHAINID.toString('16').padStart(64, '0')
    + IDENTITY_CONTRACT_ADDRESS.slice(2).padStart(64, '0')
    + SALT.slice(2),
  {encoding: 'hex'}
);
```

#### Creating a Standard Identity

An identity is registered within the `SignataIdentity` contract using the following process:

* A `SHA3 Hash` is generated of the truncation of the `TXTYPE_CREATE_DIGEST`, the `Delegate Address`, and the `Security Address`.&#x20;
* The `SHA3 Hash` is digitally signed by the `Identity Private Key`.
* The `SHA3 Hash`, the `Delegate Address`, and the `Security Address` are sent to `create()` with the sender being the `Identity Key` to register the identity.
* A `Create` event is emitted from the contract upon successful creation.

All three keys _must_ be distinct, and they cannot already be in use with another identity (that is, you cannot re-use any of the keys more than once).

`0x1901` represents EIP-191 signed data, with `01` being the version number.

```
const TXTYPE_CREATE_DIGEST = '0x469a26f6afcc5806677c064ceb4b952f409123d7e70ab1fd0a51e86205b9937b';
```

```jsx

const inputHash = web3.utils.sha3(
  TXTYPE_CREATE_DIGEST
    + d1.slice(2).padStart(64, '0')
    + s1.slice(2).padStart(64, '0'),
  {encoding: 'hex'}
);

const hashToSign = web3.utils.sha3(
  '0x19' + '01' + DOMAIN_SEPARATOR.slice(2) + inputHash.slice(2),
  {encoding: 'hex'}
);

const { r, s, v } = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from(i1Private.slice(2), 'hex')
);

const createReceipt = await idContract.create(v, r, s, d1, s1, { from: i1 });

await expectEvent(createReceipt, 'Create', {
  identity: i1,
  delegateKey: d1,
  securityKey: s1
});
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

#### Locking a Standard Identity

To prevent an identity being modified or used, such as in the event you believe it has been compromised, you can flag the identity as "locked". You will not be able to use the identity whilst it is locked, but the delegate can still perform a rollover to ensure any compromise can be mitigated.

Locking is just a call to the `lock()` function by the delegate, no additional signatures are needed.

```
await expectEvent(await idContract.lock(i1, { from: d1 }), 'Lock', { identity: i1 });

expect(await idContract.isLocked(i1, { from: i1 })).to.equal(true);
```

Unlocking an identity is more complicated and will be covered in its own section.

#### Unlocking a Standard Identity

To unlock an identity, the `delegate` and `security` keys are needed to digitally sign the authorization of unlocking.

This is more complicated than locking, as locking is designed to be able to able to be performed quickly to respond to a threat but unlocking to require a user to assess whether the original threat still remains.

```jsx
const TXTYPE_UNLOCK_DIGEST = '0xd814812ff462bae7ba452aadd08061fe1b4bda9916c0c4a84c25a78985670a7b';

const inputHash = web3.utils.sha3(TXTYPE_UNLOCK_DIGEST + "0x01".slice(2).padStart(64, '0'), {encoding: 'hex'});

const hashToSign = web3.utils.sha3(
  '0x19' + '01' + DOMAIN_SEPARATOR.slice(2) + inputHash.slice(2),
  {encoding: 'hex'}
);

const sig1 = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from(d2Private.slice(2), 'hex')
);

const delegateV = sig1.v;
const delegateR = sig1.r;
const delegateS = sig1.s;

const sig2 = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from(s2Private.slice(2), 'hex')
);c

const securityV = sig2.v;
const securityR = sig2.r;
const securityS = sig2.s;

const unlockReceipt = await idContract.unlock(
  i2,
  delegateV,
  delegateR,
  delegateS,
  securityV,
  securityR,
  securityS,
  { from: d2 }
);

await expectEvent(unlockReceipt, 'Unlock', {
  identity: i2
});
```

#### Destroying a Standard Identity

Once an identity is no longer required, it can be destroyed to prevent further use. Identity destruction cannot be undone, it is permanent. If you wish to disable an identity temporarily, just lock it instead.

Identity destruction requires both the `delegate` and the `security` keys to digitally sign the authorization of destruction.

Also bear in mind that this is a blockchain and all the history of the identity will still remain on the chain after destruction. Destruction in this case simply means to prevent the key from ever being used again in the Signata identity ecosystem, and identity providers must honour the mutation that has been made.

```jsx
const TXTYPE_DESTROY_DIGEST = '0x21459c8977584463672e32d031e5caf426140890a0f0d2172da41491b70ef9f5';

const inputHash = web3.utils.sha3(TXTYPE_DESTROY_DIGEST, {encoding: 'hex'});

const hashToSign = web3.utils.sha3(
  '0x19' + '01' + DOMAIN_SEPARATOR.slice(2) + inputHash.slice(2),
  {encoding: 'hex'}
);

const sig1 = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from(d1Private.slice(2), 'hex')
);

const delegateV = sig1.v;
const delegateR = sig1.r;
const delegateS = sig1.s;

const sig2 = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from(s1Private.slice(2), 'hex')
);

const securityV = sig2.v;
const securityR = sig2.r;
const securityS = sig2.s;

const destroyReceipt = await idContract.destroy(
  i1,
  delegateV,
  delegateR,
  delegateS,
  securityV,
  securityR,
  securityS,
  { from: i1 }
);

await expectEvent(destroyReceipt, 'Destroy', {
  identity: i1
});
```

#### Rolling Over a Standard Identity

Much like the destruction event, rollovers also need signatures from both the `delegate` and `security` keys. Rollover will transfer control of an identity address to new delegate and security keys, the identity itself will remain the same.

After this mutation has been made the old delegate and security keys can no longer be used.

```jsx
const TXTYPE_ROLLOVER_DIGEST = '0x3925a5eeb744076e798ef9df4a1d3e1d70bcca2f478f6df9e6f0496d7de53e1e';
const inputHash = web3.utils.sha3(
  TXTYPE_ROLLOVER_DIGEST
    + d3.slice(2).padStart(64, '0')
    + s3.slice(2).padStart(64, '0')
    + "0x00".slice(2).padStart(64, '0'), {encoding: 'hex'});

const hashToSign = web3.utils.sha3(
  '0x19' + '01' + DOMAIN_SEPARATOR.slice(2) + inputHash.slice(2),
  {encoding: 'hex'}
);

const sig1 = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from(d2Private.slice(2), 'hex')
);

const delegateV = sig1.v;
const delegateR = sig1.r;
const delegateS = sig1.s;

const sig2 = Util.ecsign(
  Buffer.from(hashToSign.slice(2), 'hex'),
  Buffer.from(s2Private.slice(2), 'hex')
);

const securityV = sig2.v;
const securityR = sig2.r;
const securityS = sig2.s;

const rolloverReceipt = await idContract.rollover(
  i2,
  delegateV,
  delegateR,
  delegateS,
  securityV,
  securityR,
  securityS,
  d3,
  s3,
  { from: d2 }
);

await expectEvent(rolloverReceipt, 'Rollover', {
  identity: i2,
  delegateKey: d3,
  securityKey: s3,
});
```

### Lite Identity

