# Examples

{% hint style="warning" %}
Any source code provided on this site is unaudited and provides no guarantee that it will be functional or safe to use. Ensure you test before deploying to mainnet or production use.
{% endhint %}

## Applying Signata Identities to Authentication Systems

### Claims-based Enforcement

TODO

### Applying Signata Identities to Smart Contracts

An identity by itself is of no value. Its value is derived from those that recognise it, and what systems it can be interconnected to.

This document details the application of Signata identities to blockchain-based systems. It does not prescribe a recommended or best-practice approach for integration, only a starting point that can be developed upon for real-world systems.

Integrating identity validation into smart contracts is extremely simple - connecting to an instance of the SignataIdentity and SignataRights contracts within a network will provide the basis of enforcing Signata-based identity enforcement into any contract function that you wish to use it in.

#### Simple Access Control

At a basic level, smart contracts can verify that an address has been registered with an identity contract.

For open identity contracts this provides next to no assurance as any address may register themselves as an identity and pass this check. But for an access-controlled identity contract, the provisioning of the identity can be restricted to specific accounts with a process of issuance having been performed by the contract owner.

```solidity
SignataIdentity private signataIdentity;

/* ... */

// get delegate - will throw if the identity does not exist
signataIdentity.getDelegate(msg.sender);

// or check if locked - also will throw if the identity doesn't exist
// and will also reject locked identities
require(
    !signataIdentity.isLocked(msg.sender),
    "The sender's identity is locked."
);
```

#### Rights Based Access Control

To enforce stronger access control in contracts, validating that an address holds a specific ERC721 right is possible by checking against those that have been

At a basic level, smart contracts can verify that an address has been registered with an identity contract.

For open identity contracts this provides next to no assurance as any address may register themselves as an identity and pass this check. But for an access-controlled identity contract, the provisioning of the identity can be restricted to specific accounts with a process of issuance having been performed by the contract owner.

```solidity
SignataRight private signataRight;
SignataIdentity private signataIdentity;
address private trustedMinter;

/* ... */

// check the minter of an asserted right
require (signataRight.minterOf(nftRightIndex) == trustedMinter, "Right is not from trusted minter");

// get the delegate address of the sender
address delegateAddress = signataIdentity.getDelegate(msg.sender);

// check the owner of the asserted right
require (signataRight.ownerOf(nftRightIndex) == delegateAddress, "Right is not owned by sender!");
```

With these three checks above, we've validated that the asserted right of the sender is actually owned by them via their identity delegate key, we've validated that the identity is actually currently active (getDelegate will fail if the identity is currently locked or destroyed), and we've validated that the minting of the right came from a trusted source.

#### Business Register

Most jurisdictions will manage a public registry of all operating businesses, tracking each with a unique identifier so they're identifiable separately to the individuals that own, operate, or work in those businesses.

TODO

#### Staking Pool

For staking pools, you can simply deny entry to the pool if the user does not hold a specific identity or right.

And for unstaking, adding the same checks can ensure that a mutated identity cannot claim the rewards if that is a desired outcome.

```solidity
SignataIdentity private signataIdentity;

/* ... */

function stakeTokens(uint256 _amount, uint256[] memory _tokenIds) public {
  require(
    getLastStakableBlock() > block.number,
    'this farm is expired and no more stakers can be added'
  );

  // try to get the delegate of the account. this will fail if the account isn't registered, or is in an unusable state
  signataIdentity.getDelegate(msg.sender);

  _updatePool();

  if (balanceOf(msg.sender) > 0) {
    _harvestTokens(msg.sender);
  }

  uint256 _finalAmountTransferred;
  if (pool.isStakedNft) {
    require(
      _tokenIds.length > 0,
      "you need to provide NFT token IDs you're staking"
    );
    for (uint256 _i = 0; _i < _tokenIds.length; _i++) {
      _stakedERC721.transferFrom(msg.sender, address(this), _tokenIds[_i]);
    }

    _finalAmountTransferred = _tokenIds.length;
  } else {
    uint256 _contractBalanceBefore = _stakedERC20.balanceOf(address(this));
    _stakedERC20.transferFrom(msg.sender, address(this), _amount);

    // in the event a token contract on transfer taxes, burns, etc. tokens
    // the contract might not get the entire amount that the user originally
    // transferred. Need to calculate from the previous contract balance
    // so we know how many were actually transferred.
    _finalAmountTransferred = _stakedERC20.balanceOf(address(this)).sub(
      _contractBalanceBefore
    );
  }

  if (totalSupply() == 0) {
    pool.creationBlock = block.number;
    pool.lastRewardBlock = block.number;
  }
  _mint(msg.sender, _finalAmountTransferred);
  StakerInfo storage _staker = stakers[msg.sender];
  _staker.amountStaked = _staker.amountStaked.add(_finalAmountTransferred);
  _staker.blockOriginallyStaked = block.number;
  _staker.timeOriginallyStaked = block.timestamp;
  _staker.blockLastHarvested = block.number;
  _staker.rewardDebt = _staker.amountStaked.mul(pool.accERC20PerShare).div(
    1e36
  );
  for (uint256 _i = 0; _i < _tokenIds.length; _i++) {
    _staker.nftTokenIds.push(_tokenIds[_i]);
  }
  _updNumStaked(_finalAmountTransferred, 'add');
  emit Deposit(msg.sender, _finalAmountTransferred);
}
/* ... */
```

#### Token Lock

Token locks can be restricted to only accept locks from validated identities, before the lock is created.

