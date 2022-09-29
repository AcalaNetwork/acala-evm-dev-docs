---
description: A tutorial on how to build an ERC-20 compatible smart contract in Acala EVM+.
---

# Token tutorial

### Table of contents

* [About](token-tutorial.md#about)
* [Smart contract](token-tutorial.md#smart-contract)
* [Test](token-tutorial.md#test)
* [Deploy script](token-tutorial.md#deploy-script)
* [Summary](token-tutorial.md#summary)

### About

This is an example that builds upon the [echo example](echo-tutorial.md). `Echo` was a simple example on building an intreactable state changing smart contract. `Token` is an example of [ERC20](https://eips.ethereum.org/EIPS/eip-20) token implementation in Acala EVM+. We won't be building an administrated or upgradeable token and we will use OpenZeppelin [ERC20 implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol). For the setup and naming, replace the `echo` with `token`. Let's jump into it!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/token](https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/token)
{% endhint %}

### Smart contract

In this tutorial we will be adding a simple smart contract that imports the ERC20 smart contract from `openzeppelin/contracts` and has a constructor that sets the initial balance of the sender (which also represents the total supply of the token) as well as the name of the token and its abbreviation:

To be able to import the `ERC20` from OpenZeppelin, we have to add `@openzeppelin/contracts` as a development dependency:

```shell
yarn add --dev @openzeppelin/contracts
```

Now that we have added the `@openzeppelin/contracts` dependency, we can focus on building our smart contract. Your empty smart contract should look like this:

```solidity
pragma solidity =0.8.9;

contract Token{
   
}
```

Import of the `ERC20` from `@openzeppelin/contracts` is done between the `pragma` definition and the start of the `contract` block:

```solidity
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

As we now have access to `ERC20.sol` from `@openzeppelin/contracts`, we can set the inheritance of our `Token` contract:

```solidity
contract Token is ERC20 {
```

As the `ERC20` already has the full fungible token standard implementation, we only have to add a `constructor()` function that sets all of the values:

```solidity
    constructor(uint256 _initialBalance) ERC20("Token", "TKN") public {
        _mint(msg.sender, _initialBalance);
    }
```

We pass the initial balance of the token as `_intialBalance` and it will be assigned to the account that initiates the deployment of the smart contract. We also pass `Token` as the name of our example token and `TKN` for its abbreviation to the `ERC20` contract that our `Token` contract is inheriting.

This concludes our `Token` smart contract.

<details>

<summary>Your contracts/Echo.sol should look like this:</summary>

```solidity
pragma solidity =0.8.9;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20 {
    constructor(uint256 _initialBalance) ERC20("Token", "TKN") public {
        _mint(msg.sender, _initialBalance);
    }
}
```

</details>

As the Token smart contract is ready to be compiled, we can use the `yarn build` command (like we did in the [hello-world](<../../.gitbook/assets/package (1)>)) to compile the smart contract, which will create the `artifacts` directory and contain the compiled smart contract.

### Test

Your test file should be called `Token.js` and the empty test along with the import statements should look like this:

```javascript
const { expect } = require("chai");
const { ContractFactory } = require("ethers");

const TokenContract = require("../artifacts/contracts/Token.sol/Token.json");
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

describe("Token contract", function () {

});
```

To prepare for the testing, we have to define global variables, `Token`, `instance`, `deployer`, `user`, `deployerAddress` and `userAddress`. The `Token` will be used to store the Token contract factory and the `instance` will store the deployed Token smart contract. Both `deployer` and `user` will store `Signers`. The `deployer` is the account used to deploy the smart contract (and the one that will receive the `initialBalance`). The `user` is the account we will be using to transfer the tokens to and check the allowance operation. `deployerAddress` and `userAddress` hold the addresses of the `deployer` and `user` respectively. They will be used to avoid repetitiveness in our tests. Let's assign them values in the `beforeEach` action:

```javascript
        let Token;
        let instance;
        let deployer;
        let user;
        let deployerAddress;
        let userAddress;

        beforeEach(async function () {
                [deployer, user] = await ethers.getSigners();
                deployerAddress = await deployer.getAddress();
                userAddress = await user.getAddress();
                Token = new ContractFactory(TokenContract.abi, TokenContract.bytecode, deployer);
                instance = await Token.deploy(1234567890);
        });
```

Our test will be split into two sections, `Deployment` and `Operation`:

```javascript
        describe("Deployment", function () {

        });

        describe("Operation", function () {
          
        });
```

`Deployment` block contains the examples that validate the expected initial state of the smart contract:

1. The `name` should equal `Token`.
2. The `symbol` should equal `TKN`.
3. The total supply of the smart contract should equal `1234567890`.
4. The initial balance of the `deployer` account should equal `1234567890`.
5. The `user` account should have `0` balance.
6. The allowances should be set to `0` when the smart contract is deployed.

```javascript
                it("should set the correct token name", async function () {
                        expect(await instance.name()).to.equal("Token");
                });

                it("should set the correct token symbol", async function () {
                        expect(await instance.symbol()).to.equal("TKN");
                });

                it("should set the correct total supply", async function () {
                        expect(await instance.totalSupply()).to.equal(1234567890);
                });

                it("should assign the initial balance to the deployer", async function () {
                        expect(await instance.balanceOf(deployerAddress)).to.equal(1234567890);
                });

                it("should not assign value to a random address upon deployment", async function () {
                        expect(await instance.balanceOf(userAddress)).to.equal(0);
                });

                it("should not assign allowance upond deployment", async function () {
                        expect(await instance.allowance(deployerAddress, userAddress)).to.equal(0);
                        expect(await instance.allowance(userAddress, deployerAddress)).to.equal(0);
                });
```

In the `Operation` describe block we first need to increase the timeout to 50000ms, so that the RPC adapter has enough time to return the required information:

The `Operation` block in itself is separated into two `describe` blocks, which are separated in itself:

1. `Transfer`: Contains the test cases to validate the transaction functionality of ERC20 tokens:

* `transfer()`: Validates the correct operation of the `transfer()` function.

1. `Allowances`: Contains the test cases to validate the allowances and connected functionality of ERC20 tokens:

* `approve()`: Validates the correct operation of the `approve()` function.
* `increaseAllowance()`: Validates the correct operation of the `increaseAllowance()` function.
* `decreaseAllowance()`: Validates the correct operation of the `decreaseAllowance()` function.
* `transferFrom()`: Validates the correct operation of the `transferFrom()` function.

The contents of the `Operation` block should look like this:

```javascript
                describe("Transfer", function () {
                        describe("transfer()", function () {
                                
                        });
                });

                describe("Allowances", function () {
                        describe("approve()", function () {
                                
                        });

                        describe("increaseAllowance()", function () {

                        });

                        describe("decreaseAllowance()", function() {
                                
                        });

                        describe("transferFrom()", function () {
                                
                        });
                });
```

The `transfer()` example validates the following:

1. When transferring tokens, the balances should be updated.
2. `Transfer` event should be emitted.
3. Transfers to `0x0` address should be reverted.
4. Trying to transfer more than own balance should be reverted.

These examples should look like this:

```javascript
                                it("should change the balance of the sender and receiver when transferring token", async function () {
                                        const initialDeployerBalance = await instance.balanceOf(deployerAddress);
                                        const initialUserBalance = await instance.balanceOf(userAddress);

                                        await instance.connect(deployer).transfer(userAddress, 500);

                                        const finalDeployerBalance = await instance.balanceOf(deployerAddress);
                                        const finalUserBalance = await instance.balanceOf(userAddress);

                                        expect(initialDeployerBalance - 500).to.equal(finalDeployerBalance);
                                        expect(initialUserBalance + 500).to.equal(finalUserBalance);
                                });

                                it("should emit a Transfer event when transfering the token", async function () {
                                        await expect(instance.connect(deployer).transfer(userAddress, 100)).to
                                                .emit(instance, "Transfer")
                                                .withArgs(deployerAddress, userAddress, 100);
                                });

                                it("should revert the transfer to a 0x0 address", async function () {
                                        await expect(instance.connect(deployer).transfer(NULL_ADDRESS, 100)).to
                                                .be.revertedWith("ERC20: transfer to the zero address");
                                });

                                it("should revert if trying to transfer amount bigger than balance", async function () {
                                        await expect(instance.connect(deployer).transfer(userAddress, 12345678900)).to
                                                .be.revertedWith("ERC20: transfer amount exceeds balance");
                                });
```

The `approve()` example validates the following:

1. Allowance should be granted for less funds than the owner has.
2. Allowance should be granted for more funds than the owner has.
3. `Approval` event should be emitted when giving allowance.
4. Call should be reverted if the address receiving the allowance is `0x0`.

These examples should look like this:

```javascript
                                it("should grant allowance when the caller has enough funds", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);

                                        expect(await instance.allowance(deployerAddress, userAddress)).to
                                                .equal(100);
                                });


                                it("should grant allowance when the caller has less funds than the ize of the allowance", async function () {
                                        await instance.connect(deployer).approve(userAddress, 12345678900);

                                        expect(await instance.allowance(deployerAddress, userAddress)).to
                                                .equal(12345678900);
                                });

                                it("should emit Approval event when calling approve()", async function () {
                                        await expect(instance.connect(deployer).approve(userAddress, 100)).to
                                                .emit(instance, "Approval")
                                                .withArgs(deployerAddress, userAddress, 100);
                                });

                                it("should revert when trying to give allowance to 0x0 address", async function () {
                                        await expect(instance.connect(deployer).approve(NULL_ADDRESS, 100)).to
                                                .be.revertedWith("ERC20: approve to the zero address");
                                });
```

The `increaseAllowance()` example validates the following:

1. Owner should be able to increase allowance to an amount smaller that the total funds that they have.
2. Owner should be able to increase the balance to more than the amount of total funds that they have.
3. `Approval` event should be emitted.
4. The function can be called even if there was no preexisting allowance.

These examples should look like this:

```javascript
                                it("should allow to increase allowance", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);

                                        const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                                        await instance.connect(deployer).increaseAllowance(userAddress, 50);

                                        const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                                        expect(finalAllowance - initialAllowance).to.equal(50);
                                });

                                it("should allow to increase allowance above the balance", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);

                                        const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                                        await instance.connect(deployer).increaseAllowance(userAddress, 1234567890);

                                        const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                                        expect(finalAllowance - initialAllowance).to.equal(1234567890);
                                });

                                it("should emit Approval event", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);
                                        await expect(instance.connect(deployer).increaseAllowance(userAddress, 40)).to
                                                .emit(instance, "Approval")
                                                .withArgs(deployerAddress, userAddress, 140);
                                });

                                it("should allow to increase allowance even if none was given before", async function () {
                                        await instance.connect(deployer).increaseAllowance(userAddress, 1234567890);

                                        const allowance = await instance.allowance(deployerAddress, userAddress);

                                        expect(allowance).to.equal(1234567890);
                                });
