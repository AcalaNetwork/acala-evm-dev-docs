# Ethers.js usage example

The ethers.js library can be used to interact with the EVM+. In this example, we are retrieving the `developerStatus` of the [`EVM`](../../tutorials/hardhat-tutorials/evm-tutorial.md) contract.

This will validate wether the provided address has the developer status enabled or not. If you wish to learn more about the [development accounts](../development-account/), you can read up about them.

In order to be able to use the ethers.js library to interact with the [predeployed smart contract](../../network/precompiled-and-predeployed-smart-contracts/), we need to add it as well as the precompiled smart contracts to the projects:

```shell
yarn --dev ethers @acala-network/contracts
```

The following example imports the `ethers`.js library and instantiates the connection to the RPC node of the network. It also utilizes the [`ADDRESS`](../../network/precompiled-and-predeployed-smart-contracts/#address-utility) utility of `@acala-network/contracts` dependency and the precompiled `EVM` smart contract. We instantiate, call it using the `ethers.js` library and output the result:

```typescript
const { ethers } = require("ethers");
const { EVM } = require("@acala-network/contracts/utils/Address");
const EVMContract = require("@acala-network/contracts/build/contracts/EVM.json");
const provider = new ethers.providers.JsonRpcProvider("https://eth-rpc-mandala.aca-staging.network");

const evmContract = new ethers.Contract(EVM, EVMContract.abi, provider);
// const address = '0xC9CB0Bd811343Dbf6B9f61704d736dCD1975a7cE'; // is dev
const address = '0x869e7ed360498A21da9c7EdDFc15FB98Bd1Bf607'; // is not dev

evmContract.developerStatus(address).then(res => {
  console.log("The developerStatus is: ", res);
}).catch(err => {
  console.log("An error occured", err);
})
```

### Accessing the Oracle contract using Repl.it

{% embed url="https://replit.com/@GregoryLuneau/Get-aUSD-price-from-Acalas-Oracle?embed=true" %}
Fork it and test at will
{% endembed %}
