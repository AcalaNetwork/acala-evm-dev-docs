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

This example introduces the use of Acala EVM+ predeployed EVM that is present on every network at a fixed address (the address of a predeployed contract is the same on a local development network, public test network as well as the production network). As this example focuses on showcasing the interactions with the predeployed `EVM`, it doesn't have its own smart contract. We will get all of the required imports from the [`@acala-network/contracts`](https://github.com/AcalaNetwork/predeploy-contracts) dependency. The precompiles and predeploys are a specific feature of the Acala EVM+, so this tutorial is no longer compatible with traditional EVM development networks (like Ganache) or with the Hardhat's built in network emulator.

Let's take a look!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/EVM](https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/EVM)
{% endhint %}

## Smart contract

As mentioned in the introduction, this tutorial doesn't include the smart contract, so the `contracts` folder can be removed as well. We will however be using the `@acala-network/contracts` dependency in order to gain access to the precompiled resources of the `EVM` smart contract.

## Test

Tests for this tutorial will validate the expected operation and values returned by the EVM predeployed smart contract. The test file in our case is called `EVM.js`. Within it we import the `expect` from `chai` dependency and `Contract`, `BigNumber` amd `Wallet` from the `ethers` dependency. We are using `Contract` in stead of `ContractFactory`, because the contract is already deployed to the network. The `EVM`, which is an export from the `ADDRESS` utility of `@acala-network/contracts` dependency, is imported and it holds the value of the address of the EVM smart contract. Additionally we are importing the compiled `EVM` smart contract from the `@acala-network/contracts` dependency, which we will use to instantiate the smart contract. `Token` precompile is imported from `@acala-network/contracts` so that we can deploy it and use it in the tests.

The test file with import statements and an empty test should look like this:

```javascript
const { expect } = require("chai");
const { Contract, BigNumber, Wallet } = require("ethers");
const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");

const EVMContract = require("@acala-network/contracts/build/contracts/EVM.json");
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

describe("EVM contract", function () {

});
```

To prepare for the testing, we have to define the global variables, `instance`, `contract`, `deployer`, `user`, `deployerAddress` and `userAddres`. The `instance` will store the predeployed EVM smart contract instance and `contract` will store our test contract instance. The `deployer` and `user` will store `Signer`s and the `deployerAddress` and `userAddress` will store their addresses. Let's assign these values in the `beforeEach` action:

```javascript
        let instance;
        let contract;
        let deployer;
        let user;
        let deployerAddress;
        let userAddress;

        beforeEach(async function () {
                [deployer, user] = await ethers.getSigners();
                deployerAddress = await deployer.getAddress();
                userAddress = await user.getAddress();
                instance = new Contract(EVM, EVMContract.abi, deployer);
                const Token = new ethers.ContractFactory(TokenContract.abi, TokenContract.bytecode, deployer);
                contract = await Token.deploy();
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

Before we add the inner describe blocks within the `Operation` describe block, we should increase the timeout for this test to 50s, to make sure that the tests can be run on the public test network in addition to the local development network:

```javascript
        describe("Operation", function () {
                this.timeout(50000);

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

                                expect(response).to.be.above(BigNumber.from("0"));
                        });
```

When validating the `storageDepositPerByte()` function, we will check for the following examples:

1. Storage deposit per byte is returned.

The section should look like this:

```javascript
                        it("should return the storage deposit", async function () {
                                const response = await instance.storageDepositPerByte();

                                expect(response).to.be.above(BigNumber.from("0"));
                        });
```

When validating the `maintainerOf()` function, we will check for the following examples:

1. Maintainer of the contract should be returned

The section should look like this:

```javascript
                        it("should return the maintainer of the contract", async function () {
                                const owner = await instance.maintainerOf(contract.address);

                                expect(owner).to.equal(deployerAddress);
                        });
```

When validating the `developerDeposit()` function, we will check for the following examples:

1. Value of the developer deposit should be returned.

The section should look like this:

```javascript
                        it("should return the developer deposit", async function () {
                                const response = await instance.developerDeposit();

                                expect(response).to.be.above(BigNumber.from("0"));
                        });
