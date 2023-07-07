# Web3.js usage example

The web3.js library can be used to interact with the EVM+. In this example, we are retrieving the `developerStatus` of the [`EVM`](../../examples/hardhat-tutorials/evm-tutorial.md) contract.

This will validate wether the provided address has the developer status enabled or not. If you wish to learn more about the [development accounts](../development-account/), you can read up about them.

In order to be able to use the web3.js library to interact with the [predeployed smart contract](broken-reference), we need to add it as well as the precompiled smart contracts to the projects:

```
yarn --dev web3 @acala-network/contracts
```

The following example imports the web3.js library and instantiates the connection to the RPC node of the network. It also utilizes the [`ADDRESS`](broken-reference) utility of `@acala-network/contracts` dependency and the precompiled `EVM` smart contract. We instantiate, call it using the `web3.js` library and output the result:

```typescript
const Web3 = require("web3");
const web3 = new Web3("https://eth-rpc-tc9.aca-staging.network");
const { EVM } = require("@acala-network/contracts/utils/Address");
const EVMContract = require("@acala-network/contracts/build/contracts/EVM.json");

const instance = new web3.eth.Contract(EVMContract.abi, EVM);
const address = '0xC9CB0Bd811343Dbf6B9f61704d736dCD1975a7cE'; // is dev
// const address = '0x869e7ed360498A21da9c7EdDFc15FB98Bd1Bf607'; // is not dev

instance.methods.developerStatus(address).call(function (err, res) {
  if (err) {
    console.log("An error occured", err);
    return;
  }
  console.log("The developerStatus is: ", res);
});
```
