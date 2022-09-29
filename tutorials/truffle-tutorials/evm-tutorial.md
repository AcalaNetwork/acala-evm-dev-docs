---
description: >-
  This tutorial utilizes the predeployed EVM smart contract to manage the
  account preferences and the smart contract that the account maintains.
---

# EVM tutorial

## Table of contents

* [About](evm-tutorial.md#about)
* [Smart contract](evm-tutorial.md#smart-contract)
* [Test](evm-tutorial.md#test)
* [User journey](evm-tutorial.md#user-journey)
* [Summary](evm-tutorial.md#summary)

## About

This example introduces the use of Acala EVM+ predeployed EVM that is present on every network at a fixed address (the address of a predeployed contract is the same on a local development network, public test network as well as the production network). As this example focuses on showcasing the interactions with the predeployed `EVM`, it doesn't have its own smart contract. We will get all of the required imports from the [`@acala-network/contracts`](https://github.com/AcalaNetwork/predeploy-contracts) dependency. The precompiles and predeploys are a specific feature of the Acala EVM+, so this tutorial is no longer compatible with traditional EVM development networks (like Ganache).

Let's take a look!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/truffle-tutorials/tree/master/EVM](https://github.com/AcalaNetwork/truffle-tutorials/tree/master/EVM)
{% endhint %}

## Smart contract

Smart contracts in this tutorial are only used to satisfy the Truffle's requirement to have a smart contract to compile. For this, we will create two empty smart contracts that will inherit the `EVM` and `Token` from `@acala-network/contracts` dependency. Your skeleton `PrecompiedEVM` smart contract should look like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

contract PrecompiledEVM {

}
```

Import of the `EVM` from `@acala-network/contracts` is done between the `pragma` definition and the start od the `contract` block:

```solidity
import "@acala-network/contracts/evm/EVM.sol";
```

As we now have access to `EVM.sol` from `@acala-network/contracts`, we can set the inheritance of our `PrecompiledEVM` contract:

```solidity
contract PrecompiledEVM is EVM {
```

This concludes our `PrecompiledEVM` smart contract.

<details>

<summary>Your contracts/PrecompiledEVM.sol should look like this:</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

import "@acala-network/contracts/evm/EVM.sol";

contract PrecompiledEVM is EVM {
    
}
```

</details>

{% hint style="info" %}
**NOTE: In order for Truffle to properly use the Token precompiles (used in the following section), we need to add the** [**PrecompiledToken.sol**](precompiledtoken-tutorial.md#smart-contract) **smart contract as well.**
{% endhint %}

## Test

Tests for this tutorial will validate the expected values returned and expected behaviour of EVM predeployed smart contract. The test file in our case is called `EVM.js`. Within it we import the `EVM` and `Token` from `@acala-network/contracts` dependency and assign it to `PrecompiledEVM` and `PrecompiledToken` variables. The `EVM`, which is the export from the `ADDRESS` utility of `@acala-network/contracts` dependency, is imported and it holds the value of the address of the predeployed EVM smart contract. We are also importing `truffleAssert` in order to ease our verification of the expected values and we are defining the `NULL_ADDRESS` constant, so we don't have to copy-paste the value when needed.

The test file with import statements and an empty test should look like this:

```javascript
const PrecompiledEVM = artifacts.require("@acala-network/contracts/build/contracts/EVM");
const PrecompiledToken = artifacts.require("@acala-network/contracts/build/contracts/Token");

const truffleAssert = require("truffle-assertions");

const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract("PrecompiledEVM", function (accounts) {
  
});
```

To prepare for the testing, we have to define four global variables, `instance`, `contract`, `deployer` and `user`. The `instance` will store the predeployed EVM smart contract instance. `contract` will store the value of the deployed precompiled token smart contract. The `deployer` and `user` will store the accounts that we will be using in our tests. Let's assign them values in the `beforeEach` action:

```javascript
  let instance;
  let contract;
  let deployer;
  let user;

  beforeEach("setup development environment", async function () {
    [deployer, user] = accounts;
    instance = await PrecompiledEVM.at(EVM);
    contract = await PrecompiledToken.new();
  });
```

You can see how we used the `EVM` from the `ADDRESS` utility in order to set the address of our predeployed smart contract.

Our test will only contain one top-level section called `Operation` in which we will be checking the following functions (which will each be tested in its own section):

1. `newContractExtraBytes()` function to get the NewContractExtraBytes constant.
2. `storageDepositPerByte()` function to get the StorageDepositPerByte constant.
3. `maintainerOf()` function to get the maintainer of the contract.
4. `developerDeposit()` function to get the value of the developer deposit.
5. `publicationFee()` function to get the vale of the publication fee.
6. `transferMaintainter()` function to transfer the maintainer role of the smart contract.
7. `publishContract()` function that publishes a smart contract (makes it available for interactions to the non-developer accounts).
8. `developerStatus()` function that returns the development mode status of the address.
9. `developerDisable()` function that disables developer mode of the account.
10. `developerEnable()` function that enables developer mode of the account.

The structure described above without the checks, should look like this:

```javascript
  describe("Operation", function () {
    describe("newContractExtraBytes()", function () {
      
    });

    describe("storageDepositPerByte()", function () {
      
    });

    describe("maintainerOf()", function () {
      
    });

    describe("developerDeposit()", function () {

    });

    describe("publicationFee()", function () {

    });

    describe("transferMaintainter()", function () {

    });

    describe("publishContract()", function () {
      
    });

    describe("developerStatus()", function () {
      
    });

    describe("developerDisable()", function () {
      
    });

    describe("developerEnable()", function () {
      
    });
  });
```

When validating the `newContractExtraBytes()` function, we will check for the following examples:

1. New contract extra bytes are returned.

The section should look like this:

```javascript
      it("should return the new contract extra bytes", async function () {
        const response = await instance.newContractExtraBytes();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `storageDepositPerByte()` function, we will check for the following examples:

1. Storage deposit per byte is returned.

The section should look like this:

```javascript
      it("should return the storage deposit", async function () {
        const response = await instance.storageDepositPerByte();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `maintainerOf()` function, we will check for the following examples:

1. Maintainer of the contract should be returned

The section should look like this:

```javascript
      it("should return the developer deposit", async function () {
        const response = await instance.developerDeposit();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `developerDeposit()` function, we will check for the following examples:

1. Value of the developer deposit should be returned.

The section should look like this:

```javascript
      it("should return the developer deposit", async function () {
        const response = await instance.developerDeposit();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `publicationFee()` function, we will check for the following examples:

1. Publication fee should be returned.

The section should look like this:

```javascript
      it("should return the publication fee", async function () {
        const response = await instance.publicationFee();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `transferMaintainter()` function, we will check for the following examples:

1. Maintainer of the contract should be transferred if the caller is current maintainer.
2. `TransferredMaintainer` event should be emitted when the maintainer is transferred.
3. It should revert if the caller is not the maintainer of the contract.
4. It should revert if trying to transfer the maintainer of the `0x0`.
5. It should revert if trying to transfer the maintainer to `0x0` address.

The section should look like this:

```javascript
      it("should transfer the maintainer of the contract", async function () {
        const initialOwner = await instance.maintainerOf(contract.address);

        await instance.transferMaintainer(contract.address, user, { from: deployer });

        const finalOwner = await instance.maintainerOf(contract.address);

        expect(initialOwner).to.equal(deployer);
        expect(finalOwner).to.equal(user);
      });

      it("should emit TransferredMaintainer when maintainer role of the contract is transferred", async function () {
        const maintainer = await instance.maintainerOf(contract.address);

        // Maintainer needs to be set to deployer in case any settings from the previous examples
        // remain. This is because Truffle only executes the beforeEach action before each of the
        // first-level nested describe blocks, but not the second ones.
        if(maintainer == user){
          await instance.transferMaintainer(contract.address, deployer, { from: user });
        }

        truffleAssert.eventEmitted(
          await instance.transferMaintainer(contract.address, user, { from: deployer }),
          "TransferredMaintainer",
          { contractAddress: contract.address, newMaintainer: user }
        );
      });

      it("should revert if the caller is not the maintainer of the contract", async function () {
        const maintainer = await instance.maintainerOf(contract.address);

        // Maintainer needs to be set to deployer in case any settings from the previous examples
        // remain. This is because Truffle only executes the beforeEach action before each of the
        // first-level nested describe blocks, but not the second ones.
        if(maintainer == user){
          await instance.transferMaintainer(contract.address, deployer, { from: user });
        }

        await truffleAssert.reverts(
          instance.transferMaintainer(contract.address, deployer, { from: user })
        );
      });

      it("should revert if trying to transfer maintainer of 0x0", async function () {
        await truffleAssert.reverts(
          instance.transferMaintainer(NULL_ADDRESS, user, { from: deployer }),
          "EVM: the contractAddress is the zero address"
        );
      });

      it("should revert when trying to transfer maintainer to 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.transferMaintainer(contract.address, NULL_ADDRESS, { from: deployer }),
          "EVM: the newMaintainer is the zero address"
        );
      });
```

{% hint style="info" %}
**NOTE: As the Truffle beforeEach action is only run before the nested describe block, we need to make sure that no changes from the previous test examples don't affect the following ones.**
{% endhint %}

When validating the `publishContract()` function, we will check for the following examples:

1. `ContractPublished` event should be emitted when the contract is published.
2. It should revert when the caller is not the maintainer of the contract.
3. It should revert when trying to publish `0x0` contract.

The section should look like this:

```javascript
      it("should fail when caller is not the maintainer of the contract", async function () {
        await truffleAssert.fails(instance.publishContract(contract, { from: user }));
      });

      it("should revert when trying to publish 0x0 contract", async function () {
        await truffleAssert.reverts(
          instance.publishContract(NULL_ADDRESS, { from: deployer }),
          "EVM: the contractAddress is the zero address"
        );
      });

      it("should emit ContractPublished event", async function () {
        truffleAssert.eventEmitted(
          await instance.publishContract(contract.address, { from: deployer }),
          "ContractPublished",
          { contractAddress: contract.address }
        );
      });
```

When validating the `developerStatus()` function, we will check for the following examples:

1. It should return the developer mode status of the address.

The section should look like this:

```javascript
      it("should return the status of the development account", async function () {
        const randomAddress = "0xabcabcabcabcabcabcabcabcabcabcabcabcabca";

        const responseTrue = await instance.developerStatus(deployer);
        const responseFalse = await instance.developerStatus(randomAddress);

        expect(responseTrue).to.be.true;
        expect(responseFalse).to.be.false;
      });
```

When validating the `developerDisable()` function, we will check for the following examples:

1. It should disable developer mode.
2. `DeveloperDisabled` event is emitted when developer mode is disabled.
3. It should revert if developer mode is already disabled.

The section should look like this:

```javascript
      it("should disable development mode", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(!setupStatus){
          await instance.developerEnable({ from: user });
        }

        const initalStatus = await instance.developerStatus(user);

        await instance.developerDisable({ from: user });

        const finalStatus = await instance.developerStatus(user);

        expect(initalStatus).to.be.true;
        expect(finalStatus).to.be.false;
      });

      it("should emit DeveloperDisabled", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(!setupStatus){
          await instance.developerEnable({ from: user });
        }

        truffleAssert.eventEmitted(
          await instance.developerDisable({ from: user }),
          "DeveloperDisabled",
          { accountAddress: user }
        );
      });

      it("should fail if the development account is not enabled", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(setupStatus){
          await instance.developerDisable({ from: user });
        }

        await truffleAssert.fails(instance.developerDisable({ from: user }));
      });
```

When validating the `developerEnable()` function, we will check for the following examples:

1. It should enable developer mode.
2. `DeveloperEnabled` event is emitted when developer mode is enabled.
3. It should revert if developer mode is already enabled.

The section should look like this:

```javascript
      it("should enable development mode", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(setupStatus){
          await instance.developerDisable({ from: user });
        }

        const initalStatus = await instance.developerStatus(user);

        await instance.developerEnable({ from: user });

        const finalStatus = await instance.developerStatus(user);

        expect(initalStatus).to.be.false;
        expect(finalStatus).to.be.true;
      });

      it("should emit DeveloperEnabled event", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(setupStatus){
          await instance.developerDisable({ from: user });
        }

        truffleAssert.eventEmitted(
          await instance.developerEnable({ from: user }),
          "DeveloperEnabled",
          { accountAddress: user }
        );
      });

      it("should revert if the development mode is already enabled", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(!setupStatus){
          await instance.developerEnable({ from: user });
        }

        await truffleAssert.fails(instance.developerEnable({ from: user }));
      });
```

With that, our test is ready to be run.

<details>

<summary>Your test/EVM.js should look like this:</summary>

```javascript
const PrecompiledEVM = artifacts.require("@acala-network/contracts/build/contracts/EVM");
const PrecompiledToken = artifacts.require("@acala-network/contracts/build/contracts/Token");

const truffleAssert = require("truffle-assertions");

const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

/*
* uncomment accounts to access the test accounts made available by the
* Ethereum client
* See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
*/
contract("PrecompiledEVM", function (accounts) {
  let instance;
  let contract;
  let deployer;
  let user;

  beforeEach("setup development environment", async function () {
    [deployer, user] = accounts;
    instance = await PrecompiledEVM.at(EVM);
    contract = await PrecompiledToken.new();
  });

  describe("Operation", function () {
    describe("newContractExtraBytes()", function () {
      it("should return the new contract extra bytes", async function () {
        const response = await instance.newContractExtraBytes();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
    });

    describe("storageDepositPerByte()", function () {
      it("should return the storage deposit", async function () {
        const response = await instance.storageDepositPerByte();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
    });

    describe("maintainerOf()", function () {
      it("should return the developer deposit", async function () {
        const response = await instance.developerDeposit();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
    });

    describe("developerDeposit()", function () {
      it("should return the developer deposit", async function () {
        const response = await instance.developerDeposit();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
    });

    describe("publicationFee()", function () {
      it("should return the publication fee", async function () {
        const response = await instance.publicationFee();

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
    });

    describe("transferMaintainter()", function () {
      it("should transfer the maintainer of the contract", async function () {
        const initialOwner = await instance.maintainerOf(contract.address);

        await instance.transferMaintainer(contract.address, user, { from: deployer });

        const finalOwner = await instance.maintainerOf(contract.address);

        expect(initialOwner).to.equal(deployer);
        expect(finalOwner).to.equal(user);
      });

      it("should emit TransferredMaintainer when maintainer role of the contract is transferred", async function () {
        const maintainer = await instance.maintainerOf(contract.address);

        // Maintainer needs to be set to deployer in case any settings from the previous examples
        // remain. This is because Truffle only executes the beforeEach action before each of the
        // first-level nested describe blocks, but not the second ones.
        if(maintainer == user){
          await instance.transferMaintainer(contract.address, deployer, { from: user });
        }

        truffleAssert.eventEmitted(
          await instance.transferMaintainer(contract.address, user, { from: deployer }),
          "TransferredMaintainer",
          { contractAddress: contract.address, newMaintainer: user }
        );
      });

      it("should revert if the caller is not the maintainer of the contract", async function () {
        const maintainer = await instance.maintainerOf(contract.address);

        // Maintainer needs to be set to deployer in case any settings from the previous examples
        // remain. This is because Truffle only executes the beforeEach action before each of the
        // first-level nested describe blocks, but not the second ones.
        if(maintainer == user){
          await instance.transferMaintainer(contract.address, deployer, { from: user });
        }

        await truffleAssert.reverts(
          instance.transferMaintainer(contract.address, deployer, { from: user })
        );
      });

      it("should revert if trying to transfer maintainer of 0x0", async function () {
        await truffleAssert.reverts(
          instance.transferMaintainer(NULL_ADDRESS, user, { from: deployer }),
          "EVM: the contractAddress is the zero address"
        );
      });

      it("should revert when trying to transfer maintainer to 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.transferMaintainer(contract.address, NULL_ADDRESS, { from: deployer }),
          "EVM: the newMaintainer is the zero address"
        );
      });
    });

    describe("publishContract()", function () {
      it("should fail when caller is not the maintainer of the contract", async function () {
        await truffleAssert.fails(instance.publishContract(contract, { from: user }));
      });

      it("should revert when trying to publish 0x0 contract", async function () {
        await truffleAssert.reverts(
          instance.publishContract(NULL_ADDRESS, { from: deployer }),
          "EVM: the contractAddress is the zero address"
        );
      });

      it("should emit ContractPublished event", async function () {
        truffleAssert.eventEmitted(
          await instance.publishContract(contract.address, { from: deployer }),
          "ContractPublished",
          { contractAddress: contract.address }
        );
      });
    });

    describe("developerStatus()", function () {
      it("should return the status of the development account", async function () {
        const randomAddress = "0xabcabcabcabcabcabcabcabcabcabcabcabcabca";

        const responseTrue = await instance.developerStatus(deployer);
        const responseFalse = await instance.developerStatus(randomAddress);

        expect(responseTrue).to.be.true;
        expect(responseFalse).to.be.false;
      });
    });

    describe("developerDisable()", function () {
      it("should disable development mode", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(!setupStatus){
          await instance.developerEnable({ from: user });
        }

        const initalStatus = await instance.developerStatus(user);

        await instance.developerDisable({ from: user });

        const finalStatus = await instance.developerStatus(user);

        expect(initalStatus).to.be.true;
        expect(finalStatus).to.be.false;
      });

      it("should emit DeveloperDisabled", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(!setupStatus){
          await instance.developerEnable({ from: user });
        }

        truffleAssert.eventEmitted(
          await instance.developerDisable({ from: user }),
          "DeveloperDisabled",
          { accountAddress: user }
        );
      });

      it("should fail if the development account is not enabled", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(setupStatus){
          await instance.developerDisable({ from: user });
        }

        await truffleAssert.fails(instance.developerDisable({ from: user }));
      });
    });

    describe("developerEnable()", function () {
      it("should enable development mode", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(setupStatus){
          await instance.developerDisable({ from: user });
        }

        const initalStatus = await instance.developerStatus(user);

        await instance.developerEnable({ from: user });

        const finalStatus = await instance.developerStatus(user);

        expect(initalStatus).to.be.false;
        expect(finalStatus).to.be.true;
      });

      it("should emit DeveloperEnabled event", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(setupStatus){
          await instance.developerDisable({ from: user });
        }

        truffleAssert.eventEmitted(
          await instance.developerEnable({ from: user }),
          "DeveloperEnabled",
          { accountAddress: user }
        );
      });

      it("should revert if the development mode is already enabled", async function () {
        const setupStatus = await instance.developerStatus(user);

        if(!setupStatus){
          await instance.developerEnable({ from: user });
        }

        await truffleAssert.fails(instance.developerEnable({ from: user }));
      });
    });
  });
});
```

</details>

When you run the test with (for example) `yarn test-mandala`, your tests should pass with the following output:

```shell
yarn test-mandala


yarn run v1.22.17
$ truffle test --network mandala
Using network 'mandala'.


Compiling your contracts...
===========================
> Compiling ./../DEX/contracts/PrecompiledDEX.sol
> Artifacts written to /var/folders/_c/x274s0_x6qj1xtkv60pllc800000gp/T/test--23514-geqKNhJDF16E
> Compiled successfully using:
   - solc: 0.8.9+commit.e5eed63a.Emscripten.clang


  Contract: PrecompiledEVM
    Operation
      newContractExtraBytes()
        ✓ should return the new contract extra bytes
      storageDepositPerByte()
        ✓ should return the storage deposit
      maintainerOf()
        ✓ should return the developer deposit
      developerDeposit()
        ✓ should return the developer deposit
      publicationFee()
        ✓ should return the publication fee
      transferMaintainter()
        ✓ should transfer the maintainer of the contract (1204ms)
        ✓ should emit TransferredMaintainer when maintainer role of the contract is transferred (1186ms)
        ✓ should revert if the caller is not the maintainer of the contract (88ms)
        ✓ should revert if trying to transfer maintainer of 0x0 (61ms)
        ✓ should revert when trying to transfer maintainer to 0x0 address (66ms)
      publishContract()
        ✓ should fail when caller is not the maintainer of the contract (248ms)
        ✓ should revert when trying to publish 0x0 contract (69ms)
        ✓ should emit ContractPublished event (1162ms)
      developerStatus()
        ✓ should return the status of the development account
      developerDisable()
        ✓ should disable development mode (1217ms)
        ✓ should emit DeveloperDisabled (2544ms)
        ✓ should fail if the development account is not enabled (115ms)
      developerEnable()
        ✓ should enable development mode (1301ms)
        ✓ should emit DeveloperEnabled event (2361ms)
        ✓ should revert if the development mode is already enabled (98ms)


  20 passing (39s)

✨  Done in 45.67s.
```

## User journey

Since there is no contract to deploy, let's add a simulation of a user interacting with the `EVM` and log all of the changes and information ot the console. The script will be called `userJourney.js` and will reside in the `scripts/` folder:

```shell
touch scripts/userJourney.js
```

The empty user journey script together with the imports of `EVM` from `@acala-network/contracts` and precompiled `EVM` and `Token` smart contracts from `@acala-network/contracts` should look like this:

```javascript
const EVMContract = artifacts.require("@acala-network/contracts/build/contracts/EVM");
const TokenContract = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");

module.exports = async function(callback) {
  try {
    
  }
  catch(error) {
    console.log(error)
  }

  callback()
}
```

We will pad the log to the console with empty strings in order to get a more verbose output. At the beginning of the script, we assign an address values to `deployer` and `user` and instantiate the predeployed `EVM` smart contract with the help of the `ADDRESS` utility:

```javascript
    console.log("");
    console.log("");

    const accounts = await web3.eth.getAccounts();
    const deployer = accounts[0];
    const user = accounts[1];
    
    console.log(`Interacting with EVM using accounts ${deployer} and ${user}`);

    console.log("");
    console.log("");

    console.log("Instantiating DEX and token smart contracts");

    const instance = await EVMContract.at(EVM);

    console.log("EVM instantiated with address", instance.address);
```

Now that we have instatiated the smart contract, we have to prepare the accounts for the journey. As both start the journey as non-developer accounts, we need to make sure that is the case (and disable the development mode if necessary):

```javascript
    console.log("");
    console.log("");

    console.log("Preparing addresses for the journey");

    const initalDeployerStatus = await instance.developerStatus(deployer);
    const initialUserStatus = await instance.developerStatus(user);

    if(initalDeployerStatus){
      await instance.developerDisable({ from: deployer });
    }

    if(initialUserStatus){
      await instance.developerDisable({ from: user });
    }
```

As the development mode si disabled on our `deployer` account, we need to enable it. After it is enabled, we can query and output the development status of both of the Signers:

```javascript
    console.log("");
    console.log("");

    console.log("Enabling development mode on deployer address");

    await instance.developerEnable({ from: deployer });

    const midwayDeployerStatus = await instance.developerStatus(deployer);
    const midwayUserStatus = await instance.developerStatus(user);

    console.log(`The developer status of ${deployer} in ${midwayDeployerStatus}.`);
    console.log(`The developer status of ${user} in ${midwayUserStatus}.`);
```

Now that the `deployer` has developer mode enabled, we can use it to deploy the journey smart contract. We can now check for the maintainer of the newly deployed smart contract and output the result to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Deploying a smart contract");

    const contract = await TokenContract.new();

    const deployMaintainer = await instance.maintainerOf(contract.address);

    console.log(`Contract deployed at ${contract.address} has a maintainer ${deployMaintainer}`);
```

Before the account, that doesn't have the developer mode enabled can interact with the smart contract, the smart contract has to be published. When publishing a smart contract, the maintainer has to pay the publication fee. Let's query it, publish the smart contract and log the publication fee to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Publishing the contract");

    const fee = await instance.publicationFee();

    await instance.publishContract(contract.address, { from: deployer });

    console.log(`Publication fee is ${fee}`);
    console.log("Contract is sucessfuly published!");
```

If we wish to transfer maintainer of the smart contract to another address, we need to make sure that it has the development mode enabled. Since we will be transferring the maintainer to `user`, we have to enable the development mode on it. Once we do, we can output the development mode status of both accounts to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Enabling developer mode on the user address");

    await instance.developerEnable({ from: user });

    const finalDeployerStatus = await instance.developerStatus(deployer);
    const finalUserStatus = await instance.developerStatus(user);

    console.log(`The developer status of ${deployer} in ${finalDeployerStatus}.`);
    console.log(`The developer status of ${user} in ${finalUserStatus}.`);
```

Now that the `user` Signer is ready to become the maintainer of the journey smart contract, we can transfer the maintainer and log the result to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Transferring maintainer of the contract to the user address");

    const initialMaintainer = await instance.maintainerOf(contract.address);

    await instance.transferMaintainer(contract.address, user, { from: deployer });

    const finalMaintainer = await instance.maintainerOf(contract.address);

    console.log(`Maintainer of the contract at ${contract.address} was transferred from ${initialMaintainer} to ${finalMaintainer}.`);
```

Finally our user journey is completed and all that is left to do is to add this information to the console:

```javascript
    console.log("");
    console.log("");

    console.log("User journey completed!");
```

This concludes our script.

<details>

<summary>Your script/userJourney.js should look like this:</summary>

```
const EVMContract = artifacts.require("@acala-network/contracts/build/contracts/EVM");
const TokenContract = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");

module.exports = async function(callback) {
  try {
    console.log("");
    console.log("");

    const accounts = await web3.eth.getAccounts();
    const deployer = accounts[0];
    const user = accounts[1];
    
    console.log(`Interacting with EVM using accounts ${deployer} and ${user}`);

    console.log("");
    console.log("");

    console.log("Instantiating DEX and token smart contracts");

    const instance = await EVMContract.at(EVM);

    console.log("EVM instantiated with address", instance.address);

    console.log("");
    console.log("");

    console.log("Preparing addresses for the journey");

    const initalDeployerStatus = await instance.developerStatus(deployer);
    const initialUserStatus = await instance.developerStatus(user);

    if(initalDeployerStatus){
      await instance.developerDisable({ from: deployer });
    }

    if(initialUserStatus){
      await instance.developerDisable({ from: user });
    }

    console.log("");
    console.log("");

    console.log("Enabling development mode on deployer address");

    await instance.developerEnable({ from: deployer });

    const midwayDeployerStatus = await instance.developerStatus(deployer);
    const midwayUserStatus = await instance.developerStatus(user);

    console.log(`The developer status of ${deployer} in ${midwayDeployerStatus}.`);
    console.log(`The developer status of ${user} in ${midwayUserStatus}.`);

    console.log("");
    console.log("");

    console.log("Deploying a smart contract");

    const contract = await TokenContract.new();

    const deployMaintainer = await instance.maintainerOf(contract.address);

    console.log(`Contract deployed at ${contract.address} has a maintainer ${deployMaintainer}`);

    console.log("");
    console.log("");

    console.log("Publishing the contract");

    const fee = await instance.publicationFee();

    await instance.publishContract(contract.address, { from: deployer });

    console.log(`Publication fee is ${fee}`);
    console.log("Contract is sucessfuly published!");

    console.log("");
    console.log("");

    console.log("Enabling developer mode on the user address");

    await instance.developerEnable({ from: user });

    const finalDeployerStatus = await instance.developerStatus(deployer);
    const finalUserStatus = await instance.developerStatus(user);

    console.log(`The developer status of ${deployer} in ${finalDeployerStatus}.`);
    console.log(`The developer status of ${user} in ${finalUserStatus}.`);

    console.log("");
    console.log("");

    console.log("Transferring maintainer of the contract to the user address");

    const initialMaintainer = await instance.maintainerOf(contract.address);

    await instance.transferMaintainer(contract.address, user, { from: deployer });

    const finalMaintainer = await instance.maintainerOf(contract.address);

    console.log(`Maintainer of the contract at ${contract.address} was transferred from ${initialMaintainer} to ${finalMaintainer}.`);

    console.log("");
    console.log("");

    console.log("User journey completed!");
  }
  catch(error) {
    console.log(error)
  }

  callback()
}
```

</details>

Running the `yarn user-journey-mandala` script should return the following output:

```shell
yarn user-journey-mandala


yarn run v1.22.17
$ truffle exec scripts/userJourney.js --network mandala
Using network 'mandala'.



Interacting with EVM using accounts 0x75E480dB528101a381Ce68544611C169Ad7EB342 and 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6


Instantiating DEX and token smart contracts
EVM instantiated with address 0x0000000000000000000000000000000000000800


Preparing addresses for the journey


Enabling development mode on deployer address
The developer status of 0x75E480dB528101a381Ce68544611C169Ad7EB342 in true.
The developer status of 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6 in false.


Deploying a smart contract
Contract deployed at 0x8aF989E00782BcB8D837590b88c2968454E7b6aa has a maintainer 0x75E480dB528101a381Ce68544611C169Ad7EB342


Publishing the contract
Publication fee is 1000000000000000000
Contract is sucessfuly published!


Enabling developer mode on the user address
The developer status of 0x75E480dB528101a381Ce68544611C169Ad7EB342 in true.
The developer status of 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6 in true.


Transferring maintainer of the contract to the user address
Maintainer of the contract at 0x8aF989E00782BcB8D837590b88c2968454E7b6aa was transferred from 0x75E480dB528101a381Ce68544611C169Ad7EB342 to 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6.


User journey completed!
✨  Done in 10.90s.
```

## Summary

We have built upon the knowledge on how to interact with the Acala EVM+ and gotten familiar with Acala EVM+ precompiles and predeploys. To run the test we can use the `yarn test-mandala` and `yarn test-madala:pubDev` and to run the user journey script we can use the `yarn user-journey-mandala` and `yarn user-journey-mandala:pubDev`. As we are using utilities only available in the Acala EVM+, we can no longer use a conventional development network like Ganache.