```

When validating the `publicationFee()` function, we will check for the following examples:

1. Publication fee should be returned.

The section should look like this:

```javascript
                        it("should return the publication fee", async function () {
                                const response = await instance.publicationFee();

                                expect(response).to.be.above(BigNumber.from("0"));
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

                                await instance.connect(deployer).transferMaintainer(contract.address, userAddress);

                                const finalOwner = await instance.maintainerOf(contract.address);

                                expect(initialOwner).to.equal(deployerAddress);
                                expect(finalOwner).to.equals(userAddress);
                        });

                        it("should emit TransferredMaintainer when maintainer role of the contract is transferred", async function () {
                                await expect(instance.connect(deployer).transferMaintainer(contract.address, userAddress)).to
                                        .emit(instance, "TransferredMaintainer")
                                        .withArgs(contract.address, userAddress);
                        });

                        it("should revert if the caller is not the maintainer of the contract", async function() {
                                await expect(instance.connect(user).transferMaintainer(contract.address, deployerAddress)).to
                                        .be.reverted;
                        });

                        it("should revert if trying to transfer maintainer of 0x0", async function () {
                                await expect(instance.connect(deployer).transferMaintainer(NULL_ADDRESS, userAddress)).to
                                        .be.revertedWith("EVM: the contractAddress is the zero address");
                        });

                        it("should revert when trying to transfer maintainer to 0x0 address", async function () {
                                await expect(instance.connect(deployer).transferMaintainer(contract.address, NULL_ADDRESS)).to
                                        .be.revertedWith("EVM: the newMaintainer is the zero address");
                        });
```

When validating the `publishContract()` function, we will check for the following examples:

1. `ContractPublished` event should be emitted when the contract is published.
2. It should revert when the caller is not the maintainer of the contract.
3. It should revert when tring to publish `0x0` contract.

The section should look like this:

```javascript
                        it("should emit ContractPublished event", async function () {
                                await expect(instance.connect(deployer).publishContract(contract.address)).to
                                        .emit(instance, "ContractPublished")
                                        .withArgs(contract.address);
                        });

                        it("should revert when caller is not the maintainer of the contract", async function () {
                                await expect(instance.connect(user).publishContract(contract.address)).to
                                        .be.reverted;
                        });

                        it("should revert when trying to publish 0x0 contract", async function () {
                                await expect(instance.connect(deployer).publishContract(NULL_ADDRESS)).to
                                        .be.revertedWith("EVM: the contractAddress is the zero address");
                        });
```

When validating the `developerStatus()` function, we will check for the following examples:

1. It should return the developer mode status of the address.

The section should look like this:

```javascript
                        it("should return the status of the development account", async function () {
                                const randomSigner = new Wallet.createRandom();

                                const responseTrue = await instance.developerStatus(deployerAddress);
                                const responseFalse = await instance.developerStatus(await randomSigner.getAddress());

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
                                const setupStatus = await instance.developerStatus(userAddress);
                                
                                if(!setupStatus){
                                        await instance.connect(user).developerEnable();
                                }

                                const initialStatus = await instance.developerStatus(userAddress);

                                await instance.connect(user).developerDisable();

                                const finalStatus = await instance.developerStatus(userAddress);

                                expect(initialStatus).to.be.true;
                                expect(finalStatus).to.be.false;
                        });

                        it("should emit DeveloperDisabled", async function () {
                                const initialStatus = await instance.developerStatus(userAddress);
                                
                                if(!initialStatus){
                                        await instance.connect(user).developerEnable();
                                }

                                await expect(instance.connect(user).developerDisable()).to
                                        .emit(instance, "DeveloperDisabled")
                                        .withArgs(userAddress);
                        });

                        it("should revert if the development account is not enabled", async function () {
                                const setupStatus = await instance.developerStatus(userAddress);
                                
                                if(setupStatus){
                                        await instance.connect(user).developerDisable();
                                }

                                await expect(instance.connect(user).developerDisable()).to
                                        .be.reverted;
                        });
