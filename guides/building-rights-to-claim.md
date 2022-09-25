# Building Rights to Claim

{% hint style="info" %}
This guide is for developers and integrators.
{% endhint %}

Signata Rights are represented as ERC721 Non-Fungible Tokens (NFTs), but extended to include schema information within each NFT. Each NFT right is defined first as a schema NFT, and then rights are issued against that schema to identities.

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

## Building a KYC Rights Claiming Service

Building your own rights claiming service requires 3 core pieces. A webhook processor, a signing authority, and a claiming contract. The below sections cover the issuance of KYC proofs from Blockpass, but the overall process should be the same no matter what provider actually does the KYC validation.

### Webhook Processor

Blockpass emits events as people perform KYC. You need to define the URL that will receive the webhook events, and have an API that receives those events and writes them to some form of data storage. It is crucial to use HMAC signatures on the events to ensure only Blockpass can send in events, otherwise someone could spoof valid transaction data to your API.

In this example, all events are pushed into a supabase database table:

```javascript
const BLOCKPASS_SECRET = process.env.BLOCKPASS_SECRET;
const supabaseUrl = process.env.SUPABASE_URL;
const supabaseKey = process.env.SUPABASE_KEY;

const supabase = createClient(supabaseUrl, supabaseKey);

/**
 * Process webhooks generated by blockpass.
 * X-Hub-Signature is used to verify the authenticity of the request.
 */
app.post("/api/v1/blockpassWebhook", async (req, res) => {
  const data = req.body;

  if (!data) {
    return res.status(400).json({ error: "No Data" });
  }

  const requestSignature = req.get("X-Hub-Signature");
  const algo = "sha256";
  const hmac = createHmac(algo, BLOCKPASS_SECRET);
  hmac.update(JSON.stringify(data));
  const result = hmac.digest("hex");

  if (result !== requestSignature) {
    return res.status(403).json({ error: "Invalid Signature" });
  }

  const { error } = await supabase.from("blockpass_events").insert(data);
  if (error) {
    console.error(error);
    return res.status(500).json({ error: "Events Error" });
  }
  return res.status(200).json({ message: "Event Added" });
});
```

### Signing Authority

A signing authority is simply a wallet that creates ecdsa signatures if certain criteria are met. This is fundamentally the same mechanism that Public Key Infrastructure uses, just on a much more simplified scale.

In this example an address is passed in, the function queries supabase for event records related to it, and then generates a signature of a hash containing the user's identity and a random number (salt). The salt ensures that the signature can't be replayed by the identity after it claims an NFT.

The signingAuthority is just a bip39 wallet seed. Generate the seed somewhere securely, and make sure it's encrypted for passing in to the environment variables.

```javascript
const signingAuthority = process.env.SIGNING_KEY;
const TXTYPE_CLAIM_DIGEST = process.env.TXTYPE_CLAIM_DIGEST;
const DOMAIN_SEPARATOR = process.env.DOMAIN_SEPARATOR;
const supabaseUrl = process.env.SUPABASE_URL;
const supabaseKey = process.env.SUPABASE_KEY;

const supabase = createClient(supabaseUrl, supabaseKey);
/**
 * Request a KYC claim signature. Verifies that the user has completed KYC with Blockpass.
 * Later this will be extended to support other KYC providers.
 */
app.get("/api/v1/requestKyc/:id", async (req, res) => {
  const { data, error } = await supabase
    .from("blockpass_events")
    .select("*")
    .eq("refId", req.params.id);

  if (error) {
    console.error(error);
    return res.status(500).json({ error: "Events Error" });
  }
  if (data.length === 0) {
    return res.status(204).json({ message: "No data found" });
  }
  // find an existing signature
  const { data: existingRecord, error: existingRecordError } = await supabase
    .from("kyc_claims")
    .select("sigR, sigS, sigV, salt")
    .eq("identity", req.params.id);

  if (existingRecordError) {
    console.error(existingRecordError);
    return res.status(500).json({ error: "Existing Record Error" });
  }

  if (existingRecord.length > 0) {
    // return the existing signature
    return res.status(200).json({
      sigR: existingRecord[0].sigR,
      sigS: existingRecord[0].sigS,
      sigV: existingRecord[0].sigV,
      salt: existingRecord[0].salt,
    });
  }

  // salt doesn't need to be ultra random. It's more about restricting the reuse of claims.
  const salt = crypto.randomBytes(32).toString("hex");
  const inputHash = ethers.utils.keccak256(
    `${TXTYPE_CLAIM_DIGEST}${req.params.id
      .slice(2)
      .padStart(64, "0")}${salt.padStart(64, "0")}`
  );
  const hashToSign = ethers.utils.keccak256(
    `0x1901${DOMAIN_SEPARATOR.slice(2)}${inputHash.slice(2)}`
  );
  const signature = new ethers.utils.SigningKey(signingAuthority).signDigest(
    hashToSign
  );
  
  const { error: insertError } = await supabase.from("kyc_claims").insert({
    identity: req.params.id,
    sigR: signature.r,
    sigS: signature.s,
    sigV: signature.v,
    salt,
  });

  if (insertError) {
    console.error(insertError);
    return res.status(500).json({ error: "Insert Error" });
  }

  return res.status(200).json({
    sigR: signature.r,
    sigS: signature.s,
    sigV: signature.v,
    salt,
  });
});
```

### Claiming Contract

With the other 2 components, their ultimate use is the smart contract that mints NFTs for identities. The example below first takes a fee for issuing the right, it makes sure the right (from the random salt generated) has not already been claimed, and then it recovers the address of the signingAuthority created the signature.

If the address of the signature is the same as the signing authority, then we have cryptographically proven that the authority issued the claim and the minting can proceed.

```solidity
function claimRight(
    address delegate,
    uint8 sigV,
    bytes32 sigR,
    bytes32 sigS,
    bytes32 salt
)
    external
    nonReentrant
{
    // take the fee
    if (feeAmount > 0 && !collectNative) {
        paymentToken.transferFrom(msg.sender, address(this), feeAmount);
        emit FeesTaken(feeAmount);
    }
    if (feeAmount > 0 && collectNative) {
        (bool success, ) = payable(address(this)).call{ value: feeAmount }(""); 
        require(success, "ClaimRight: Payment not received.");
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