```solidity
SignataIdentity private signataIdentity;

/* ... */

function createLocker(
  address _tokenAddress,
  uint256 _amountOrTokenId,
  uint48 _end,
  uint8 _numberVests,
  address[] memory _withdrawableAddresses,
  bool _isNft
) external payable {
  require(
    _end > block.timestamp,
    'Locker end date must be after current time.'
  );

  // try to get the delegate of the account. this will fail if the account isn't registered, or is in an unusable state
  signataIdentity.getDelegate(msg.sender);

  _payForService(0);

  if (_isNft) {
    IERC721 _token = IERC721(_tokenAddress);
    _token.transferFrom(msg.sender, address(this), _amountOrTokenId);
  } else {
    IERC20 _token = IERC20(_tokenAddress);
    _token.transferFrom(msg.sender, address(this), _amountOrTokenId);
  }

  lockers.push(
    Locker({
      owner: msg.sender,
      isNft: _isNft,
      token: _tokenAddress,
      amountSupply: _isNft ? 1 : _amountOrTokenId,
      tokenId: _isNft ? _amountOrTokenId : 0,
      start: uint48(block.timestamp),
      end: _end,
      withdrawable: _withdrawableAddresses,
      amountWithdrawn: 0,
      numberVests: _isNft ? 1 : (_numberVests == 0 ? 1 : _numberVests)
    })
  );
  uint16 _newIdx = uint16(lockers.length - 1);
  lockersByOwner[msg.sender].push(_newIdx);
  lockersByToken[_tokenAddress].push(_newIdx);
  if (_withdrawableAddresses.length > 0) {
    for (uint16 _i = 0; _i < _withdrawableAddresses.length; _i++) {
      lockersByWithdrawable[_withdrawableAddresses[_i]].push(_newIdx);
    }
  }
  emit CreateLocker(msg.sender, _newIdx);
}
/* ... */
```

#### DAO Governor with Identity Validation

The simplest way to enforce identity usage within a DAO is to just validate the voter when they attempt to cast votes or delegate their voting rights.

```solidity
/* override votes on the governor */
castVote(uint256 proposalId, uint8 support)
castVoteWithReason(uint256 proposalId, uint8 support, string reason)
castVoteWithReasonAndParams(uint256 proposalId, uint8 support, string reason, bytes params)
castVoteBySig(uint256 proposalId, uint8 support, uint8 v, bytes32 r, bytes32 s)
castVoteWithReasonAndParamsBySig(uint256 proposalId, uint8 support, string reason, bytes params, uint8 v, bytes32 r, bytes32 s)

/* override delegation on the token contract */
delegate(address delegatee)
delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
```

#### Batch Airdrop

Note: this is not a common way to do airdrops, as batch ERC20 token transactions are expensive for the sender. If you do wish to distribute tokens in this manner, then you can skip any recipients that aren't registered identities.

```solidity
SignataIdentity private signataIdentity;

/* ... */
function bulkSendErc20Tokens(
  address _tokenAddress,
  Receiver[] memory _addressesAndAmounts
) external payable returns (bool) {
  _payForService(0);

  IERC20 _token = IERC20(_tokenAddress);
  for (uint256 _i = 0; _i < _addressesAndAmounts.length; _i++) {

    Receiver memory _user = _addressesAndAmounts[_i];

    try signataIdentity.getDelegate(_user.userAddress) {
      _token.transferFrom(msg.sender, _user.userAddress, _user.amountOrTokenId);
    } catch { } // ignoring the index if the identity was not found
  }
  return true;
}
/* ... */
```

#### Merkle Airdrop

Merkle-based drops are far more cost effective for distributing tokens to a large number of holders. If you wish to only distribute tokens to created identities, or identities that hold a specific NFT right (such as a KYC claim), then you can simply inject a check into the claim function to prevent invalid identities from claiming the airdrop.

For a merkle-based drop it would be advisable to only create the proofs for addresses that are actually valid in the first place, performing that computation off-chain, but as identities may mutate after the claim contract is deployed it can provide an extra level of on-chain assurance.

```solidity
/* ... */
function claim(uint256 index, address account, uint256 amount, bytes32[] calldata merkleProof) external override {
  require(claimsEnabled, "Airdrop::claim: Claims not enabled.");
  require(!isClaimed(index), "Airdrop::claim: Drop already claimed.");

  // try to get the delegate of the account. this will fail if the account isn't registered, or is in an unusable state
  signataIdentity.getDelegate(account);

  bytes32 node = keccak256(abi.encodePacked(index, account, amount));
  require(MerkleProof.verify(merkleProof, merkleRoot, node), "Airdrop::claim: Invalid proof.");

  _setClaimed(index);

  require(token.transfer(msg.sender, amount), "Airdrop::claim: Transfer failed.");
  emit Claimed(index, msg.sender, amount);
}
/* ... */
```

#### Dividend Rewards Distribution

Some DeFi projects use "dividend" based distribution of assets. Enforcing of holding an identity, or a specific NFT right, can be simply included in-line with functions that set the balance of the address. If it holds a Signata Identity, then it can receive a balance fo the dividend contract. Otherwise it is treated like an excluded contract

```solidity
/* ... */
function setBalance(address payable account, uint256 newBalance)
    external
    onlyOwner
{
    if (excludedFromRewards[account]) {
        return;
    }

    // if it's not a registered identity, then it will be treated like it's in the exclusion list
    try signataIdentity.getDelegate(account) {
      if (newBalance >= minTokenBalanceForRewards) {
          _setBalance(account, newBalance);
      } else {
          _setBalance(account, 0);
      }
    } catch { return; }
}
/* ... */
```