```

When validating the `developerEnable()` function, we will check for the following examples:

1. It should enable developer mode.
2. `DeveloperEnabled` event is emitted when developer mode is enabled.
3. It should revert if developer mode is already enabled.

The section should look like this:

```javascript
                        it("should enable development mode", async function () {
                                const setupStatus = await instance.developerStatus(userAddress);
                                
                                if(setupStatus){
                                        await instance.connect(user).developerDisable();
                                }

                                const initialStatus = await instance.developerStatus(userAddress);

                                await instance.connect(user).developerEnable();

                                const finalStatus = await instance.developerStatus(userAddress);

                                expect(initialStatus).to.be.false;
                                expect(finalStatus).to.be.true;
                        });

                        it("should emit DeveloperEnabled event", async function () {
                                const setupStatus = await instance.developerStatus(userAddress);
                                
                                if(setupStatus){
                                        await instance.connect(user).developerDisable();
                                }

                                await expect(instance.connect(user).developerEnable()).to
                                        .emit(instance, "DeveloperEnabled")
                                        .withArgs(userAddress);
                        });

                        it("should revert if the development mode is already enabled", async function () {
                                const setupStatus = await instance.developerStatus(userAddress);
                                
                                if(!setupStatus){
                                        await instance.connect(user).developerEnable();
                                }

                                await expect(instance.connect(user).developerEnable()).to
                                        .be.reverted;
                        });
