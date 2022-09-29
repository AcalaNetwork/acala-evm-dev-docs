---
description: Walk through on deploying a smart contract to Acala EVM+ using Remix IDE.
---

# Deploying a smart contract

Assuming you have already connected Remix IDE to MetaMask and added the `Echo` smart contract from the [`Interacting with the deployed smart contracts`](interacting-with-the-deployed-smart-contracts.md) walk through, we can take a look at how to deploy the `Echo` smart contract using Remix IDE.

{% hint style="info" %}
This walk through focuses on specifics of deploying a smart contract to Acala EVM+ using Remix IDE. If you wish to learn more about deploying smart contracts using Remix IDE, please refer to the [official documentation](https://remix-ide.readthedocs.io/en/latest/create\_deploy.html#deploy-the-contract).
{% endhint %}

You should see a `scripts` folder under your `File explorers` section. We will be modifying the `deploy_ethers.js` in this walk through.



As the example smart contract is called `Echo`, we have to modify the 6th line in the file, so that the value of `contractName` variable is `'Echo'`.

```javascript
        const contractName = 'Echo'
```

Now that we have configured the script to deploy the `Echo` smart contract, we need to make sure, the transaction parameters are set correctly. As Acala EVM+ requires us to pay a storage rent fee in order to be able to deploy a smart contract to the network, we need to manually specify the transaction parameters `gasLimit` and `gasPrice`, to be passed to the deployment transaction. We have to set the `gasLimit` to a value of `0x329b140` and `gasPrice` to a value of `0x2f2ad303ea`.

```javascript
        const constructorArgs = [{gasLimit: 0x329b140, gasPrice: 0x2f2ad303ea}]
```

{% hint style="warning" %}
The gas limit and gas price of the example might get out of sync as new blocks are mined on Mandala test network. If you encounter OutOfStorage error, or any other for that matter, while trying to deploy your smart contract using the Remix IDE, we suggest verifying these values following [these](../../network/gas-parameters.md#for-developers) instructions.
{% endhint %}

This finishes up the modifications we need to do to the `deploy_ethers.js` file. We are now able to run the script, by option-clicking on it in the `File explorers` menu and selecting the `Run` option. This will open a MetaMask prompt, where we have to confirm the deploy transaction.

![](<../../.gitbook/assets/Screenshot from 2022-01-31 00-01-34.png>)

After the deployment transaction is included in a block, we can start interacting with our newly deployed smart contract, just like we did in the `Interacting with the deployed smart contracts` walk through.