```

The `decreaseAllowance()` example validates the following:

1. Owner should be able to decrease allowance.
2. `Approval` event should be emitted.
3. Call should be reverted when trying to decrease the allowance below 0.

These examples should look like this:

We can now add the following test cases to our describe block:

```javascript
                                it("should decrease the allowance", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);
                                        await instance.connect(deployer).decreaseAllowance(userAddress, 40);

                                        const allowance = await instance.allowance(deployerAddress, userAddress);

                                        expect(allowance).to.equal(60);
                                });

                                it("should emit Approval event", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);
                                        await expect(instance.connect(deployer).decreaseAllowance(userAddress, 40)).to
                                                .emit(instance, "Approval")
                                                .withArgs(deployerAddress, userAddress, 60);
                                });

                                it("should revert when tyring to decrease the allowance below 0", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);
                                        await expect(instance.connect(deployer).decreaseAllowance(userAddress, 1000)).to
                                                .be.revertedWith("ERC20: decreased allowance below zero");
                                });
```

The `transferFrom()` example validates the following:

1. Should allow transfer when allowance is given.
2. `Transfer` event should be emitted.
3. `Approval` event should be emitted.
4. Should update allowance.
5. Should revert if trying to transfer more than allowance.
6. Should revert when trying to transfer to `0x0` address.
7. Should revert when owner doesn't have enough funds.
8. Should revert when no allowance was given.

These examples should look like this:

```javascript
                                it("should allow to transfer tokens when allowance is given", async function () {

                                        await instance.connect(deployer).approve(userAddress, 100);

                                        const initialBalance = await instance.balanceOf(userAddress);

                                        await instance.connect(user).transferFrom(deployerAddress, userAddress, 50);

                                        const finalBalance = await instance.balanceOf(userAddress);

                                        expect(initialBalance + 50).to.equal(finalBalance);
                                });

                                it("should emit Transfer event when transferring from another address", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);

                                        await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 40)).to
                                                .emit(instance, "Transfer")
                                                .withArgs(deployerAddress, userAddress, 40);
                                });

                                it("should emit Approval event when transferring from another address", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);

                                        await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 40)).to
                                                .emit(instance, "Approval")
                                                .withArgs(deployerAddress, userAddress, 60);
                                });

                                it("should update the allowance when transferring from another address", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);
                                        await instance.connect(user).transferFrom(deployerAddress, userAddress, 40);

                                        expect(await instance.allowance(deployerAddress, userAddress)).to.equal(60);
                                });

                                it("should revert when tring to transfer more than allowed amount", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);
                                        await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 1000)).to
                                                .be.revertedWith("ERC20: transfer amount exceeds allowance");
                                });

                                it("should revert when transfering to 0x0 address", async function () {
                                        await instance.connect(deployer).approve(userAddress, 100);
                                        await expect(instance.connect(user).transferFrom(deployerAddress, NULL_ADDRESS, 50)).to
                                                .be.revertedWith("ERC20: transfer to the zero address");
                                });

                                it("should revert when owner doesn't have enough funds", async function () {
                                        await instance.connect(deployer).approve(userAddress, 12345678900);
                                        await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 12345678900)).to
                                                .be.revertedWith("ERC20: transfer amount exceeds balance");
                                });

                                it("should revert when trying to transfer from without being given allowance", async function () {
                                        await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 10)).to
                                                .be.revertedWith("ERC20: transfer amount exceeds allowance");
                                });
