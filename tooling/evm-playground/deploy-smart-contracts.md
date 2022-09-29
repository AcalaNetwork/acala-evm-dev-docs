---
description: Instructions on how to deploy a smart contract using EVM playgrounds
---

# Deploy smart contracts

{% hint style="info" %}
This entry assumes that you have already [configured your Metamask](../metamask/connect-to-the-network.md) to connect to Acala EVM+.
{% endhint %}

To deploy a smart contract using [EVM playgrounds](https://evm.acala.network/), you need to compile your smart contract in your preferred development framework so that you have the ABI bundle available to upload.

## Upload the ABI bundle

Open the [Upload tab](https://evm.acala.network/#/upload) in the EVM playgrounds. Here you can upload an ABI bundle of the smart contract that you want to deploy. This will allow you to use it to deploy the smart contract:

![EVM playgrounds => Upload](<../../.gitbook/assets/image (35).png>)

Assign the `Name` of your smart contract. You will be able to identify the smart contract in the `Deploy` tab with it, once it gets uploaded.

To upload the ABI bundle itself, you can either drag and drop it into the upload section, or click on the section and select the file.

![EVM playgrounds => Upload => Add file](<../../.gitbook/assets/image (49).png>)

Once you have selected the correct ABI bundle, the methods of the smart contract should be displayed. You can verify that the correct methods are listed and press `Upload` to upload the ABI bundle.

## Deploy the smart contract

Smart contracts can be deployed under the [Deploy tab](https://evm.acala.network/#/deploy) of the EVM playgrounds.

The ABI bundles that you uploaded in the `Upload` tab can be seen here:

![EVM playgrounds => Deploy](<../../.gitbook/assets/image (33).png>)

The methods available for an ABI bundle can be seen by expanding the `ABI` menu. This can be helpful if you have multiple bundles uploaded and you want to be sure that you will be interacting with the right one.

![EVM playgrounds => Deploy => Expand ABI section ](<../../.gitbook/assets/image (26).png>)

When you have verified that you are interacting with the ABI bundle that has the correct methods available, you can click `Deploy`, which should open a deployment interface:

![EVM Playgrounds => Deploy => Deploy selected ABI bundle](<../../.gitbook/assets/image (55).png>)

The interface consists of the following components:

* Button to connect to your EVM wallet (this is why connecting MetaMask to the EVM+ is a prerequisite for this entry)
* Smart contract name, that can be changed, so you can deploy the same ABI bundle multiple times and easily differentiate between them
* ABI bundle identifications
* Fields to input the smart contract constructor parameters
* Value field to determine wether to send some of the native currency with the deploy transaction
* Fields to override the [`gas parameters`](../../network/gas-parameters.md)
* `Deploy` button to deploy the smart contract once you are satisfied with the deployment parameters

### 1. Connect your EVM wallet

Pressing the <img src="../../.gitbook/assets/image (57).png" alt="" data-size="line"> button will prompt your EVM wallet to connect to the site. You can select the account that you want to use with the EVM playgrounds and connect it.

![](<../../.gitbook/assets/image (51).png>)<img src="../../.gitbook/assets/image (47).png" alt="" data-size="original">

The selected account should be displayed at the top of the page now:

![Displayed deployment account](<../../.gitbook/assets/image (50).png>)

### 2. Update the required deployment parameters

Depending on the requirements, you can modify the deployment parameters of your smart contract. It is required to fill out the constructor parameters, but modifying other values is optional.

![Filled out deployment values](<../../.gitbook/assets/image (54).png>)

Once the values are filled out and double checked, the smart contract is ready to be deployed.

{% hint style="warning" %}
The `validUntil` field value has to be higher than the current block number, or the deployment transaction will fail, due to the validator treating it as outdated. You can verify the current block number in a [block explorer](../chain-explorer.md).
{% endhint %}

### 3. Deploy the smart contract

Once the parameters of deployment are ready, you can deploy the smart contract by pressing the `Deploy` button. This should prompt your EVM wallet to confirm your deployment transaction:

![Confirming the deployment transaction](<../../.gitbook/assets/image (53).png>)

The deployed smart contract can now be seen in the `Execute` tab, which is [further explained](interacting-with-smart-contracts.md) in the next entry.
