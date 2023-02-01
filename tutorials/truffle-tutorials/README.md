# Truffle tutorials

**If you are searching for** [**Waffle**](../waffle-tutorials.md) **or** [**Hardhat**](../hardhat-tutorials/) **tutorial, please follow the links.**

## Prerequisites

To be able to run the tutorial steps that require an operational development network (like deploying and testing), please refer to the [guide](https://github.com/AcalaNetwork/Acala#5-development) on how to setup a local development network.

{% hint style="warning" %}
**WARNING: All of the tests included within tutorials is meant to be run in the local development network and cold potentially fail on the public network.**
{% endhint %}

## Current tutorials

1. [hello-world](helloworld-tutorial.md): This tutorial contains instructions on how to setup a simple Truffle project that is compatible, deployable and testable with Acala EVM+.
2. [echo](echo-tutorial.md): This tutorial builds upon the previous one and adds return values to the function calls, events and changes of storage variables.
3. [token](token-tutorial.md): This tutorial builds upon the previous ones and adds the ERC20 token using OpenZeppelin dependency.
4. [NFT](nft-tutorial.md): This tutorial demonstrates how to build a NFT contract in Acala EVM+.

This section of tutorials uses Acala EVM+ specific mechanics and is incompatible with the legacy EVM. It introduces the use of our precompiled smart contracts that are accessible to anyone using the Acala EVM+

1. [precompiled-token](precompiledtoken-tutorial.md): This tutorial utilizes the precompiled and predeployed ERC20 tokens present in the Acala EVM+. It uses the `ADDRESS` utility, which serves as an automatic getter of the precompiled smart contract addresses, so we don't have to search for them in the documentation and hardcode them into our project.
2. [DEX](dex-tutorial.md): This tutorial utilizes the predeployed `DEX` smart contract to swap the ERC20 tokens of the predeployed `Token` smart contracts, which we instantiate with the help of the `ADDRESS` utility.
3. [EVM](evm-tutorial.md): This tutorial utilizes the predeployed `EVM` smart contract to manage the account preferences and the smart contract that the account maintains. It introduces the publishing of the smart contracts in the Acala EVM+ as well as enabling and disabling the developer mode of the account directly in the Acala EVM+.
4. [AdvancedEscrow](advanced-escrow-tutorial.md): This tutorial utilizes the predeployed `DEX`, `Token`s and `Schedule` smart contracts in order to build an escrow service that accepts any of the predeployed ERC20 tokens, swaps them for `AUSD` and at a set block releases the funds in `AUSD` or in another predeployed ERC20 token.