```

With that, our test is ready to be run.

<details>

<summary>Your test/EVM.js should look like this:</summary>

```javascript
    const { expect } = require("chai");
    const { Contract, BigNumber, Wallet } = require("ethers");
    const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");

    const EVMContract = require("@acala-network/contracts/build/contracts/EVM.json");
    const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");
    const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

    describe("EVM contract", function () {
            let instance;
            let contract;
            let deployer;
            let user;
            let deployerAddress;
            let userAddress;

            beforeEach(async function () {
                    [deployer, user] = await ethers.getSigners();
                    deployerAddress = await deployer.getAddress();
                    userAddress = await user.getAddress();
                    instance = new Contract(EVM, EVMContract.abi, deployer);
                    const Token = new ethers.ContractFactory(TokenContract.abi, TokenContract.bytecode, deployer);
                    contract = await Token.deploy();
            });

            describe("Operation", function () {
                    this.timeout(50000);

                    describe("newContractExtraBytes()", function () {
                            it("should return the new contract extra bytes", async function () {
                                    const response = await instance.newContractExtraBytes();

                                    expect(response).to.be.above(BigNumber.from("0"));
                            });
                    });

                    describe("storageDepositPerByte()", function () {
                            it("should return the storage deposit", async function () {
                                    const response = await instance.storageDepositPerByte();

                                    expect(response).to.be.above(BigNumber.from("0"));
                            });
                    });

                    describe("maintainerOf()", function () {
                            it("should return the maintainer of the contract", async function () {
                                    const owner = await instance.maintainerOf(contract.address);

                                    expect(owner).to.equal(deployerAddress);
                            });
                    });

                    describe("developerDeposit()", function () {
                            it("should return the developer deposit", async function () {
                                    const response = await instance.developerDeposit();

                                    expect(response).to.be.above(BigNumber.from("0"));
                            });
                    });

                    describe("publicationFee()", function () {
                            it("should return the publication fee", async function () {
                                    const response = await instance.publicationFee();

                                    expect(response).to.be.above(BigNumber.from("0"));
                            });
                    });

                    describe("transferMaintainter()", function () {
                            it("should transfer the maintainer of the contract", async function () {
                                    const initialOwner = await instance.maintainerOf(contract.address);

                                    await instance.connect(deployer).transferMaintainer(contract.address, userAddress);

                                    const finalOwner = await instance.maintainerOf(contract.address);

                                    expect(initialOwner).to.equal(deployerAddress);
                                    expect(finalOwner).to.equals(userAddress);
                            });

                            it("should emit TransferredMaintainer when maintainer role of the contract is transferred", async function () {
                                    await expect(instance.connect(deployer).transferMaintainer(contract.address, userAddress)).to
                                            .emit(instance, "TransferredMaintainer")
                                            .withArgs(contract.address, userAddress);
                            });

                            it("should revert if the caller is not the maintainer of the contract", async function() {
                                    await expect(instance.connect(user).transferMaintainer(contract.address, deployerAddress)).to
                                            .be.reverted;
                            });

                            it("should revert if trying to transfer maintainer of 0x0", async function () {
                                    await expect(instance.connect(deployer).transferMaintainer(NULL_ADDRESS, userAddress)).to
                                            .be.revertedWith("EVM: the contractAddress is the zero address");
                            });

                            it("should revert when trying to transfer maintainer to 0x0 address", async function () {
                                    await expect(instance.connect(deployer).transferMaintainer(contract.address, NULL_ADDRESS)).to
                                            .be.revertedWith("EVM: the newMaintainer is the zero address");
                            });
                    });

                    describe("publishContract()", function () {
                            it("should emit ContractPublished event", async function () {
                                    await expect(instance.connect(deployer).publishContract(contract.address)).to
                                            .emit(instance, "ContractPublished")
                                            .withArgs(contract.address);
                            });

                            it("should revert when caller is not the maintainer of the contract", async function () {
                                    await expect(instance.connect(user).publishContract(contract.address)).to
                                            .be.reverted;
                            });

                            it("should revert when trying to publish 0x0 contract", async function () {
                                    await expect(instance.connect(deployer).publishContract(NULL_ADDRESS)).to
                                            .be.revertedWith("EVM: the contractAddress is the zero address");
                            });
                    });

                    describe("developerStatus()", function () {
                            it("should return the status of the development account", async function () {
                                    const randomSigner = new Wallet.createRandom();

                                    const responseTrue = await instance.developerStatus(deployerAddress);
                                    const responseFalse = await instance.developerStatus(await randomSigner.getAddress());

                                    expect(responseTrue).to.be.true;
                                    expect(responseFalse).to.be.false;
                            });
                    });

                    describe("developerDisable()", function () {
                            it("should disable development mode", async function () {
                                    const setupStatus = await instance.developerStatus(userAddress);
                                    
                                    if(!setupStatus){
                                            await instance.connect(user).developerEnable();
                                    }

                                    const initialStatus = await instance.developerStatus(userAddress);

                                    await instance.connect(user).developerDisable();

                                    const finalStatus = await instance.developerStatus(userAddress);

                                    expect(initialStatus).to.be.true;
                                    expect(finalStatus).to.be.false;
                            });

                            it("should emit DeveloperDisabled", async function () {
                                    const initialStatus = await instance.developerStatus(userAddress);
                                    
                                    if(!initialStatus){
                                            await instance.connect(user).developerEnable();
                                    }

                                    await expect(instance.connect(user).developerDisable()).to
                                            .emit(instance, "DeveloperDisabled")
                                            .withArgs(userAddress);
                            });

                            it("should revert if the development account is not enabled", async function () {
                                    const setupStatus = await instance.developerStatus(userAddress);
                                    
                                    if(setupStatus){
                                            await instance.connect(user).developerDisable();
                                    }

                                    await expect(instance.connect(user).developerDisable()).to
                                            .be.reverted;
                            });
                    });

                    describe("developerEnable()", function () {
                            it("should enable development mode", async function () {
                                    const setupStatus = await instance.developerStatus(userAddress);
                                    
                                    if(setupStatus){
                                            await instance.connect(user).developerDisable();
                                    }

                                    const initialStatus = await instance.developerStatus(userAddress);

                                    await instance.connect(user).developerEnable();

                                    const finalStatus = await instance.developerStatus(userAddress);

                                    expect(initialStatus).to.be.false;
                                    expect(finalStatus).to.be.true;
                            });

                            it("should emit DeveloperEnabled event", async function () {
                                    const setupStatus = await instance.developerStatus(userAddress);
                                    
                                    if(setupStatus){
                                            await instance.connect(user).developerDisable();
                                    }

                                    await expect(instance.connect(user).developerEnable()).to
                                            .emit(instance, "DeveloperEnabled")
                                            .withArgs(userAddress);
                            });

                            it("should revert if the development mode is already enabled", async function () {
                                    const setupStatus = await instance.developerStatus(userAddress);
                                    
                                    if(!setupStatus){
                                            await instance.connect(user).developerEnable();
                                    }

                                    await expect(instance.connect(user).developerEnable()).to
                                            .be.reverted;
                            });
                    });
            });
    });
