---
description: Example of updating an unpublished smart contract
---

# Updating unpublished smart contract

One of the features of Acala EVM+ is the ability to verify the expected behaviour of your smart contract before the users can interact with it (before it is published). In the event of the business logic of the contract being broken, it can be updated without the need to redeploy a new version of the smart contract.

{% hint style="danger" %}
We can only update **unpublished** contracts.
{% endhint %}

Let's take a simple `Echo` smart contract that we deployed to the chain:

```solidity
pragma solidity =0.8.9;

contract Echo{
    string public echo;
    uint echoCount;

    event NewEcho(string message, uint count);

    constructor() {
        echo = "Deployed successfully!";
    }

    function scream(string memory message) public returns(string memory){
        echo = message;
        echoCount += 2;
        emit NewEcho(message, echoCount);
        return message;
    }
}
```

As the smart contract was deployed using an EVM account bound to the substrate account, that account can update the bytecode of an unpublished contract. This is because the EVM module considers deployer a maintainer of the smart contract.

The example smart contract has a faulty configuration, where it increments the `echoCount` by 2 instead of 1, so we can update the smart contract to be correct and compile it in order to get the new bytecode:

```solidity
pragma solidity =0.8.9;

contract Echo{
    string public echo;
    uint echoCount;

    event NewEcho(string message, uint count);

    constructor() {
        echo = "Deployed successfully!";
    }

    function scream(string memory message) public returns(string memory){
        echo = message;
        echoCount += 1;
        emit NewEcho(message, echoCount);
        return message;
    }
}
```

The code can be updated using extrinsics in the [Polkadot.js application](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fmandala-rpc.aca-staging.network%2Fws#/extrinsics). The call can be found under `Developer menu` in `Extrinsics`. Then select `evm` in the extrinsics selection dropdown and `setCode` call. The arguments passed to it is the address of the unpublished smart contract we are updating and the new bytecode.

![Developer > Extrinsics > evm > setCode](<../.gitbook/assets/image (9).png>)

Pressing `Submit transaction` will send the transaction to update the bytecode of the smart contract. Once you are done with testing it, you can [publish](../tooling/development-account/publishing-a-smart-contract.md#mark-a-given-contract-as-published-in-the-developer-section-of-the-polkadot-app) your smart contract.

{% hint style="danger" %}
Updating storage layout of your smart contract can result in unexpected behaviour, so only the business logic of the smart contracts should be updated in this way.
{% endhint %}
