# How to Use BlockWallet Bridges

BlockWallet Bridges lets you move tokens from one blockchain to another, right inside of BlockWallet. Limit your exposure to potentially malicious third-party protocols.

## Before you start bridging

Here are a couple of tips and safety reminders you should know about before using BlockWallet Bridges.

### Slippage

Similar to token slippage, slippage is the amount of change between the price the bridge was initiated at and the final transaction price that BlockWallet will tolerate. Given the highly liquid and fluctuating nature of cryptocurrencies, prices can change from second to second. By default, slippage is set to 0.5% - if you’d like to increase or decrease the slippage on your swap, select the “Settings” tab on the bridges confirmation page (after you click “Review”).

### Gas Fees

Even though there are zero trading fees for your bridges, you still need to pay gas fees. **Make sure you have enough native network tokens to pay the gas fees on your current chain AND destination chain!**

### Border Cases

While very simple to use, this is still crypto and while we’re still so early, lots of different border cases can occur. Check out the list below to make sure you’re prepared if you ever run into one of these scenarios.

#### Missing Native Token on Destination Chain

We just mentioned it, but since this is by far the most common error when bridging, we’re going to mention it again.

**When bridging, please make sure you have enough native network tokens to pay the gas fees on your current chain AND destination chain!**

For example, if you’re bridging USDT from the Ethereum Network to BNB Chain, make sure you have enough ETH tokens and BNB tokens as well. If you don’t have any BNB tokens in your account, you won’t have the gas fees to move the bridged USDT anywhere. Don’t worry though, we have warnings in place to help you avoid this.

#### Alternate Token Refunds

Just like the name implies, alternate token refunds are when the bridge operation fails, and the user is refunded an alternate token from the one that was used to initiate transaction.

For example, if you want to bridge 5 DAI from Ethereum to Polygon, but somehow the bridge operation fails. During an alternate token refund, instead of being refunded your 5 DAI, you would receive 4.9988 USDC instead.

A similar border case to alternate token refunds is **alternate token bridges**. This happens when the user receives a different token on the destination chain than the one they initiated the bridge with.

For example, if you wanted to bridge 5 USDC from Polygon to Gnosis. With an **alternate token bridge**, you would receive 5.0005 DAI on the Gnosis chain instead of the 5 USDC you’d be expecting.

While these situations aren’t ideal, keep in mind that the **token asset value remains the same in both scenarios.**

#### Provider/Tool Errors

When requesting a quote or routes to bridge your tokens, it’s possible that one or more tools that we’re using won’t be able to provide results. The possible error messages you may run into are outlined below, followed by what caused to error message to appear.

NO\_POSSIBLE\_ROUTE: No route was found for this action.

INSUFFICIENT\_LIQUIDITY: The tool's liquidity is insufficient.

TOOL\_TIMEOUT: The third-party tool timed out.

UNKNOWN\_ERROR: An unknown error occurred.

RPC\_ERROR: There was a problem getting on-chain data. Please try again later.

AMOUNT\_TOO\_LOW: The initial amount is too low to transfer using this tool.

AMOUNT\_TOO\_HIGH: The initial amount is too high to transfer using this tool.

FEES\_HGHER\_THAN\_AMOUNT: The fees are higher than the initial amount -- this would result in a negative resulting token.

DIFFERENT\_RECIPIENT\_NOT\_SUPPORTED: This tool does not support different recipient addresses.

TOOL\_SPECIFIC\_ERROR: The third-party tool returned an error.

CANNOT\_GUARANTEE\_MIN\_AMOUNT: The tool cannot guarantee that the minimum amount will be met.

### How to Bridge Tokens

1. Select the desired network you’d like to bridge tokens from.
2. Select the token you would like to bridge from the “Assets” tab on the home screen.
3. Select the Bridge icon.
4. Enter the number of tokens you would like to bridge.
5. Select the network you would like to bridge to.
6. Press “Review”.
7. On the Bridge confirmation page, there are several features to ensure you’re satisfied with your bridge.
   1. In the “Settings” tab, you can customize slippage and also turn on Flashbots Protection. This option will only be available for transactions consuming more than 42,000 gas.
   2. In the “Details” tab, you can view the contract address and other details like fees and rates.
8. If you are satisfied with the details of your transactions, press “Bridge”.
9. Once the bridge transaction has been initiated, the “Activity” tab on BlockWallet home screen will show that your “BlockWallet Bridge” is pending. You can also check the “Activity” tab in the destination network to confirm there’s an “Incoming Bridge”.
10. Once the bridge is complete, your tokens will now be available on the destination network!

For all the visual learners out there, don’t worry, we’ve got you covered! Here’s a screen recording walkthrough on how to bridge ETH tokens from the Ethereum network to the Polygon network. In the recording, you’ll see the details of all the steps outlined in the instructions above.

![ETH\_to\_BNBCHAIN\_GIF.gif](https://help.blockwallet.io/hc/article\_attachments/10790529073425)