```

</details>

When you run the test with (for example) `yarn test-mandala`, your tests should pass with the following output:

```shell
yarn test-mandala


yarn run v1.22.19
$ hardhat test test/EVM.js --network mandala


  EVM contract
    Operation
      newContractExtraBytes()
        ✔ should return the new contract extra bytes
      storageDepositPerByte()
        ✔ should return the storage deposit
      maintainerOf()
        ✔ should return the maintainer of the contract
      developerDeposit()
        ✔ should return the developer deposit
      publicationFee()
        ✔ should return the publication fee
      transferMaintainter()
        ✔ should transfer the maintainer of the contract (2234ms)
        ✔ should emit TransferredMaintainer when maintainer role of the contract is transferred (3235ms)
        ✔ should revert if the caller is not the maintainer of the contract
        ✔ should revert if trying to transfer maintainer of 0x0
        ✔ should revert when trying to transfer maintainer to 0x0 address
      publishContract()
        ✔ should emit ContractPublished event (3263ms)
        ✔ should revert when caller is not the maintainer of the contract
        ✔ should revert when trying to publish 0x0 contract
      developerStatus()
        ✔ should return the status of the development account (83ms)
      developerDisable()
        ✔ should disable development mode (2227ms)
        ✔ should emit DeveloperDisabled (5425ms)
        ✔ should revert if the development account is not enabled (39ms)
      developerEnable()
        ✔ should enable development mode (2235ms)
        ✔ should emit DeveloperEnabled event (5471ms)
        ✔ should revert if the development mode is already enabled (39ms)


  20 passing (1m)

✨  Done in 69.30s.
```

## User journey

Since there is no contract to deploy, let's add a simulation of a user interacting with a `EVM` and log all of the changes and information to the console. The script will be called `userJourney.js` and will reside in the `scripts/` folder:

```shell
touch scripts/userJourney.js
```

The empty user journey script together with the imports of `EVM` from `@acala-network/contracts`, `Contract` from `ethers`, `txParams` from `transactionHelper` utility and precompiled `EVM` and `Token` smart contracts from `@acala-network/contracts` should look like this:

```javascript
const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");
const { Contract } = require("ethers");

const { txParams } = require("../utils/transactionHelper");

const EVMContract = require("@acala-network/contracts/build/contracts/EVM.json");
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");

