# Using Veriswap

## Veriswap Design

{% content-ref url="../design/veriswap-design.md" %}
[veriswap-design.md](../design/veriswap-design.md)
{% endcontent-ref %}

## Create a Swap

Creating a swap requires 3 pieces of information:

1. The token you're sending, and how much you'll send,
2. The token you're receiving, and how much you'll receive,
3. The wallet that will execute the swap.

To get started, browse to the Veriswap website and connect your wallet

{% embed url="https://veriswap.io" %}

![](<../.gitbook/assets/image (14) (1).png>)

Once your wallet is connected, you can start by specifying the token to send. You need to paste in a valid ERC20 token address of the token you're sending.

![](<../.gitbook/assets/image (3) (1) (1).png>)

When you've pasted in a valid ERC20 token address you can then specify the amount you want to send. To save time you can click on the purple percentage amount buttons to automatically fill in a that percentage of your wallet balance of that token. If you click 100%, you will be sending all of your balance of that token.

![](<../.gitbook/assets/image (1) (1).png>)

![](<../.gitbook/assets/image (8) (1) (1).png>)

Now specify which token you want to receive in the swap. Once you provide a valid ERC20 token address you will be able to specify how many tokens you will receive in the swap.

The swap won't check that the executor has enough tokens to complete the swap at this stage.

![](<../.gitbook/assets/image (4) (1).png>)

Now set the executor of the swap. This is the wallet address of the other party that you want to complete the swap. Nobody else but the executor can complete the swap.

Specifying the executor will reveal extra options to Require Signata Identity and Enforce Risk Detection. You can enable these if you wish for the swap.

![](<../.gitbook/assets/image (6) (1).png>)

Now, you must **Approve** and then **Open Swap**. Both of these actions require signatures submitted on the blockchain, and will incur network gas fees. Approval is needed to let the Veriswap contract transfer your tokens out of your wallet, and Open Swap creates the swap record on the chain.

![](<../.gitbook/assets/image (9) (1).png>)

![](<../.gitbook/assets/image (10) (1) (1).png>)

Once the swap has been opened on the blockchain, you will see a confirmation message. You also now cannot create any more swaps until the currently open swap is completed or canceled.

![](<../.gitbook/assets/image (15) (1).png>)

You will see a link at the top of the page to **Manage Your Swap**.

![](<../.gitbook/assets/image (17) (1).png>)

Click on the button to open the swap information page. You can give the URL of the page you're looking at to the executor so they can complete the swap.

![](<../.gitbook/assets/image (16) (1).png>)

## Execute a Swap

To execute a swap, go to the swap URL provided to you by the creator of the swap and connect your wallet.

![](<../.gitbook/assets/image (12) (1).png>)

You will be shown details about the swap for you to review before you execute.

![](<../.gitbook/assets/image (11) (1).png>)

If you are content with the swap details, click **Approve** and then **Complete Swap**. Both of these actions require signatures submitted on the blockchain, and will incur network gas fees. Approval is needed to let the Veriswap contract transfer your tokens out of your wallet, and Complete Swap executes the swap record on the chain.

![](<../.gitbook/assets/image (13) (1).png>)

## Change Executor or Cancel Swap

If you need to change the executor of a swap after it's been created, you can simply open up your created swap. You'll see a message at the top confirming that you're looking at your own swap, and a warning that you're not the designed executor of it.

![](<../.gitbook/assets/image (2) (1) (1).png>)

Down the bottom you will find a textbox to enter a new executor address. Provide a new valid wallet address and click **Change Executor**.

![](<../.gitbook/assets/image (5) (1) (1) (1).png>)

If you wish to cancel the swap, just click **Cancel Swap** at the bottom of the page.

![](<../.gitbook/assets/image (1) (1) (1).png>)

