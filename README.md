# **Getting Started with Golem Base First Testnet (L2+L3)**

This guide is intended for Web3 developers and enthusiasts.

Follow these simple steps to start using Golem Base L2 "Erech" and L3 "Kaolin" testnets. 

During this tutorial you will:

* Bridge test ETH from Holesky to Golem Base L2 “Erech”  
* Run sample transaction on Golem Base L2  
* Bridge test ETH from Golem Base L2 “Erech” to Golem Base L3 “Kaolin”  
* Interact with data on Golem Base L3 either by writing your own code or through simple PoC built to demonstrate L3 capabilities \- dPaste.

**Disclaimer:** This guide is provided for informational and educational purposes only. Golem Base is an experimental, pre-production testnet platform intended for developers and technically proficient users. Terms and conditions apply: [glm.zone/base-toc](https://glm.zone/base-toc) 

## **Step 0: Organize a Wallet for Ethereum Testnet**

We recommend starting with [MetaMask](https://metamask.io/) as your browser-based wallet. You can install it [here](https://metamask.io/); simply click the Get Metamask link and follow the steps for getting started by agreeing to the terms and then clicking **Create a new wallet**. (However, you can use any other browser-based wallet as well.) 

## **Step 1: Get Test ETH on Holesky**

First, configure your Wallet to use Holesky Testnet. 

1. Open the MetaMask extension  
2. Click the Ethereum Mainnet icon; a box should pop up listing networks. Scroll to the bottom of this box and click **Add a custom network**.  
3. In the **Add a custom network** box, fill it in with the following:  
    **Network name:** Holesky  
    **Default RPC URL:** https://ethereum-holesky.publicnode.com  
    **Chain ID:** 17000  
    **Currency symbol:** ETH  
    **Block explorer URL:** [https://holesky.etherscan.io/](https://holesky.etherscan.io/)  
   

Grab some test ETH from any public Holesky faucet. (i.e. [Google](https://cloud.google.com/application/web3/faucet/ethereum/holesky), [Holesky proof of work](https://holesky-faucet.pk910.de/))  You'll bridge those test tokens to Golem Base Testnet “Erech” in the next step.

## **Step 2: Bridge Your ETH to Golem Base L2 “Erech”**

Transfer your test ETH to Golem Base bridge smart contract to move test ETH from Holesky to Golem Base L2:

1. In your wallet, select the **Holesky** network and click **Send**.  
2. Copy the following into the To box: `0x54D6C1435ac7B90a5d46d01EE2f22Ed6fF270ED3`  
   (This is a direct bridge contract address).  
3. Click **Continue**.  
4. A Transaction Request box will open. Click **Confirm**.

This transaction bridges your test ETH from Holesky to Golem Base L2 testnet "Erech" automatically. The bridging process typically takes just a few minutes

## **Step 3: Add Golem Base Testnet “Erech” to MetaMask / Wallet**

1. Follow the same steps as earlier for adding a network and use the following:  
    **Network name:** Golem Base L2 Testnet-001 "Erech"  
    **Default RPC URL:** [https://execution.holesky.l2.gobas.me](https://execution.holesky.l2.gobas.me)  
    **Chain ID:** 393530  
    **Currency symbol:** ETH  
    **Block explorer URL:** [https://explorer.golem-base.io](https://explorer.golem-base.io)  
2. Click Save. You should see a message at the bottom of the screen that the network was successfully added.

You can now switch between networks by clicking the network name in the upper-left corner and choosing one from the box that opens.

## **Step 4: Test Your L2 Setup**

Let's verify everything is working:

* Switch your wallet's network to **Golem Base L2 Testnet-001 "Erech"**.  
  * Tip: You should see the amount you sent in the previous step.  
* Optionally, you can test that the L2 Testnet setup is correct by sending a small amount of test ETH (like 0.001) to your own address.

  Visit Golem Base block explorer at [explorer.golem-base.io](http://explorer.golem-base.io) and look up your wallet address. You should see your transaction in the list, confirming you've successfully completed your first interaction with Golem Base\!

## **Step 5: Bridge from L2 to L3 "Kaolin"**

Now let's move some test ETH from L2 "Erech" to L3 "Kaolin":

* While still on the "Erech" network, send the desired amount of your L2 test ETH to L3 bridge: 0x5c857718caea1f6e9b0a7adf1415d0b98b6498d0 

* This will bridge your test ETH from L2 to Golem Base L3 testnet "Kaolin" 

* You can track your bridge transactions using bridging monitoring tool:

* Visit [https://bridgette.kaolin.holesky.golem-base.io/](https://bridgette.kaolin.holesky.golem-base.io/) to check if your L2→L3 bridge transaction is still pending or has been finalized. You’ll see a list of recent transactions associated with your wallet address.

## **Step 6: Add Golem Base L3 Testnet to MetaMask / Wallet**

Set up Golem Base L3 in your MetaMask / Wallet by adding a new network with these details:

* **Network Name:** Golem Base L3 Testnet "Kaolin"  
* **RPC URL:** [https://rpc.kaolin.holesky.golem-base.io/](https://rpc.kaolin.holesky.golem-base.io/)  
* **Chain ID:** 600606  
* **Currency Symbol:** ETH

## **Step 7: Verify Your L3 Setup**

Let's check that your L3 setup is working:

* Switch your wallet's network to "Golem Base L3 Testnet Kaolin"  
* After the bridging process is successful (typically less than a minute), you should see the test ETH in your wallet on L3 

## **Step 8: Explore the Golem Base Ecosystem**

Congratulations\! You've now successfully:

* Moved from Ethereum Holesky testnet to Golem Base L2 "Erech"  
* Bridged from L2 "Erech" to L3 "Kaolin"  
* Now you can experiment with L3 either by:  
  * [**Checking out dPaste**](https://dpaste.golem-base.io/) \- simple application built on top of Golem Base that allows you to save and share notes.   
  * **Create your own app \-** by using [**Golem Base SDK**](https://github.com/Golem-Base/typescript-sdk/). We will provide more tutorials soon. 
