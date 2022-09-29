---
description: >-
  Specifics of the development account (deposit, interacting with private
  contracts, how to transform own account)
---

# Development account

Smart Contracts can only be deployed by an address with the developer role.  Here is how you can enable that role on your account.

### Prerequisites:&#x20;

1. A Polkadot Account [Create with this Extension](https://polkadot.js.org/extension/)
2. An Ethereum Account [Create with this Extension](https://metamask.io/download/)
3. Some ACA or KAR in your Polkadot Account

### Enable Contract Development

1. Go to the Developer Section of the [Polkadot App](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-tc7-rpcnode.aca-dev.network%2Fws#/extrinsics) then select **Extrinsics**.
2. Select the target **account** from the top dropdown
3. Select **evm** from the **extrinsic** dropdown menu
4. Select **enableContractDevelopment()** from the method/action dropdown
5. Click **Submit Transaction** & **Sign and Submit**

![Developer > Extrinsic > Submission > evm > enableContractDevelopment()](<../../.gitbook/assets/image (21).png>)

#### Verify developer status

You can verify your or someone else's developer status by accessing a smart contract like in these examples:

{% content-ref url="../interaction-libraries/web3.js-usage-example.md" %}
[web3.js-usage-example.md](../interaction-libraries/web3.js-usage-example.md)
{% endcontent-ref %}

{% content-ref url="../interaction-libraries/ethers.js-usage-example.md" %}
[ethers.js-usage-example.md](../interaction-libraries/ethers.js-usage-example.md)
{% endcontent-ref %}

### Bind Accounts

#### The easy way

One of our developers created a simple tool to bing your Substrate and EVM addresses. The tool is still in the early stages of refinement, but it is operational, so you can use it. In case this tool has any issues with binging your addresses, please refer to the following section, called **The manual way**, and follow the binding instructions there.

{% embed url="https://acala-evm.netlify.app" %}
Binding EVM & Substrate Accounts
{% endembed %}

#### The manual way

#### Step 1: Get the Genesis hash

1. Select the **Metadata** from the **Settings** Section of the [Polkadot App](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-tc7-rpcnode.aca-dev.network%2Fws#/settings/metadata)
2. Copy the **Genesis Hash** hex string

![Step 1: Getting the Genesis hash](<../../.gitbook/assets/image (5).png>)

#### Step 2: Get the Chain id for your target chain

![Developer > Chain state > Storage > evm > chainId](<../../.gitbook/assets/image (30).png>)

1. Select the **Developer** tab, then **Chain state** from the dropdown
2. Select **Storage** and then **evm** from the **state query** dropdown
3. Choose **chainId** from  the method/action dropdown
4. Click the **+** button on the right

#### Step 3: Create the signature of the claim on the [EVM+ Playground](https://evm.acala.network/#/Bind%20Account)

1. Select the right account in Metamask
2. Fill in the **Substrate address**, **Chain id** & **Genesis hash**
3. Click **Sign** & copy the **signature** to the next step

![Step 2: Create the signature of the claim ](<../../.gitbook/assets/image (27).png>)

#### Step 4: Claim Account on the Developer Section of the [Polkadot App](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-tc7-rpcnode.aca-dev.network%2Fws#/extrinsics)

The **ethAddress** should be the same as your metamask wallet address that you used above to generate the signature.

1. Select **evmAcounts** from the **extrinsic** dropdown menu
2. Select **claimAccount(ethAddress, ethSignature)** from the method/action dropdown
3. Fill in the **ethAddress** & **ethSignature**
4. Click **Submit Transaction**

![Step 3: Fill in eth Address and eth Signature](<../../.gitbook/assets/image (52).png>)

#### Step 5: Confirm the bindings

1. Select the **Developer** tab, then **Chain state** from the dropdown
2. Select **Storage** and then **evmAccounts** from the **state query** dropdown
3. Click the **+** button on the right
4. Double check that the **evmAccounts.evmAddresses** is indeed the right one.

![Developer > Chain state > Storage > evmAccounts > evmAddresses](<../../.gitbook/assets/image (42).png>)

{% content-ref url="../metamask/" %}
[metamask](../metamask/)
{% endcontent-ref %}
