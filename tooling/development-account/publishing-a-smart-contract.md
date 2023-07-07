---
description: Making a smart contract public
---

# Publishing a smart contract

Contracts will initially only be accessible by the contract developers, a special role that anyone can opt-in to.

Once the contracts have been deployed and tested, they can be made public by paying a small amount of ACA in order for them to become publicly accessible.

Trying to interact with an unpublished contract with an account that does not have the developer role will receive an `Error: -32603` with the `message: "Internal JSON-RPC error."`

{% hint style="info" %}
Only the contract's maintainer can publish it, no one else.
{% endhint %}

## Publish a contract
We can publish a contract either with [substrate account](#with-substrate-account) or [evm account](#with-evm-account).
### with substrate account
{% hint style="info" %}
The example below uses Mandala as example. For other networks, you will need to switch to different [Polkadot App](../chain-explorer.md#polkadotjs-app).
{% endhint %}

First make sure the substrate account is bound to the maintainer of the contract to be published, then we can publish the contract:

1. Under the **Submission** tab of [Polkadot App](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-tc9-rpc.aca-staging.network%2Fws#/extrinsics), select **evm** from the **extrinsic** dropdown menu
2. Select **publishContract(contract)** from the method/action dropdown
3. Fill in the **contract** address
4. Click **Submit Transaction**

![Developer > Extrinsic > Submission > evm > publishContract(contract)](<../../.gitbook/assets/image (1).png>)

### with evm account
A simple example can be found [here](https://github.com/AcalaNetwork/asset-router/blob/master/scripts/publish.ts)

## Verify that the smart contract has been published successfully

Polkadot App can be used to verify whether or not the smart contract is published. This can be done using the [Chain state](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-tc9-rpc.aca-staging.network%2Fws#/chainstate) viewer:

1. Using the **Storage** tab, select the **evm** from the **select state query** dropdown
2. Select **accounts(H160)** from the method/action dropdown
3. Fill in the **contract** address
4. Click **Submit transaction**

![Developer > Chain state > Storage > evm > accounts(H160)](<../../.gitbook/assets/image (31).png>)

The `published` value of the response indicates whether the smart contract is published or not.

## Child smart contracts

In case your smart contract has a contract factory function, the child smart contracts inherit the publication status of the parent smart contract.

This means that the child smart contracts created using a published smart contract's contract factory are already published and don't need to be published manually.

Keep in mind that the maintainer of a child smart contract is the parent smart contract and not the caller of the transaction that triggered its creation. In case the maintainer role of the smart contract needs to be transferred from the parent smart contract to the caller, the transfer should be implemented in the same call as the creation of the child smart contract. You can do so using the [EVM predeployed smart contract](https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/EVM).
