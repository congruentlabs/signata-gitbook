# Rights Management

Signata Rights are represented as ERC721 Non-Fungible Tokens (NFTs). Each NFT right is defined first as a schema NFT, and then rights are issued against that schema to identities.

As each schema is unique to a specific service or purpose this document will not dictate how integrators should specifically mint rights for users, but instead this document will illustrate a reference implementation. If you aren't sure how to build out your own rights management solution, use the Links page to reach out to the development community:

{% content-ref url="../extras/links.md" %}
[links.md](../extras/links.md)
{% endcontent-ref %}

## Claiming Rights

Whilst anyone can create their own schema at any time, the expected approach will be to define a schema and mint NFTs for it using a smart contract. The contract can then capture payment or enforce rules on who can claim the rights.

`ClaimRight.sol` is a smart contract that illustrates how to use a contract for identities to purchase rights.

{% embed url="https://github.com/congruentlabs/sata-contracts-v1/blob/main/contracts/ClaimRight.sol" %}

There are a few considerations for the creation of a right's claiming contract:

* Defining a schema requires a unique `schemaURI` (that is, no two schemas can have the same URI set). Define this as a URI that you control, for example: `https://my.signata.net/foo-right.json`.
* The contract must implement the interface `IERC721Receiver`. If you don't intend on implementing `onERC721Received()` then just define `_ERC721_RECEIVED` as a constant return value for the implementation.

```solidity
bytes4 private constant _ERC721_RECEIVED = 0x150b7a02;

function onERC721Received(
    address operator,
    address from,
    uint256 tokenId,
    bytes calldata data
)
    external
    pure
    returns (bytes4)
{
    return _ERC721_RECEIVED;
}
```

* The contract cannot mint the NFT schema during deployment. Create a second function to mint the schema that the contract will be using, and call that immediately after contract deployment.

```solidity
function mintSchema(
    string memory _schemaURI
) external onlyOwner {
    schemaId = signataRight.mintSchema(address(this), false, true, _schemaURI);
}
```

* Decide if you want the right to be transferrable or revocable.
* Decide if you want to just mint rights on receipt of payment, or if you want to add additional rule enforcement before minting.
* Set the `schemaId` visibility to `public` so it's consumable by dApps.

`ClaimRight.sol` shows both the capture of payment for minting rights, and the enforcement of a signature issued by a _signingAuthority_. If you add additional requirements you may need to add additional functions for adjusting the parameters for those requirements.

```solidity
function claimRight(
    address delegate,
    uint8 sigV,
    bytes32 sigR,
    bytes32 sigS,
    bytes32 salt
)
    external
{
    // take the fee
    if (feeAmount > 0) {
        signataToken.transferFrom(msg.sender, address(this), feeAmount);
        emit FeesTaken(feeAmount);
    }

    // check if the right is already claimed
    require(!cancelledClaim[delegate], "ClaimRight: Claim cancelled");
    require(claimedRight[delegate] != salt, "ClaimRight: Already claimed");

    claimedRight[delegate] = salt;

    // validate the signature
    bytes32 digest = keccak256(
        abi.encodePacked(
            "\x19\x01",
            domainSeparator,
            keccak256(
                abi.encode(
                    TXTYPE_CLAIM_DIGEST,
                    delegate,
                    salt
                )
            )
        )
    );

    address signerAddress = ecrecover(digest, sigV, sigR, sigS);
    require(signerAddress == signingAuthority, "ClaimRight: Invalid signature");

    // assign the right to the identity
    signataRight.mintRight(schemaId, delegate, false);

    emit RightClaimed(delegate);
}
```

## Inspecting Rights

The simplest way to tell if an identity holds a right is to inspect the schema id from the smart contract that issued it. Using the schema id, you can then call `holdsTokenOfSchema` on the Signata Rights contract, which returns a `bool` if the identity delegate has been issued the NFT or not.

<pre class="language-jsx"><code class="lang-jsx"><strong>// this example is a hooks based React implementation using
</strong><strong>// usedapp.io.
</strong><strong>
</strong><strong>export const useGetSingleValue = (method, args, contractAddress, contract) => {
</strong>  const { value, error } = useCall(
    contractAddress &#x26;&#x26; {
      contract,
      method,
      args,
    },
  ) ?? {};
  if (error) {
    console.error(error.message);
    return {};
  }
  return value?.[0];
};

// read the schemaId from a KYC claim contract
const schemaId = useGetSingleValue(
  'schemaId',
  [],
  kycClaimContractAddress,
  kycClaimContract
);

// check if the delegate holds a token in that schema
const hasBlockpassKycToken = useGetSingleValue(
  'holdsTokenOfSchema',
  [id.delegateAddress || '', schemaId],
  getRightsContractAddress(chainId),
  getRightsContract(chainId),
);</code></pre>
