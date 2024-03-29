---
description: >-
  Specifics of the development account (deposit, interacting with private
  contracts, how to transform own account)
---

# Development account

Smart Contracts can only be deployed by an address with the developer role.  Here is how you can enable that role on your account.

## Prerequisites:&#x20;

1. A Polkadot Account [Create with Polkadot.js](https://polkadot.js.org/extension/)
2. An Ethereum Account [Create with Metamask](https://metamask.io/download/)
3. Some ACA or KAR in your Polkadot Account

## Enable Contract Development

1. Go to the Developer Section of the [Polkadot App](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-tc9-rpc.aca-staging.network%2Fws#/extrinsics) then select **Extrinsics**.
2. Select the target **account** from the top dropdown
3. Select **evm** from the **extrinsic** dropdown menu
4. Select **enableContractDevelopment()** from the method/action dropdown
5. Click **Submit Transaction** & **Sign and Submit**

![Developer > Extrinsic > Submission > evm > enableContractDevelopment()](<../../.gitbook/assets/image (21).png>)

### Verify developer status

You can verify your or someone else's developer status by accessing a smart contract like in these examples:

{% content-ref url="../interaction-libraries/web3.js-usage-example.md" %}
[web3.js-usage-example.md](../interaction-libraries/web3.js-usage-example.md)
{% endcontent-ref %}

{% content-ref url="../interaction-libraries/ethers.js-usage-example.md" %}
[ethers.js-usage-example.md](../interaction-libraries/ethers.js-usage-example.md)
{% endcontent-ref %}

## Bind Accounts
### On Karura/Acala
binding evm and substrate accounts can be done easily on our Dapp UI:
- [karura](https://apps.karura.network/bridge/bind-address)
- [acala](https://apps.acala.network/bridge/bind-address)

for more details, checkout [address binding](https://guide.acalaapps.wiki/general/address-binding) on acala wiki.

### On Mandala
There is no UI available for Mandala address binding, but we can still do this manually.

#### 1) Create the signature of the claim on the [EVM+ Playground](https://evm.acala.network/#/Bind%20Account)

1. Select the right account in Metamask
2. Fill in: 
   - **Substrate address**: you substrate address
   -  **Chain id**: 595
   -  **Genesis hash**: `0x3035b88c212be330a1a724c675d56d53a5016ec32af1790738832db0227ac54c`
3. Click **Sign** & copy the **signature** to the next step

#### 2) Claim Account on the Developer Section of the [Polkadot App](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-tc9-rpc.aca-staging.network%2Fws#/extrinsics)

The **ethAddress** should be the same as your metamask wallet address that you used above to generate the signature.

1. Select **evmAcounts** from the **extrinsic** dropdown menu
2. Select **claimAccount(ethAddress, ethSignature)** from the method/action dropdown
3. Fill in the **ethAddress** & **ethSignature**
4. Click **Submit Transaction**

![Step 3: Fill in eth Address and eth Signature](<../../.gitbook/assets/image (52).png>)

#### 3) Confirm the bindings

1. Select the **Developer** tab, then **Chain state** from the dropdown
2. Select **Storage** and then **evmAccounts** from the **state query** dropdown
3. Click the **+** button on the right
4. Double check that the **evmAccounts.evmAddresses** is indeed the right one.

![Developer > Chain state > Storage > evmAccounts > evmAddresses](<../../.gitbook/assets/image (42).png>)

{% content-ref url="../metamask/" %}
[metamask](../metamask/)
{% endcontent-ref %}
