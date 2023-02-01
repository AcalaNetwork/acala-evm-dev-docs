---
description: >-
  Section detailing the precompiled and predeployed smart contracts of the Acala
  EVM+.
---

# Predeployed smart contracts

One of the great features of Acala EVM+ is the precompiled and the predeployed smart contracts.

## Precompiled smart contracts

The precompiled smart contracts allow for the use of the predeployed smart contracts within your project's smart contracts and scripts. Just import the `@acala-network/contracts` package and import them into your project. The package includes the smart contracts to import into the smart contracts of your project as well as their compiled versions in order to import them into the deploy, test and interaction scripts.

To add them to your project simply use:

```shell
yarn add --dev @acala-network/contracts
```

Once you add the dependency to the project you can simply import the contracts included in it like this:

```solidity
import "@acala-network/contracts/token/Token.sol";
```

And to use the precompiled smart contract within your script use the:

```javascript
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");
```

For more information about how to use the precompiled smart contracts of the Acala EVM+ and how to interact with the predeployed smart contracts, please refer to [Hardhat](../../tutorials/hardhat-tutorials/), [Truffle](../../tutorials/truffle-tutorials/) or [Waffle](../../tutorials/waffle-tutorials.md) tutorials.

## Predeployed smart contracts

Predeployed smart contracts in Acala EVM+ allow for the reliable use of the smart contracts that are always deployed at the same address no matter the chain. The predeployed smart contracts include Tokens smart contracts, the native on chain scheduler called Schedule, Oracle, DEX and StateRent.

### ADDRESS utility

The `@acala-network/contracts` dependency contains the [`ADDRESS`](https://github.com/AcalaNetwork/predeploy-contracts/blob/master/contracts/utils/MandalaAddress.sol) utility which can be used in your smart contracts as well as scripts to access the predeployed smart contracts. It allows for using the addresses of the predeployed smart contracts without the need to copy-paste and hardcode these addresses into your project. There is an `ADDRESS` utility for each of the networks. The local development network and Mandala public test network use `MandalaAddress`, the Acala network uses the `AcalaAddress` and the Karura network uses the `KaruraAddress` to provide the correct addresses.

To use the utility within your smart contract simply import it using (for Mandala):

```solidity
import "@acala-network/contracts/utils/MandalaAddress.sol";
```

Make sure to set the inheritance of your contract to be able to interact with the `ADDRESS` utility:

```solidity
contract YourContract is ADDRESS {

}
```

To get the address of the DEX predeployed smart contract you can now simply use:

```solidity
ADDRESS.DEX
```

If you want to refer to the addresses of the predeployed smart contracts, take a look at the following page.
