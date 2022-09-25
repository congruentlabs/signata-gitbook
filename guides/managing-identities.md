# Managing Identities

To access the Signata Identity Manager, use the following link:

{% embed url="https://my.signata.net" %}

## Wallet Identities

### Creating a Wallet Identity

In the **Add Identity** section, click on the **Wallet** tab.

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

Click **Generate** to randomly generate a new set of Identity seeds. The delegate for your identity will be your connected wallet. Click **Add Identity**.

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Your wallet will pop up asking you for a Signature. **This is not a payment signature**, this signature just proves you control your wallet and will let you save your identity data to IPFS.

<figure><img src="../.gitbook/assets/image (3) (3).png" alt=""><figcaption></figcaption></figure>

### Registering your Wallet Identity

Once an identity is created it needs to be **registered** on a blockchain to be usable.

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Sign the request from your wallet to register the identity.

{% hint style="warning" %}
The wallet you have connected to your browser will pay for the transaction fees, and will link the creation of the identity to that wallet. If you're wanting to isolate your wallet on-chain, make sure you don't pay for transactions with the wrong connected wallet.
{% endhint %}

### Renaming a Wallet Identity

{% hint style="info" %}
Identity names are just so you can easily identify which identity is which. The names are not visible to anyone else, and are not written to the blockchain.
{% endhint %}

In your identity, click **Rename**.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

A confirmation window will appear. Specify your new identity name and click **Save New Name**.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Deleting a Wallet Identity

{% hint style="warning" %}
Identities can only be deleted if they haven't been registered. Registering your identity writes it to the blockchain, and so you can destroy it on-chain but not delete it once it's registered.
{% endhint %}

Click **Delete** in the identity options.

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

A confirmation window will appear. Click **Delete Identity**.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### Locking a Wallet Identity



Click **Lock**.

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

A confirmation popup will appear. Click **Lock Identity**.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Confirm the signature request in your browser. Your wallet may show a warning about signing the message - this warning can be ignored.\


<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Once you have signed the message, confirm the transaction in your web3 wallet.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Unlocking a Wallet Identity

Click **Unlock**.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

A confirmation popup will appear. Click **Unlock Identity**.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Confirm the transaction in your web3 wallet.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

