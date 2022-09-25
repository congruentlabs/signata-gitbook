# Using the DAO

{% hint style="warning" %}
Some of the screenshots and content in this page are copied from another project's DAO documentation. These will be updated over time to be derived from the Signata DAO, but for now they still illustrate the exact same process.
{% endhint %}

## Proposal Discussion

All Signata DAO proposals are managed on GitHub. Create proposals here for discussion:

{% embed url="https://github.com/congruentlabs/signata-dao" %}

Discussion about proposals also happens on Discord:

{% content-ref url="../extras/links.md" %}
[links.md](../extras/links.md)
{% endcontent-ref %}

## Delegating Votes <a href="#delegating-votes" id="delegating-votes"></a>

To be able to vote in the Signata DAO as a dSATA token holder, you simply need to do one thing: Delegate your votes. It's crucial you do this before proposals are started, otherwise you will not be able to vote in them.

To do this, open the Signata DAO on Tally.xyz:​​

{% embed url="https://www.tally.xyz/governance/eip155:1:0x3D3255D21654B9a8325DfE6353ac6B37352Eb80B" %}

At the top of the page, click **Delegate vote**:

![](<../.gitbook/assets/image (3) (1) (2).png>)

Connect your MetaMask or WalletConnect wallet:

![](https://blog.aggregated.finance/content/images/2022/05/image-5.png)

Then you will be asked to either **Delegate to self**, or **Delegate to an address**. If you Delegate to yourself then **you** will need to participate in voting. If you delegate to another address, then you are transferring your voting power to that other address.

![](https://blog.aggregated.finance/content/images/2022/05/image-6.png)

You will then need to confirm on-chain your delegation.

![](https://blog.aggregated.finance/content/images/2022/05/image-12.png)

Once the transaction is confirmed that's it! You are ready to vote in the DAO. Your address will show up in Tally as a voter (it can sometimes take a few minutes to appear), and show you how much voting power you have.

![](<../.gitbook/assets/image (5) (2).png>)

If your dSATA balance changes, make sure you delegate your votes again so your new balance is tracked in the DAO.

## Casting Votes <a href="#casting-votes" id="casting-votes"></a>

{% hint style="warning" %}
TODO
{% endhint %}

## Creating Proposals on Tally

On the Tally DAO interface, click **Create new proposal**.

![](<../.gitbook/assets/image (20) (1).png>)

Connect your wallet and click **Continue**.

![](<../.gitbook/assets/image (1) (2).png>)

Fill out the description from the proposal information defined on the GitHub proposal and click **Continue**.

![](<../.gitbook/assets/image (9) (2).png>)

Depending on what actions the proposal is meant to take, you must define at least one for the proposal.

For example, transferring tokens will show the tokens held in the timelock treasury that can be transferred, and you can define who will receive the tokens and how many:

![](<../.gitbook/assets/image (10) (2).png>)

Alternatively, you can define a custom action that the proposal will perform. This is any action that calls a smart contract function, so you can use these to modify things like the Signata Governor contract, Timelock contract, and more.

To define a custom action, paste in a contract address to the **Target contract address**. If it's a contract that has been verified on Etherscan then the contract methods will automatically load. If it's not a verified contract you will need the contract ABI data to show the contract methods.

Once you've set the address, select the **Contract method** you want the vote to call. Then any parameters that are required for that contract method will appear, and you will need to define the values that they're changing to. In the example below the action is to call setVotingPeriod with the newVotingPeriod set to 1000.

![](<../.gitbook/assets/image (6).png>)

{% hint style="warning" %}
The DAO interface won't warn you if your proposal actions are technically possible or not. Some function calls require the DAO contract to hold certain rights, like ownership over another contract. If you aren't sure what you're doing, consult the DAO Proposal Managers for advice.
{% endhint %}

If you want a proposal to not actually perform any action at all because it's a "meta" change to the project, then simple define the action as a transfer of 0 ETH to any address.

Once you've defined the actions for the proposal (you can define up to 10, so if you need to execute a sequence of contract calls that's entirely possible), click **Continue**.

You will get a preview of your proposal. Click **Submit on-chain** if you want to get it opened immediately, or click **Save Draft** if you want to save it for later.

![](<../.gitbook/assets/image (23).png>)

Once a proposal is submitted on-chain a voting delay period starts. This period is by default set to 1 day, which gives people enough time to delegate their votes if required. While this delay period is in effect the proposal will show as "Pending" on Tally.

![](<../.gitbook/assets/image (4) (2) (1).png>)

## Queueing Proposals on Tally

{% hint style="info" %}
Proposals can only be queued once the voting period has finished, and the vote received more FOR votes than AGAINST/ABSTAIN votes.
{% endhint %}

To queue a proposal, open **Manage Proposals** on the Tally DAO interface and click **Queue**. You will be prompted by your wallet software to confirm the transaction, which will incur a network gas cost.

![](<../.gitbook/assets/image (21).png>)

A queued proposal must wait for the timelock period to expire before it can be executed. If you hold the right to cancel proposals, you can also cancel the proposal at this point if required.

## Executing Proposals on Tally

{% hint style="info" %}
Proposals can only be executed if they've been queued in the timelock and not still waiting for the timelock period to end, which by default is 1 day from when the proposal was queued.
{% endhint %}

To execute a proposal, open **Manage Proposals** on the Tally DAO interface and click **Execute**. You will be prompted by your wallet software to confirm the transaction, which will incur a network gas cost.

![](<../.gitbook/assets/image (19).png>)

Once the proposal has been executed the proposal process is complete.