```

With that, our test is ready to be run.

<details>

<summary>Your test/Token.js should look like this:</summary>

```javascript
    const { expect } = require("chai");
    const { ContractFactory } = require("ethers");

    const TokenContract = require("../artifacts/contracts/Token.sol/Token.json");
    const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

    describe("Token contract", function () {
            let Token;
            let instance;
            let deployer;
            let user;
            let deployerAddress;
            let userAddress;

            beforeEach(async function () {
                    [deployer, user] = await ethers.getSigners();
                    deployerAddress = await deployer.getAddress();
                    userAddress = await user.getAddress();
                    Token = new ContractFactory(TokenContract.abi, TokenContract.bytecode, deployer);
                    instance = await Token.deploy(1234567890);
            });

            describe("Deployment", function () {
                    it("should set the correct token name", async function () {
                            expect(await instance.name()).to.equal("Token");
                    });

                    it("should set the correct token symbol", async function () {
                            expect(await instance.symbol()).to.equal("TKN");
                    });

                    it("should set the correct total supply", async function () {
                            expect(await instance.totalSupply()).to.equal(1234567890);
                    });

                    it("should assign the initial balance to the deployer", async function () {
                            expect(await instance.balanceOf(deployerAddress)).to.equal(1234567890);
                    });

                    it("should not assign value to a random address upon deployment", async function () {
                            expect(await instance.balanceOf(userAddress)).to.equal(0);
                    });

                    it("should not assign allowance upon deployment", async function () {
                            expect(await instance.allowance(deployerAddress, userAddress)).to.equal(0);
                            expect(await instance.allowance(userAddress, deployerAddress)).to.equal(0);
                    });
            });

            describe("Operation", function () {
                    this.timeout(50000);

                    describe("Transfer", function () {
                            describe("transfer()", function () {
                                    it("should change the balance of the sender and receiver when transferring token", async function () {
                                            const initialDeployerBalance = await instance.balanceOf(deployerAddress);
                                            const initialUserBalance = await instance.balanceOf(userAddress);

                                            await instance.connect(deployer).transfer(userAddress, 500);

                                            const finalDeployerBalance = await instance.balanceOf(deployerAddress);
                                            const finalUserBalance = await instance.balanceOf(userAddress);

                                            expect(initialDeployerBalance - 500).to.equal(finalDeployerBalance);
                                            expect(initialUserBalance + 500).to.equal(finalUserBalance);
                                    });

                                    it("should emit a Transfer event when transfering the token", async function () {
                                            await expect(instance.connect(deployer).transfer(userAddress, 100)).to
                                                    .emit(instance, "Transfer")
                                                    .withArgs(deployerAddress, userAddress, 100);
                                    });

                                    it("should revert the transfer to a 0x0 address", async function () {
                                            await expect(instance.connect(deployer).transfer(NULL_ADDRESS, 100)).to
                                                    .be.revertedWith("ERC20: transfer to the zero address");
                                    });

                                    it("should revert if trying to transfer amount bigger than balance", async function () {
                                            await expect(instance.connect(deployer).transfer(userAddress, 12345678900)).to
                                                    .be.revertedWith("ERC20: transfer amount exceeds balance");
                                    });
                            });
                    });

                    describe("Allowances", function () {
                            describe("approve()", function () {
                                    it("should grant allowance when the caller has enough funds", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);

                                            expect(await instance.allowance(deployerAddress, userAddress)).to
                                                    .equal(100);
                                    });


                                    it("should grant allowance when the caller has less funds than the ize of the allowance", async function () {
                                            await instance.connect(deployer).approve(userAddress, 12345678900);

                                            expect(await instance.allowance(deployerAddress, userAddress)).to
                                                    .equal(12345678900);
                                    });

                                    it("should emit Approval event when calling approve()", async function () {
                                            await expect(instance.connect(deployer).approve(userAddress, 100)).to
                                                    .emit(instance, "Approval")
                                                    .withArgs(deployerAddress, userAddress, 100);
                                    });

                                    it("should revert when trying to give allowance to 0x0 address", async function () {
                                            await expect(instance.connect(deployer).approve(NULL_ADDRESS, 100)).to
                                                    .be.revertedWith("ERC20: approve to the zero address");
                                    });
                            });

                            describe("increaseAllowance()", function () {
                                    it("should allow to increase allowance", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);

                                            const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                                            await instance.connect(deployer).increaseAllowance(userAddress, 50);

                                            const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                                            expect(finalAllowance - initialAllowance).to.equal(50);
                                    });

                                    it("should allow to increase allowance above the balance", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);

                                            const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                                            await instance.connect(deployer).increaseAllowance(userAddress, 1234567890);

                                            const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                                            expect(finalAllowance - initialAllowance).to.equal(1234567890);
                                    });

                                    it("should emit Approval event", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);
                                            await expect(instance.connect(deployer).increaseAllowance(userAddress, 40)).to
                                                    .emit(instance, "Approval")
                                                    .withArgs(deployerAddress, userAddress, 140);
                                    });

                                    it("should allow to increase allowance even if none was given before", async function () {
                                            await instance.connect(deployer).increaseAllowance(userAddress, 1234567890);

                                            const allowance = await instance.allowance(deployerAddress, userAddress);

                                            expect(allowance).to.equal(1234567890);
                                    });
                            });

                            describe("decreaseAllowance()", function() {
                                    it("should decrease the allowance", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);
                                            await instance.connect(deployer).decreaseAllowance(userAddress, 40);

                                            const allowance = await instance.allowance(deployerAddress, userAddress);

                                            expect(allowance).to.equal(60);
                                    });

                                    it("should emit Approval event", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);
                                            await expect(instance.connect(deployer).decreaseAllowance(userAddress, 40)).to
                                                    .emit(instance, "Approval")
                                                    .withArgs(deployerAddress, userAddress, 60);
                                    });

                                    it("should revert when tyring to decrease the allowance below 0", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);
                                            await expect(instance.connect(deployer).decreaseAllowance(userAddress, 1000)).to
                                                    .be.revertedWith("ERC20: decreased allowance below zero");
                                    });
                            });

                            describe("transferFrom()", function () {
                                    it("should allow to transfer tokens when allowance is given", async function () {

                                            await instance.connect(deployer).approve(userAddress, 100);

                                            const initialBalance = await instance.balanceOf(userAddress);

                                            await instance.connect(user).transferFrom(deployerAddress, userAddress, 50);

                                            const finalBalance = await instance.balanceOf(userAddress);

                                            expect(initialBalance + 50).to.equal(finalBalance);
                                    });

                                    it("should emit Transfer event when transferring from another address", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);

                                            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 40)).to
                                                    .emit(instance, "Transfer")
                                                    .withArgs(deployerAddress, userAddress, 40);
                                    });

                                    it("should emit Approval event when transferring from another address", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);

                                            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 40)).to
                                                    .emit(instance, "Approval")
                                                    .withArgs(deployerAddress, userAddress, 60);
                                    });

                                    it("should update the allowance when transferring from another address", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);
                                            await instance.connect(user).transferFrom(deployerAddress, userAddress, 40);

                                            expect(await instance.allowance(deployerAddress, userAddress)).to.equal(60);
                                    });

                                    it("should revert when tring to transfer more than allowed amount", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);
                                            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 1000)).to
                                                    .be.revertedWith("ERC20: transfer amount exceeds allowance");
                                    });

                                    it("should revert when transfering to 0x0 address", async function () {
                                            await instance.connect(deployer).approve(userAddress, 100);
                                            await expect(instance.connect(user).transferFrom(deployerAddress, NULL_ADDRESS, 50)).to
                                                    .be.revertedWith("ERC20: transfer to the zero address");
                                    });

                                    it("should revert when owner doesn't have enough funds", async function () {
                                            await instance.connect(deployer).approve(userAddress, 12345678900);
                                            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 12345678900)).to
                                                    .be.revertedWith("ERC20: transfer amount exceeds balance");
                                    });

                                    it("should revert when trying to transfer from without being given allowance", async function () {
                                            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 10)).to
                                                    .be.revertedWith("ERC20: transfer amount exceeds allowance");
                                    });
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
$ hardhat test test/Token.js --network mandala


  Token contract
    Deployment
      ✔ should set the correct token name (1112ms)
      ✔ should set the correct token symbol (1091ms)
      ✔ should set the correct total supply (1100ms)
      ✔ should assign the initial balance to the deployer (1100ms)
      ✔ should not assign value to a random address upon deployment (1094ms)
      ✔ should not assign allowance upon deployment (1116ms)
    Operation
      Transfer
        transfer()
          ✔ should change the balance of the sender and receiver when transferring token (4446ms)
          ✔ should emit a Transfer event when transfering the token (4292ms)
          ✔ should revert the transfer to a 0x0 address (1107ms)
          ✔ should revert if trying to transfer amount bigger than balance (1087ms)
      Allowances
        approve()
          ✔ should grant allowance when the caller has enough funds (4345ms)
          ✔ should grant allowance when the caller has less funds than the ize of the allowance (4327ms)
          ✔ should emit Approval event when calling approve() (4301ms)
          ✔ should revert when trying to give allowance to 0x0 address (1098ms)
        increaseAllowance()
          ✔ should allow to increase allowance (7623ms)
          ✔ should allow to increase allowance above the balance (7616ms)
          ✔ should emit Approval event (7602ms)
          ✔ should allow to increase allowance even if none was given before (4363ms)
        decreaseAllowance()
          ✔ should decrease the allowance (7580ms)
          ✔ should emit Approval event (7570ms)
          ✔ should revert when tyring to decrease the allowance below 0 (4375ms)
        transferFrom()
          ✔ should allow to transfer tokens when allowance is given (7689ms)
          ✔ should emit Transfer event when transferring from another address (7576ms)
          ✔ should emit Approval event when transferring from another address (7610ms)
          ✔ should update the allowance when transferring from another address (7604ms)
          ✔ should revert when tring to transfer more than allowed amount (4306ms)
          ✔ should revert when transfering to 0x0 address (4387ms)
          ✔ should revert when owner doesn't have enough funds (4318ms)
          ✔ should revert when trying to transfer from without being given allowance (1104ms)


  29 passing (3m)

