# How to Add a Custom Network RPC

With the many EVM-compatible networks popping up, adding custom networks will be essential to navigating your favorite DApps and DEXes.

**What is a RPC?**

RPC stands for remote procedure call, a set of protocols that allow a client such as BlockWallet to interact with a blockchain. Adding network details in the manual process outlined below allows BlockWallet to connect to a particular network and interact with it.

Note that if you’re operating BlockWallet from multiple devices, adding a network on one device does not mean it will appear on another device even when accessing the same account.

Also, you will need to re-add custom networks if you uninstall the BlockWallet extension.

### Using Chainlist to add a network

The easiest way to add a custom network is [ChainList](https://chainlist.org/). It’s a trusted service which does not require technical details like an RPC URL or Chain ID.

We recommend taking a look at the list of networks available on [ChainList](https://chainlist.org/) and adding networks from there, especially for beginners. For a helping hand, check out our HelpDesk article on[ How to Add a Network Using Chainlist.](https://help.blockwallet.io/hc/en-us/articles/7786076532497)

### Adding a network

Follow the steps below to add a custom network manually.

1. In the main wallet screen, select the gear icon on the top-right-hand corner.
2. Select “Networks”
3. Select “+ Add New Network”
4. Type in the network's Chain Identification or Chain Name
   1. Note that if you are looking for a **testnet** OR **networks supported by default** you will only see those results by modifying your search filter by selecting the icon in the top right-hand corner.
5. If your search query comes back with zero results, it’s due to one of the following reasons:
   1. You entered the Chain Identification or Chain Name incorrectly. Double-check your entry and try again.
   2. Your desired network is a non-recognized network and it needs to be added manually.
6. To add a network manually, enter the Network Name, RPC URL, Chain ID, and Currency Symbol.
7. Once this info has been entered, press “Save”.

### Editing network details

_This is an advanced feature. If you’re not 100% sure with what you’re editing, do not change network details._

You have the option to edit network details as long as it’s not selected for use. To do so, navigate to the “Network” tab in settings, select a network, and edit the details. Remember to press save once you’re done.

### Risks of adding an unknown network

As the blockchain ecosystem expands, many custom networks are built, but not all of them can be trusted. This is an inherent risk to adding custom networks.

Here are a couple of best practices when adding custom networks:

* Do your due diligence
  * BlockWallet does not verify non-recognized networks, so be sure you trust the network provider before adding it to your BlockWallet account. A malicious network provider can lie about the state of the blockchain, withhold transactions, and record your network activity and IP address.
  * Additionally, the quality of the underlying RPC is not guaranteed, the wallet experience can be degraded because of a low-quality RPC provided by the custom network.
* Always use established bridges or portals to move tokens between networks.
  * When bridging your tokens (moving your tokens from one chain to another), make sure you trust the network operator and the receiving address you send funds.
  * **If you attempt to send crypto assets directly from one network or chain to another, it will most likely result in permanent and irreversible asset loss.**
* Make sure that custom network information is accurate.
  * Always verify custom network information before adding it to your BlockWallet account. As mentioned above, BlockWallet does not verify non-recognized networks.
