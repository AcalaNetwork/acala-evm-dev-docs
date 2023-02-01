---
description: A tutorial on how to build an ERC-20 compatible smart contract in Acala EVM+.
---

# Token tutorial

## Table of contents

* [About](token-tutorial.md#about)
* [Smart contract](token-tutorial.md#smart-contract)
* [Test](token-tutorial.md#test)
* [Deploy script](token-tutorial.md#deploy-script)
* [Summary](token-tutorial.md#summary)

## About

This is an example that builds upon the [echo example](broken-reference). `Echo` was a simple example on building an intreactable state changing smart contract. `Token` is an example of [ERC20](https://eips.ethereum.org/EIPS/eip-20) token implementation in Acala EVM+. We won't be building an administrated or upgradeable token and we will use OpenZeppelin [ERC20 implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol). For the setup and naming, replace the `echo` with `token`. Let's jump into it!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/truffle-tutorials/tree/master/token](https://github.com/AcalaNetwork/truffle-tutorials/tree/master/token)
{% endhint %}

## Smart contract

In this tutorial we will be adding a simple smart contract that imports the ERC20 smart contract from `openzeppelin/contracts` and has a constructor that sets the initial balance of the sender (which also represents the total supply of the token) as well as the name of the token and its abbreviation:

To be able to import the `ERC20` from OpenZeppelin, we have to add `@openzeppelin/contracts` as a development dependency:

```shell
yarn add --dev @openzeppelin/contracts
```

Now that we have added the `@openzeppelin/contracts` dependency, we can focus on building our smart contract. Your empty smart contract should look like this:

```solidity
// SPDX-License-Identifier: MIT
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

<summary>Your contracts/Token.sol should look like this:</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20 {
    constructor(uint256 _initialBalance) ERC20("Token", "TKN") public {
        _mint(msg.sender, _initialBalance);
    }
}
```

</details>

As the Token smart contract is ready to be compiled, we can use the `yarn build` command (like we did in the hello-world) to compile the smart contract, which will create the `build` directory and contain the compiled smart contract.

## Test

We will be using the truffle-assertions dependency to validate reverts, so we can add it to our project with:

```shell
yarn add --dev truffle-assertions
```

Your test file should be called `token.js` and the empty test along with the import statement and global variable should look like this:

```javascript
const Token = artifacts.require("Token");
const truffleAssert = require('truffle-assertions');
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract("Token", function (accounts) {
  
});
```

{% hint style="info" %}
**NOTE: Make sure to uncomment the `accounts` test input variable, as we will be using different accounts in our tests.**
{% endhint %}

To prepare for the testing, we have to define `instance`, `deployer` and `user` global variables. The `instance` will store the deployed Token smart contract. The `deployer` and `user` variables will store the accounts that we will be using in the tests. Let's assign them values in the `beforeEach` action:

```javascript
  let instance;
  let deployer;
  let user;

  beforeEach("setup development environment", async function () {
    instance = await Token.deployed();
    deployer = accounts[0];
    user = accounts[1];
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

1. The contract should successfully deploy.
2. The `name` should equal `Token`.
3. The `symbol` should equal `TKN`.
4. The total supply of the smart contract should equal `1234567890`.
5. The initial balance of the `deployer` account should equal `1234567890`.
6. The `user` account should have `0` balance.
7. The allowances should be set to `0` when the smart contract is deployed.

```javascript
    it("should assert true", async function () {
      return assert.isTrue(true);
    });

    it("should set the correct token name", async function() {
      const name = await instance.name();

      expect(name).to.equal("Token");
    });

    it("should set the correct token symbol", async function() {
      const symbol = await instance.symbol();

      expect(symbol).to.equal("TKN");
    });

    it("should set the correct total supply", async function() {
      const totalSupply = await instance.totalSupply();

      expect(totalSupply.toNumber()).to.equal(1234567890);
    });

    it("should set the correct deployer balance", async function() {
      const balance = await instance.balanceOf(deployer);

      expect(balance.toNumber()).to.equal(1234567890);
    });

    it("should not assign value to a radnom addresss", async function() {
      const balance = await instance.balanceOf(user);

      expect(balance.toNumber()).to.equal(0);
    });

    it("should not assign allowance upon deployment", async function() {
      const allowance = await instance.allowance(deployer, user);

      expect(allowance.toNumber()).to.equal(0);
    });
```

The `Operation` block in itself is separated into two `describe` blocks, whichja are separated in itself:

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

      describe("decreaseAllowance()", function () {

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
        it("should update balances when transferring tokens", async function () {
          const initialDeployerBalance = await instance.balanceOf(deployer);
          const initialUserBalance = await instance.balanceOf(user);

          await instance.transfer(user, 100, { from: deployer });

          const finalDeployerBalance = await instance.balanceOf(deployer);
          const finalUserBalance = await instance.balanceOf(user);

          expect(initialDeployerBalance.toNumber() - finalDeployerBalance.toNumber()).to.equal(100);
          expect(finalUserBalance.toNumber() - initialUserBalance.toNumber()).to.equal(100);
        });

        it("should emit Transfer event", async function () {
          const response = await instance.transfer(user, 100, { from: deployer });

          const event = response.logs[0].event;
          const sender = response.logs[0].args.from;
          const receiver = response.logs[0].args.to;
          const value = response.logs[0].args.value;

          expect(event).to.equal("Transfer");
          expect(sender).to.equal(deployer);
          expect(receiver).to.equal(user);
          expect(value.toNumber()).to.equal(100);
        });

        it("should revet when trying to transfer to 0x0 address", async function () {
          await truffleAssert.reverts(
            instance.transfer(NULL_ADDRESS, 100, { from: deployer }),
            "ERC20: transfer to the zero address"
          );
        });

        it("should revert when trying to transfer more than own balance", async function () {
          await truffleAssert.reverts(
            instance.transfer(user, 12345678900, { from: deployer }),
            "ERC20: transfer amount exceeds balance"
          );
        });
```

The `approve()` example validates the following:

1. Allowance should be granted for less funds than the owner has.
2. Allowance should be granted for more funds than the owner has.
3. `Approval` event should be emitted when giving allowance.
4. Call should be reverted if the address receiving the allowance is `0x0`.

These examples should look like this:

```javascript
        it("should grant allowance for an amount smaller than own balance", async function () {
          await instance.approve(user, 100, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(100);
        });

        it("should grant allowance for an amount higher than own balance", async function () {
          await instance.approve(user, 12345678900, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(12345678900);
        });

        it("should emit Approval event", async function () {
          const response = await instance.approve(user, 100, { from: deployer });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal("Approval");
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(100);
        });

        it("should revert when trying to grant allowance to 0x0", async function () {
          await truffleAssert.reverts(
            instance.approve(NULL_ADDRESS, 100, { from: deployer }),
            "ERC20: approve to the zero address"
          );
        });
```

The `increaseAllowance()` example validates the following:

1. Owner should be able to increase allowance to an amount smaller that the total funds that they have.
2. Owner should be able to increase the balance to more than the amount of total funds that they have.
3. `Approval` event should be emitted.
4. The function can be called even if there was no preexisting allowance.

These examples should look like this:

```javascript
        it("should allow to increase the allowance to a total of less than the balance", async function () {
          await instance.approve(user, 100, { from: deployer });
          await instance.increaseAllowance(user, 50, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(150);
        });

        it("should allow to increase allowance to a gihger amount than own balance", async function () {
          await instance.approve(user, 100, { from: deployer });
          await instance.increaseAllowance(user, 1234567890, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(1234567990);
        });

        it("should emit Approval event", async function () {
          await instance.approve(user, 100, { from: deployer });

          const response = await instance.increaseAllowance(user, 50, { from: deployer });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal("Approval");
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(150);
        });

        it("should be allowed to be called without preexisting allowance", async function () {
          await instance.increaseAllowance(deployer, 50, { from: user });

          const allowance = await instance.allowance(user, deployer);

          expect(allowance.toNumber()).to.equal(50);
        });
```

The `decreaseAllowance()` example validates the following:

1. Owner should be able to decrease allowance.
2. `Approval` event should be emitted.
3. Call should be reverted when trying to decrease the allowance below 0.

These examples should look like this:

We can now add the following test cases to our describe block:

```javascript
        it("should allow owner to decrease allowance", async function () {
          await instance.approve(user, 100, { from: deployer });

          const initialAllowance = await instance.allowance(deployer, user);

          await instance.decreaseAllowance(user, 40, { from: deployer });

          const finalAllowance = await instance.allowance(deployer, user);

          expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(40);
        });

        it("should emit Approval event", async function () {
          await instance.approve(user, 100, { from: deployer });

          const response = await instance.decreaseAllowance(user, 40, { from: deployer });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal("Approval");
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(60);
        });

        it("should revert when trying to decrease allowance to below 0", async function () {
          await truffleAssert.reverts(
            instance.decreaseAllowance(user, 1000, { from: deployer }),
            "ERC20: decreased allowance below zero"
          );
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
        it("should allow transfer when allowance is given", async function () {
          await instance.approve(user, 1500, { from: deployer });

          const initalBalance = await instance.balanceOf(user);

          await instance.transferFrom(deployer, user, 1000, { from: user });

          const finalBalance = await instance.balanceOf(user);

          expect(finalBalance.toNumber() - initalBalance.toNumber()).to.equal(1000);
        });

        it("should emit Transfer event", async function () {
          await instance.approve(user, 1500, { from: deployer });

          const response = await instance.transferFrom(deployer, user, 1000, { from: user });

          const event = response.logs[1].event;
          const sender = response.logs[1].args.from;
          const receiver = response.logs[1].args.to;
          const value = response.logs[1].args.value;

          expect(event).to.equal("Transfer");
          expect(sender).to.equal(deployer);
          expect(receiver).to.equal(user);
          expect(value.toNumber()).to.equal(1000);
        });

        it("should emit Approval event", async function () {
          await instance.approve(user, 1500, { from: deployer });

          const response = await instance.transferFrom(deployer, user, 1000, { from: user });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal("Approval");
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(500);
        });

        it("should update the allowance", async function () {
          await instance.approve(user, 1500, { from: deployer });

          const initialAllowance = await instance.allowance(deployer, user);

          await instance.transferFrom(deployer, user, 1000, { from: user });

          const finalAllowance = await instance.allowance(deployer, user);

          expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(1000);
        });

        it("should revert when trying to transfer more than allowance", async function () {
          await instance.approve(user, 1500, { from: deployer });

          await truffleAssert.reverts(
            instance.transferFrom(deployer, user, 10000, { from: user }),
            "ERC20: insufficient allowance"
          );
        });

        it("should revert when trying to transfer to 0x0 address", async function () {
          await instance.approve(user, 1500, { from: deployer });

          await truffleAssert.reverts(
            instance.transferFrom(deployer, NULL_ADDRESS, 1000, { from: user }),
            "ERC20: transfer to the zero address"
          );
        });

        it("should revert when owner doesn't have enough funds", async function () {
          await instance.approve(user, 12345678900, { from: deployer });

          await truffleAssert.reverts(
            instance.transferFrom(deployer, user, 12345678900, { from: user }),
            "ERC20: transfer amount exceeds balance"
          );
        });

        it("should revert when no allowance was given", async function () {
          await truffleAssert.reverts(
            instance.transferFrom(user, deployer, 100, { from: deployer }),
            "ERC20: insufficient allowance"
          );
        });
```

With that, our test is ready to be run.

<details>

<summary>Your test/token.js should look like this:</summary>

```javascript
const Token = artifacts.require('Token');
const truffleAssert = require('truffle-assertions');
const NULL_ADDRESS = '0x0000000000000000000000000000000000000000';

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract('Token', function (accounts) {
  let instance;
  let deployer;
  let user;

  beforeEach('setup development environment', async function () {
    instance = await Token.deployed();
    deployer = accounts[0];
    user = accounts[1];
  });

  describe('Deployment', function () {
    it('should assert true', async function () {
      return assert.isTrue(true);
    });

    it('should set the correct token name', async function () {
      const name = await instance.name();

      expect(name).to.equal('Token');
    });

    it('should set the correct token symbol', async function () {
      const symbol = await instance.symbol();

      expect(symbol).to.equal('TKN');
    });

    it('should set the correct total supply', async function () {
      const totalSupply = await instance.totalSupply();

      expect(totalSupply.toNumber()).to.equal(1234567890);
    });

    it('should set the correct deployer balance', async function () {
      const balance = await instance.balanceOf(deployer);

      expect(balance.toNumber()).to.equal(1234567890);
    });

    it('should not assign value to a random addresss', async function () {
      const balance = await instance.balanceOf(user);

      expect(balance.toNumber()).to.equal(0);
    });

    it('should not assign allowance upon deployment', async function () {
      const allowance = await instance.allowance(deployer, user);

      expect(allowance.toNumber()).to.equal(0);
    });
  });

  describe('Operation', function () {
    describe('Transfer', function () {
      describe('transfer()', function () {
        it('should update balances when transferring tokens', async function () {
          const initialDeployerBalance = await instance.balanceOf(deployer);
          const initialUserBalance = await instance.balanceOf(user);

          await instance.transfer(user, 100, { from: deployer });

          const finalDeployerBalance = await instance.balanceOf(deployer);
          const finalUserBalance = await instance.balanceOf(user);

          expect(initialDeployerBalance.toNumber() - finalDeployerBalance.toNumber()).to.equal(100);
          expect(finalUserBalance.toNumber() - initialUserBalance.toNumber()).to.equal(100);
        });

        it('should emit Transfer event', async function () {
          const response = await instance.transfer(user, 100, { from: deployer });

          const event = response.logs[0].event;
          const sender = response.logs[0].args.from;
          const receiver = response.logs[0].args.to;
          const value = response.logs[0].args.value;

          expect(event).to.equal('Transfer');
          expect(sender).to.equal(deployer);
          expect(receiver).to.equal(user);
          expect(value.toNumber()).to.equal(100);
        });

        it('should revet when trying to transfer to 0x0 address', async function () {
          await truffleAssert.reverts(
            instance.transfer(NULL_ADDRESS, 100, { from: deployer }),
            'ERC20: transfer to the zero address'
          );
        });

        it('should revert when trying to transfer more than own balance', async function () {
          await truffleAssert.reverts(
            instance.transfer(user, 12345678900, { from: deployer }),
            'ERC20: transfer amount exceeds balance'
          );
        });
      });
    });

    describe('Allowances', function () {
      describe('approve()', function () {
        it('should grant allowance for an amount smaller than own balance', async function () {
          await instance.approve(user, 100, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(100);
        });

        it('should grant allowance for an amount higher than own balance', async function () {
          await instance.approve(user, 12345678900, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(12345678900);
        });

        it('should emit Approval event', async function () {
          const response = await instance.approve(user, 100, { from: deployer });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal('Approval');
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(100);
        });

        it('should revert when trying to grant allowance to 0x0', async function () {
          await truffleAssert.reverts(
            instance.approve(NULL_ADDRESS, 100, { from: deployer }),
            'ERC20: approve to the zero address'
          );
        });
      });

      describe('increaseAllowance()', function () {
        it('should allow to increase the allowance to a total of less than the balance', async function () {
          await instance.approve(user, 100, { from: deployer });
          await instance.increaseAllowance(user, 50, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(150);
        });

        it('should allow to increase allowance to a gihger amount than own balance', async function () {
          await instance.approve(user, 100, { from: deployer });
          await instance.increaseAllowance(user, 1234567890, { from: deployer });

          const allowance = await instance.allowance(deployer, user);

          expect(allowance.toNumber()).to.equal(1234567990);
        });

        it('should emit Approval event', async function () {
          await instance.approve(user, 100, { from: deployer });

          const response = await instance.increaseAllowance(user, 50, { from: deployer });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal('Approval');
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(150);
        });

        it('should be allowed to be called without preexisting allowance', async function () {
          await instance.increaseAllowance(deployer, 50, { from: user });

          const allowance = await instance.allowance(user, deployer);

          expect(allowance.toNumber()).to.equal(50);
        });
      });

      describe('decreaseAllowance()', function () {
        it('should allow owner to decrease allowance', async function () {
          await instance.approve(user, 100, { from: deployer });

          const initialAllowance = await instance.allowance(deployer, user);

          await instance.decreaseAllowance(user, 40, { from: deployer });

          const finalAllowance = await instance.allowance(deployer, user);

          expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(40);
        });

        it('should emit Approval event', async function () {
          await instance.approve(user, 100, { from: deployer });

          const response = await instance.decreaseAllowance(user, 40, { from: deployer });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal('Approval');
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(60);
        });

        it('should revert when trying to decrease allowance to below 0', async function () {
          await truffleAssert.reverts(
            instance.decreaseAllowance(user, 1000, { from: deployer }),
            'ERC20: decreased allowance below zero'
          );
        });
      });

      describe('transferFrom()', function () {
        it('should allow transfer when allowance is given', async function () {
          await instance.approve(user, 1500, { from: deployer });

          const initalBalance = await instance.balanceOf(user);

          await instance.transferFrom(deployer, user, 1000, { from: user });

          const finalBalance = await instance.balanceOf(user);

          expect(finalBalance.toNumber() - initalBalance.toNumber()).to.equal(1000);
        });

        it('should emit Transfer event', async function () {
          await instance.approve(user, 1500, { from: deployer });

          const response = await instance.transferFrom(deployer, user, 1000, { from: user });

          const event = response.logs[1].event;
          const sender = response.logs[1].args.from;
          const receiver = response.logs[1].args.to;
          const value = response.logs[1].args.value;

          expect(event).to.equal('Transfer');
          expect(sender).to.equal(deployer);
          expect(receiver).to.equal(user);
          expect(value.toNumber()).to.equal(1000);
        });

        it('should emit Approval event', async function () {
          await instance.approve(user, 1500, { from: deployer });

          const response = await instance.transferFrom(deployer, user, 1000, { from: user });

          const event = response.logs[0].event;
          const owner = response.logs[0].args.owner;
          const spender = response.logs[0].args.spender;
          const value = response.logs[0].args.value;

          expect(event).to.equal('Approval');
          expect(owner).to.equal(deployer);
          expect(spender).to.equal(user);
          expect(value.toNumber()).to.equal(500);
        });

        it('should update the allowance', async function () {
          await instance.approve(user, 1500, { from: deployer });

          const initialAllowance = await instance.allowance(deployer, user);

          await instance.transferFrom(deployer, user, 1000, { from: user });

          const finalAllowance = await instance.allowance(deployer, user);

          expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(1000);
        });

        it('should revert when trying to transfer more than allowance', async function () {
          await instance.approve(user, 1500, { from: deployer });

          await truffleAssert.reverts(
            instance.transferFrom(deployer, user, 10000, { from: user }),
            'ERC20: insufficient allowance'
          );
        });

        it('should revert when trying to transfer to 0x0 address', async function () {
          await instance.approve(user, 1500, { from: deployer });

          await truffleAssert.reverts(
            instance.transferFrom(deployer, NULL_ADDRESS, 1000, { from: user }),
            'ERC20: transfer to the zero address'
          );
        });

        it("should revert when owner doesn't have enough funds", async function () {
          await instance.approve(user, 12345678900, { from: deployer });

          await truffleAssert.reverts(
            instance.transferFrom(deployer, user, 12345678900, { from: user }),
            'ERC20: transfer amount exceeds balance'
          );
        });

        it('should revert when no allowance was given', async function () {
          await truffleAssert.reverts(
            instance.transferFrom(user, deployer, 100, { from: deployer }),
            'ERC20: insufficient allowance'
          );
        });
      });
    });
  });
});
```

</details>

{% hint style="warning" %}
**NOTE: You need to add a deployment script for `Token` before you can run the tests.**
{% endhint %}

When you run the test with (for example) `yarn test`, your tests should pass with the following output:

```shell
yarn test


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ truffle test
Using network 'development'.


Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.

Deploying Token
Token deployed at: 0xAF525B4ad720E87c1f695AfaD4A508CE6439c324


  Contract: Token
    Deployment
      ✓ should assert true
      ✓ should set the correct token name (76ms)
      ✓ should set the correct token symbol (57ms)
      ✓ should set the correct total supply (48ms)
      ✓ should set the correct deployer balance (48ms)
      ✓ should not assign value to a radnom addresss (38ms)
      ✓ should not assign allowance upon deployment (45ms)
    Operation
      Transfer
        transfer()
          ✓ should update balances when transferring tokens (393ms)
          ✓ should emit Transfer event (330ms)
          ✓ should revet when trying to transfer to 0x0 address (2465ms)
          ✓ should revert when trying to transfer more than own balance (195ms)
      Allowances
        approve()
          ✓ should grant allowance for an amount smaller than own balance (312ms)
          ✓ should grant allowance for an amount higher than own balance (295ms)
          ✓ should emit Approval event (251ms)
          ✓ should revert when trying to grant allowance to 0x0 (216ms)
        increaseAllowance()
          ✓ should allow to increase the allowance to a total of less than the balance (554ms)
          ✓ should allow to increase allowance to a gihger amount than own balance (631ms)
          ✓ should emit Approval event (597ms)
          ✓ should be allowed to be called without preexisting allowance (453ms)
        decreaseAllowance()
          ✓ should allow owner to decrease allowance (715ms)
          ✓ should emit Approval event (573ms)
          ✓ should revert when trying to decrease allowance to below 0 (514ms)
        transferFrom()
          ✓ should allow transfer when allowance is given (699ms)
          ✓ should emit Transfer event (694ms)
          ✓ should emit Approval event (780ms)
          ✓ should update the allowance (956ms)
          ✓ should revert when trying to transfer more than allowance (617ms)
          ✓ should revert when trying to transfer to 0x0 address (628ms)
          ✓ should revert when owner doesn't have enough funds (562ms)
          ✓ should revert when no allowance was given (356ms)


  30 passing (17s)

✨  Done in 33.93s.
```

## Deploy script

This deployment script will deploy the contract and output its address.

Within the `x_token.js` we will import the `Token` smart contract and have the blank migration ready. We do this by placing the following code within the file:

```javascript
const Token = artifacts.require("Token");

module.exports = async function (deployer) {
  
};
```

Within the script, we first log the `Deploying Token` to the console, to signal the start of the deployment, then we deploy it and log its address to the console:

```javascript
  console.log("Deploying Token");

  await deployer.deploy(Token, 1234567890);

  console.log("Token deployed at:", Token.address);
```

<details>

<summary>Your script/x_token.js should look like this:</summary>

```javascript
const Token = artifacts.require("Token");

module.exports = async function (deployer) {
  console.log("Deploying Token");
  
  await deployer.deploy(Token, 1234567890);

  console.log("Token deployed at:", Token.address);
};
```

</details>

Running the `yarn deploy` script should return the following output:

```shell
yarn deploy


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ truffle migrate

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      1638536305242
> Block gas limit: 6721975 (0x6691b7)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0x9063e7a597a7906237a103b99a6f2bf16012892fd99dea6672fec44723212710
   > Blocks: 0            Seconds: 0
   > contract address:    0x8E5C18252e6467fDf3B0Ab062a7B948564f4395B
   > block number:        418
   > block timestamp:     1638545238
   > account:             0x3fC83C6cbeE9B2F135f77afa617006E600190d64
   > balance:             99.31309972
   > gas used:            248842 (0x3cc0a)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00497684 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00497684 ETH


1638452059_token.js
===================
Deploying Token

   Deploying 'Token'
   -----------------
   > transaction hash:    0x03c5e73537081591ef4e592ab145707f2ab455785849732d340c4ccd46b841b7
   > Blocks: 0            Seconds: 0
   > contract address:    0xE7Ab5525c8526674CB75Dcd531D6c30e5002e1Db
   > block number:        420
   > block timestamp:     1638545240
   > account:             0x3fC83C6cbeE9B2F135f77afa617006E600190d64
   > balance:             99.28744262
   > gas used:            1240342 (0x12ed16)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.02480684 ETH

Token deployed at: 0xE7Ab5525c8526674CB75Dcd531D6c30e5002e1Db

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.02480684 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.02978368 ETH


✨  Done in 45.38s.
```

## Summary

We have built upon the previous examples and added an ERC20 smart contract and tested all of its functionalities. The tests were more detailed and covered more examples. We also ensured that we can interact with the smart contract and that its storage is modified as expected. We can compile smart contract with `yarn build`, test it with `yarn test` or `yarn test-mandala` and deploy it with `yarn deploy` or `yarn deploy-mandala`.