async function main () {
        
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

We will pad the log to the console with empty strings in order to get a more verbose output. At the beginning of the script, we assign a Signer value to `deployer` and `user`, get the Signer's addresses and instantiate the predeployed `EVM` smart contract with the help of the `ADDRESS` utility:

```javascript
    console.log("");
    console.log("");

    const [deployer, user] = await ethers.getSigners();
    const deployerAddress = await deployer.getAddress();
    const userAddress = await user.getAddress();

    console.log(`Interacting with EVM using accounts ${deployerAddress} and ${userAddress}`);

    console.log("");
    console.log("");

    console.log("Instantiating EVM smart contract");

    const instance = new Contract(EVM, EVMContract.abi, deployer);

    console.log("EVM instantiated with address", instance.address);
```

Now that we have instantiated the smart contract, we have to prepare the Signers for the journey. As both start the journey as non-developer accounts, we need to make sure that is the case (and disable the development mode if necessary):

```javascript
    console.log("");
    console.log("");

    console.log("Preparing addresses for the journey");

    const initalDeployerStatus = await instance.developerStatus(deployerAddress);
    const initialUserStatus = await instance.developerStatus(userAddress);

    if(initalDeployerStatus){
      await instance.connect(deployer).developerDisable();
    }

    if(initialUserStatus){
      await instance.connect(user).developerDisable();
    }
```

As the development mode si disabled on our `deployer` Signer, we need to enable it. After it is enabled, we can query and output the development status of both of the Signers:

```javascript
    console.log("");
    console.log("");

    console.log("Enabling development mode on deployer address");

    await instance.connect(deployer).developerEnable();

    const midwayDeployerStatus = await instance.developerStatus(deployerAddress);
    const midwayUserStatus = await instance.developerStatus(userAddress);

    console.log(`The developer status of ${deployerAddress} in ${midwayDeployerStatus}.`);
    console.log(`The developer status of ${userAddress} in ${midwayUserStatus}.`);
```

Now that the `deployer` has developer mode enabled, we can use it to deploy the journey smart contract. Before we are able to deploy it, we need to use the `txParams` from the `transactionHelper` utility in order to calculate the transaction parameters for the deploy transaction. We can now check for the maintainer of the newly deployed smart contract and output the result to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Deploying a smart contract");

    const ethParams = await txParams();
    const Token = new ethers.ContractFactory(TokenContract.abi, TokenContract.bytecode, deployer);
    const contract = await Token.deploy({
            gasPrice: ethParams.txGasPrice,
            gasLimit: ethParams.txGasLimit,
    });

    const deployMaintainer = await instance.maintainerOf(contract.address);

    console.log(`Contract deployed at ${contract.address} has a maintainer ${deployMaintainer}`);
```

Before the account, that doesn't have the developer mode enabled can interact with the smart contract, the smart contract has to be published. When publishing a smart contract, the maintainer has to pay the publication fee. Let's query it, publish the smart contract and log the publication fee to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Publishing the contract");

    const fee = await instance.publicationFee();

    await instance.connect(deployer).publishContract(contract.address);

    console.log(`Publication fee is ${fee}`);
    console.log("Contract is sucessfuly published!");
```

If we wish to transfer maintainer of the smart contract to another address, we need to make sure that it has the development mode enabled. Since we will be transferring the maintainer to `user`, we have to enable the development mode on it. Once we do, we can output the development mode status of both Signers to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Enabling developer mode on the user address");

    await instance.connect(user).developerEnable();

    const finalDeployerStatus = await instance.developerStatus(deployerAddress);
    const finalUserStatus = await instance.developerStatus(userAddress);

    console.log(`The developer status of ${deployerAddress} in ${finalDeployerStatus}.`);
    console.log(`The developer status of ${userAddress} in ${finalUserStatus}.`);
```

Now that the `user` Signer is ready to become the maintainer of the journey smart contract, we can transfer the maintainer and log the result to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Transferring maintainer of the contract to the user address");

    const initialMaintainer = await instance.maintainerOf(contract.address);

    await instance.connect(deployer).transferMaintainer(contract.address, userAddress);

    const finalMaintainer = await instance.maintainerOf(contract.address);

    console.log(`Maintainer of the contract at ${contract.address} was transferred from ${initialMaintainer} to ${finalMaintainer}.`);
```

Finally our user journey is completed and all that is left to do is to add this information to the console:

```javascript
    console.log("");
    console.log("");

    console.log("User journey completed!");
```

<details>

<summary>Your scripts/userJourney.js should look like this:</summary>

```javascript
    const { EVM } = require("@acala-network/contracts/utils/MandalaAddress");
    const { Contract } = require("ethers");

    const { txParams } = require("../utils/transactionHelper");

    const EVMContract = require("@acala-network/contracts/build/contracts/EVM.json");
    const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");

    async function main () {
            console.log("");
            console.log("");

            const [deployer, user] = await ethers.getSigners();
            const deployerAddress = await deployer.getAddress();
            const userAddress = await user.getAddress();

            console.log(`Interacting with EVM using accounts ${deployerAddress} and ${userAddress}`);

            console.log("");
            console.log("");

            console.log("Instantiating EVM smart contract");

            const instance = new Contract(EVM, EVMContract.abi, deployer);

            console.log("EVM instantiated with address", instance.address);

            console.log("");
            console.log("");

            console.log("Preparing addresses for the journey");

            const initalDeployerStatus = await instance.developerStatus(deployerAddress);
            const initialUserStatus = await instance.developerStatus(userAddress);

            if(initalDeployerStatus){
                    await instance.connect(deployer).developerDisable();
            }

            if(initialUserStatus){
                    await instance.connect(user).developerDisable();
            }

            console.log("");
            console.log("");

            console.log("Enabling development mode on deployer address");

            await instance.connect(deployer).developerEnable();

            const midwayDeployerStatus = await instance.developerStatus(deployerAddress);
            const midwayUserStatus = await instance.developerStatus(userAddress);

            console.log(`The developer status of ${deployerAddress} in ${midwayDeployerStatus}.`);
            console.log(`The developer status of ${userAddress} in ${midwayUserStatus}.`);

            console.log("");
            console.log("");

            console.log("Deploying a smart contract");

            const ethParams = await txParams();
            const Token = new ethers.ContractFactory(TokenContract.abi, TokenContract.bytecode, deployer);
            const contract = await Token.deploy({
                    gasPrice: ethParams.txGasPrice,
                    gasLimit: ethParams.txGasLimit,
            });

            const deployMaintainer = await instance.maintainerOf(contract.address);

            console.log(`Contract deployed at ${contract.address} has a maintainer ${deployMaintainer}`);

            console.log("");
            console.log("");

            console.log("Publishing the contract");

            const fee = await instance.publicationFee();

            await instance.connect(deployer).publishContract(contract.address);

            console.log(`Publication fee is ${fee}`);
            console.log("Contract is sucessfuly published!");

            console.log("");
            console.log("");

            console.log("Enabling developer mode on the user address");

            await instance.connect(user).developerEnable();

            const finalDeployerStatus = await instance.developerStatus(deployerAddress);
            const finalUserStatus = await instance.developerStatus(userAddress);

            console.log(`The developer status of ${deployerAddress} in ${finalDeployerStatus}.`);
            console.log(`The developer status of ${userAddress} in ${finalUserStatus}.`);

            console.log("");
            console.log("");

            console.log("Transferring maintainer of the contract to the user address");

            const initialMaintainer = await instance.maintainerOf(contract.address);

            await instance.connect(deployer).transferMaintainer(contract.address, userAddress);

            const finalMaintainer = await instance.maintainerOf(contract.address);

            console.log(`Maintainer of the contract at ${contract.address} was transferred from ${initialMaintainer} to ${finalMaintainer}.`);

            console.log("");
            console.log("");

            console.log("User journey completed!");
    }

    main()
            .then(() => process.exit(0))
            .catch((error) => {
                    console.error(error);
                    process.exit(1);
            });
```

</details>

To use the script within the local development network or a public development network, you need to add the following scripts to `scripts` section of your `package.json`:

```json
    "user-journey-mandala": "hardhat run scripts/userJourney.js --network mandala",
    "user-journey-mandala:pubDev": "hardhat run scripts/userJourney.js --network mandalaPubDev"
```

Running the `yarn user-journey-mandala` script should return the following output:

```shell
yarn user-journey-mandala


yarn run v1.22.17
$ hardhat run scripts/userJourney.js --network mandala


Interacting with EVM using accounts 0x75E480dB528101a381Ce68544611C169Ad7EB342 and 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6


Instantiating EVM smart contract
EVM instantiated with address 0x0000000000000000000000000000000000000800


Preparing addresses for the journey


Enabling development mode on deployer address
The developer status of 0x75E480dB528101a381Ce68544611C169Ad7EB342 in true.
The developer status of 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6 in false.


Deploying a smart contract
Contract deployed at 0x59368c7F0898B1e569c9058b6247a84f90692B47 has a maintainer 0x75E480dB528101a381Ce68544611C169Ad7EB342


Publishing the contract
Publication fee is 1000000000000000000
Contract is sucessfuly published!


Enabling developer mode on the user address
The developer status of 0x75E480dB528101a381Ce68544611C169Ad7EB342 in true.
The developer status of 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6 in true.


Transferring maintainer of the contract to the user address
Maintainer of the contract at 0x59368c7F0898B1e569c9058b6247a84f90692B47 was transferred from 0x75E480dB528101a381Ce68544611C169Ad7EB342 to 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6.


User journey completed!
✨  Done in 10.91s.
```

## Summary

We have built upon the knowledge on how to interact with the Acala EVM+ and gotten familiar with Acala EVM+ precompiles and predeploys. To run the test we can use the `yarn test-mandala` and `yarn test-madala:pubDev` and to run the user journey script we can use the `yarn user-journey-mandala` and `yarn user-journey-mandala:pubDev`. As we are using utilities only available in the Acala EVM+, we can no longer use a conventional development network like Ganache or Hardhat's emulated network.