✨  Done in 188.19s.
```

### Deploy script

This deployment script will deploy the contract and output the value of the `totalSupply` variable.

Within the `deploy.js` we will have the definition of main function called `main()` and then run it. Above it we will be importing the values needed for the deployment transaction parameters. We do this by placing the following code within the file:

```javascript
const { txParams } = require("../utils/transactionHelper");

async function main() {
    
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Our deploy script will reside in the definition (`async function main()`). First, we will set the transaction parameters for the deployment transaction and get the address of the account which will be used to deploy the smart contract. Then we get the `Token.sol` to the contract factory and deploy it and assign the deployed smart contract to the `instance` variable. Assigning the `instance` variable is optional and is only done, so that we can output the value returned by the `totalSupply()` getter to the terminal. We retrieve the value of `totalSupply` variable by calling `totalSupply()` from instance and outputting the result using console.log()\`:

```javascript
  const ethParams = await txParams();
  
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contract with the account:", deployer.address);

  console.log("Account balance:", (await deployer.getBalance()).toString());

  const Token = await ethers.getContractFactory("Token");
  const instance = await Token.deploy(
    1234567890,
    {
      gasPrice: ethParams.txGasPrice,
      gasLimit: ethParams.txGasLimit,
    }
  );

  console.log("Token address:", instance.address);

  const value = await instance.totalSupply();

  console.log("Total supply:", value.toNumber());
```

<details>

<summary>Your script/deploy.js should look like this:</summary>

```javascript
    const { txParams } = require("../utils/transactionHelper");

    async function main() {
            const ethParams = await txParams();

            const [deployer] = await ethers.getSigners();

            console.log("Deploying contract with the account:", deployer.address);

            console.log("Account balance:", (await deployer.getBalance()).toString());

            const Token = await ethers.getContractFactory("Token");
            const instance = await Token.deploy(
                    1234567890,
                    {
                            gasPrice: ethParams.txGasPrice,
                            gasLimit: ethParams.txGasLimit,
                    }
            );

            console.log("Token address:", instance.address);

            const value = await instance.totalSupply();

            console.log("Total supply:", value.toNumber());
    }

    main()
            .then(() => process.exit(0))
            .catch((error) => {
                    console.error(error);
                    process.exit(1);
    });
```

</details>

Running the `yarn deploy` script should return the following output:

```
yarn deploy


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ hardhat run scripts/deploy.js
Deploying contract with the account: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Account balance: 10000000000000000000000
Token address: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Total supply: 1234567890
✨  Done in 5.29s.
```

### Summary

We have built upon the previous examples and added an ERC20 smart contract and tested all of its functionalities. The tests were more detailed and covered more examples. We also ensured that we can interact with the smart contract and that its storage is modified as expected. We can compile smart contract with `yarn build`, test it with `yarn test`, `yarn test-mandala` or `yarn test-mandala:pubDev` and deploy it with `yarn deploy`, `yarn deploy-mandala` or `yarn deploy-mandala:pubDev`.
