---
description: >-
  This tutorial utilizes the predeployed DEX smart contract to swap the ERC20
  tokens of the  predeployed Token smart contracts, which we instantiate with
  the help of the ADDRESS utility.
---

# DEX tutorial

## Table of contents

* [About](dex-tutorial.md#about)
* [Smart contract](dex-tutorial.md#smart-contract)
* [Test](dex-tutorial.md#test)
* [User journey](dex-tutorial.md#user-journey)
* [Summary](dex-tutorial.md#summary)

## About

This example introduces the use of Acala EVM+ predeployed DEX that is present on every network at a fixed address (the address of a predeployed contract is the same on a local development network, public test network as well as the production network). As this example focuses on showcasing the interactions with the predeployed `DEX`, it doesn't have its own smart contract. We will get all of the required imports from the [`@acala-network/contracts`](https://github.com/AcalaNetwork/predeploy-contracts) dependency. The precompiles and predeploys are a specific feature of the Acala EVM+, so this tutorial is no longer compatible with traditional EVM development networks (like Ganache).

Let's take a look!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial [https://github.com/AcalaNetwork/truffle-tutorials/tree/master/DEX](https://github.com/AcalaNetwork/truffle-tutorials/tree/master/DEX)
{% endhint %}

## Smart contract

The smart contract in this tutorial is only used to satisfy the Truffle's requirement to have a smart contract to compile. For this, we will create an empty smart contract that will inherit the `DEX` from `@acala-network/contracts` dependency. Your skeleton smart contract should look like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

contract PrecompiledDEX {

}
```

Import of the `DEX` from `@acala-network/contracts` is done between the `pragma` definition and the start od the `contract` block:

```solidity
import "@acala-network/contracts/dex/DEX.sol";
```

As we now have access to `DEX.sol` from `@acala-network/contracts`, we can set the inheritance of our `PrecompiledDEX` contract:

```solidity
contract PrecompiledDEX is DEX {
```

This concludes our `PrecompiledDEX` smart contract.

<details>

<summary>Your contracts/PrecompiledDEX.sol should look like this:</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

import "@acala-network/contracts/dex/DEX.sol";

contract PrecompiledDEX is DEX {
    
}
```

</details>

{% hint style="info" %}
**NOTE: in order for Truffle to properly use the Token precompiles (used in the following section), we need to add the** [**PrecompiledToken**](precompiledtoken-tutorial.md#smart-contract)**.sol smart contract as well.**
{% endhint %}

## Test

Tests for this tutorial will validate the expected values returned and expected behaviour of DEX predeployed smart contract. The test file in our case is called `DEX.js`. Within it we import the `DEX` and `Token` from `@acala-network/contracts` dependency and assign it to `PrecompiledDEX` and `PrecompiledToken` variables. The `ACA`, `AUSD`, `LP_ACA_AUSD`, `DOT`, `RENBTC` and `DEX`, which are the exports from the `ADDRESS` utility of `@acala-network/contracts` dependency, are imported and they hold the values of the addresses of the predeployed smart contracts. The `MandalaAddress` utility holds the values of the predeployed smart contracts in the local development and Mandala network and we will be using this one. There are also `AcalaNetwork` and `KaruraNetwork`, that hold the addresses of their respective networks. We are also importing `truffleAssert` and `parseUnits` in order to ease our verification of the expected values and we are defining the `NULL_ADDRESS` constant, so we don't have to copy-paste the value when needed.

The test file with import statements and an empty test should look like this:

```javascript
const PrecompiledDEX = artifacts.require("@acala-network/contracts/build/contracts/DEX");
const PrecompiledToken = artifacts.require("@acala-network/contracts/build/contracts/Token");

const truffleAssert = require("truffle-assertions");
const { parseUnits } = require("ethers/lib/utils");

const { ACA, AUSD, LP_ACA_AUSD, DOT, RENBTC, DEX } = require("@acala-network/contracts/utils/MandalaAddress");
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract("PrecompiledDEX", function (accounts) {
  
});
```

To prepare for the testing, we have to define four global variables, `instance`, `AUSDinstance`, `ACAinstance` and `deployer`. The `instance` will store the predeployed DEX smart contract instance. `AUSDinstance` and `ACAinstance` will store the values of the token predeployed smart contracts. he `deployer` will store the account that we will be using in our tests. Let's assign them values in the `beforeEach` action:

```javascript
  let instance;
  let ACAinstance;
  let AUSDinstance;
  let deployer;

  beforeEach("setup development environment", async function () {
    deployer = accounts[0];
    instance = await PrecompiledDEX.at(DEX);
    ACAinstance = await PrecompiledToken.at(ACA);
    AUSDinstance = await PrecompiledToken.at(AUSD);
  });
```

You can see how we used the `DEX`, `ACA` and `AUSD` from the `ADDRESS` utility in order to set the addresses of our predeployed smart contract.

Our test will only contain one top-level section called `Operation` in which we will be checking the following functions (which will each be tested in its own section):

1. `getLiquidityPool` function to get the liquidity pool of the desired pair.
2. `getLiquidityTokenAddress` function to get the address of the liquidity token for the desired pair.
3. `getSwapTargetAddress` function to get the informative amount of the swap egress token based on set supply of the ingress token.
4. `getSwapSupplyAmount` function to get the informative amount of the swap supply based on the set target amount of egress token.
5. `swapWithExactSupply` function to swap tokens based on the set supply of the ingress token.
6. `swapWithExactTarget` function to swap tokens based on the set target of the egress token.
7. `addLiquidity` function that adds liquidity of the desired pair.
8. `removeLiquidity` function that removes liquidity of the desired pair.

The structure described above without the checks, should look like this:

```javascript
  describe("Operation", function () {
    describe("getLiquidityPool", function () {
    });

    describe("getLiquidityTokenAddress", function () {
      
    });

    describe("getSwapTargetAddress", function () {
      
    });

    describe("getSwapSupplyAmount", function () {
      
    });

    describe("swapWithExactSupply", function () {
      
    });

    describe("swapWithExactTarget", function () {
      
    });

    describe("addLiquidity", function () {
      
    });

    describe("removeLiquidity", function () {
      
    });
  });
```

When validating the `getLiquidityPool` function, we will check for the following examples:

1. `tokenA` should not be a `0x0` address.
2. `tokenB` should not be a `0x0` address.
3. Liquidity of the non-existent pair should be 0.
4. Liquidity should be returned for the existing pairs.

The section should look like this:

```javascript
      it("should not allow tokenA to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.getLiquidityPool(NULL_ADDRESS, ACA),
          "DEX: tokenA is zero address"
        );
      });

      it("should not allow tokenB to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.getLiquidityPool(ACA, NULL_ADDRESS),
          "DEX: tokenB is zero address"
        );
      });

      it("should return 0 liquidity for nonexistent pair", async function () {
        const response = await instance.getLiquidityPool(ACA, DOT);

        const liquidityA = response[0];
        const liquidityB = response[1];

        expect(liquidityA.isZero()).to.be.true;
        expect(liquidityB.isZero()).to.be.true;
      });

      it("should return liquidity for existing pairs", async function () {
        const response = await instance.getLiquidityPool(ACA, AUSD);

        const liquidityA = response[0];
        const liquidityB = response[1];

        expect(liquidityA.gt(web3.utils.toBN("0"))).to.be.true;
        expect(liquidityB.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `getLiquidityTokenAddress` function, we will check for the following examples:

1. `tokenA` should not be a `0x0` address.
2. `tokenB` should not be a `0x0` address.
3. Liquidity token address should be returned for the existing pairs.

The section should look like this:

```javascript
      it("should not allow tokenA to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.getLiquidityTokenAddress(NULL_ADDRESS, ACA),
          "DEX: tokenA is zero address"
        );
      });

      it("should not allow tokenB to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.getLiquidityTokenAddress(ACA, NULL_ADDRESS),
          "DEX: tokenB is zero address"
        );
      });

      it("should return liquidity token address for an existing pair", async function () {
        const response = await instance.getLiquidityTokenAddress(ACA, AUSD);

        expect(response).to.equal(LP_ACA_AUSD);
      });
```

When validating the `getSwapTargetAddress` function, we will check for the following examples:

1. `path` should not include the `0x0` address.
2. `supplyAmount` should not be 0.
3. Getting swap target amount should return 0 for an incompatible path.
4. Swap target amount should be returned when all parameters are correct.

The section should look like this:

```javascript
      it("should not allow for the path to include a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.getSwapTargetAmount([NULL_ADDRESS, ACA, DOT, RENBTC], 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.getSwapTargetAmount([ACA, NULL_ADDRESS, DOT, RENBTC], 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.getSwapTargetAmount([ACA, DOT, NULL_ADDRESS, RENBTC], 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.getSwapTargetAmount([ACA, DOT, RENBTC, NULL_ADDRESS], 12345678990),
          "DEX: token is zero address"
        );
      });

      it("should not allow supplyAmount to be 0", async function () {
        await truffleAssert.reverts(
          instance.getSwapTargetAmount([ACA, AUSD], 0),
          "DEX: supplyAmount is zero"
        );
      });

      it("should return 0 for an incompatible path", async function () {
        const expected_target = await instance.getSwapTargetAmount([ACA, DOT], 100);
        expect(expected_target).to.deep.equal(web3.utils.toBN('0'));
      });

      it("should return a swap target amount", async function () {
        const response = await instance.getSwapTargetAmount([ACA, AUSD], 100);

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `getSwapSupplyAmount` function, we will check for the following examples:

1. `path` should not include the `0x0` address.
2. `targetAmount` should not be 0.
3. Getting swap supply amount should return 0 for an incompatible path.
4. Swap supply amount should be returned when all parameters are correct.

The section should look like this:

```javascript
      it("should not allow an address in the path to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([NULL_ADDRESS, ACA, DOT, RENBTC], 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([ACA, NULL_ADDRESS, DOT, RENBTC], 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([ACA, DOT, NULL_ADDRESS, RENBTC], 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([ACA, DOT, RENBTC, NULL_ADDRESS], 12345678990),
          "DEX: token is zero address"
        );
      });

      it("should not allow targetAmount to be 0", async function () {
        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([ACA, AUSD], 0),
          "DEX: targetAmount is zero"
        );
      });

      it("should return 0 for an incompatible path", async function () {
        const expected_supply = await instance.getSwapSupplyAmount([ACA, DOT], 100);
        expect(expected_supply).to.deep.equal(web3.utils.toBN('0'));
      });

      it("should return the supply amount", async function () {
        const response = await instance.getSwapSupplyAmount([ACA, AUSD], 100);

        expect(response.gt(web3.utils.toBN("0"))).to.be.true;
      });
```

When validating the `swapWithExactSupply` function, we will check for the following examples:

1. `path` should not include the `0x0` address.
2. `supplyAmount` should not be 0.
3. Egress token balance of the caller should increase.
4. Successful execution should emit a `Swaped` event.

The section should look like this:

```javascript
      it("should not allow path to contain a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.swapWithExactSupply([NULL_ADDRESS, ACA, DOT, RENBTC], 12345678990, 1),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.swapWithExactSupply([ACA, NULL_ADDRESS, DOT, RENBTC], 12345678990, 1),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.swapWithExactSupply([ACA, DOT, NULL_ADDRESS, RENBTC], 12345678990, 1),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.swapWithExactSupply([ACA, DOT, RENBTC, NULL_ADDRESS], 12345678990, 1),
          "DEX: token is zero address"
        );
      });

      it("should not allow supplyAmount to be 0", async function () {
        await truffleAssert.reverts(
          instance.swapWithExactSupply([ACA, AUSD], 0, 1),
          "DEX: supplyAmount is zero"
        );
      });

      it("should allocate the tokens to the caller", async function () {
        const initalBalance = await ACAinstance.balanceOf(deployer);
        const initBal = await AUSDinstance.balanceOf(deployer);
        const path = [ACA, AUSD];
        const expected_target = await instance.getSwapTargetAmount(path, 100);

        await instance.swapWithExactSupply(path, 100, 1, { from: deployer });

        const finalBalance = await ACAinstance.balanceOf(deployer);
        const finBal = await AUSDinstance.balanceOf(deployer);

        // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
        expect(finalBalance.lt(initalBalance.sub(web3.utils.toBN(100)))).to.be.true;
        expect(finBal.eq(initBal.add(expected_target))).to.be.true;
      });

      it("should emit a Swaped event", async function () {
        const path = [ACA, AUSD];
        const expected_target = await instance.getSwapTargetAmount(path, 100);

        const tx = await instance.swapWithExactSupply(path, 100, 1, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const event_path = tx.logs[0].args.path;
        const supplyAmount = tx.logs[0].args.supplyAmount;
        const targetAmount = tx.logs[0].args.targetAmount;

        expect(event).to.equal("Swaped");
        expect(sender).to.equal(deployer);
        expect(event_path).to.deep.equal(path);
        expect(supplyAmount).to.deep.equal(web3.utils.toBN(100));
        expect(targetAmount).to.deep.equal(expected_target);
      });
```

When validating the `swapWithExactTarget` function, we will check for the following examples:

1. `path` should not include the `0x0` address.
2. `targetAmount` should not be 0.
3. Egress token balance of the caller should increase.
4. Successful execution should emit a `Swaped` event.

The section should look like this:

```javascript
      it("should not allow a token in a path to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.swapWithExactTarget([NULL_ADDRESS, ACA, DOT, RENBTC], 1, 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, NULL_ADDRESS, DOT, RENBTC], 1, 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, DOT, NULL_ADDRESS, RENBTC], 1, 12345678990),
          "DEX: token is zero address"
        );

        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, DOT, RENBTC, NULL_ADDRESS], 1, 12345678990),
          "DEX: token is zero address"
        );
      });

      it("should not allow targetAmount to be 0", async function () {
        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, AUSD], 0, 1234567890),
          "DEX: targetAmount is zero"
        );
      });

      it("should allocate tokens to the caller", async function () {
        const initalBalance = await ACAinstance.balanceOf(deployer);
        const initBal = await AUSDinstance.balanceOf(deployer);
        const path = [ACA, AUSD];
        const expected_supply = await instance.getSwapSupplyAmount(path, 100);

        await instance.swapWithExactTarget(path, 100, 1234567890, { from: deployer });

        const finalBalance = await ACAinstance.balanceOf(deployer);
        const finBal = await AUSDinstance.balanceOf(deployer);

        // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
        expect(finalBalance.lt(initalBalance.sub(expected_supply))).to.be.true;
        expect(finBal.eq(initBal.add(web3.utils.toBN(100)))).to.be.true;
      });

      it("should emit Swaped event", async function () {
        const path = [ACA, AUSD];
        const expected_supply = await instance.getSwapSupplyAmount(path, 100);

        const tx = await instance.swapWithExactTarget(path, 100, 1234567890, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const event_path = tx.logs[0].args.path;
        const supplyAmount = tx.logs[0].args.supplyAmount;
        const targetAmount = tx.logs[0].args.targetAmount;

        expect(event).to.equal("Swaped");
        expect(sender).to.equal(deployer);
        expect(event_path).to.deep.equal(path);
        expect(supplyAmount).to.deep.equal(expected_supply);
        expect(targetAmount).to.deep.equal(web3.utils.toBN(100));
      });
```

When validating the `addLiquidity` function, we will check for the following examples:

1. `tokenA` should not be a `0x0` address.
2. `tokenB` should not be a `0x0` address.
3. `maxAmountA` should not be 0.
4. `maxAmountB` should not be 0.
5. Successfull execution should increase the liquidity of the pair.
6. Successfull execution should emit `AddedLiquidity` event.

The section should look like this:

```javascript
      it("should not allow tokenA to be 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.addLiquidity(NULL_ADDRESS, AUSD, 1000, 1000, 1),
          "DEX: tokenA is zero address"
        );
      });

      it("should not allow tokenB to be 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.addLiquidity(ACA, NULL_ADDRESS, 1000, 1000, 1),
          "DEX: tokenB is zero address"
        );
      });

      it("should not allow maxAmountA to be 0", async function () {
        await truffleAssert.reverts(
          instance.addLiquidity(ACA, AUSD, 0, 1000, 1),
          "DEX: maxAmountA is zero"
        );
      });

      it("should not allow maxAmountB to be 0", async function () {
        await truffleAssert.reverts(
          instance.addLiquidity(ACA, AUSD, 1000, 0, 1),
          "DEX: maxAmountB is zero"
        );
      });
      
      it("should increase liquidity", async function () {
        const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        await instance.addLiquidity(ACA, AUSD, parseUnits("2", 12), parseUnits("2", 12), 1);

        const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        expect(finalLiquidity[0].gt(intialLiquidity[0])).to.be.true;
        expect(finalLiquidity[1].gt(intialLiquidity[1])).to.be.true;
      });

      it("should emit AddedLiquidity event", async function () {
        const tx = await instance.addLiquidity(ACA, AUSD, 1000, 1000, 1, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const tokenA = tx.logs[0].args.tokenA;
        const tokenB = tx.logs[0].args.tokenB;
        const maxAmountA = tx.logs[0].args.maxAmountA;
        const maxAmountB = tx.logs[0].args.maxAmountB;

        expect(event).to.equal("AddedLiquidity");
        expect(sender).to.equal(deployer);
        expect(tokenA).to.deep.equal(ACA);
        expect(tokenB).to.deep.equal(AUSD);
        expect(maxAmountA).to.deep.equal(web3.utils.toBN(1000));
        expect(maxAmountB).to.deep.equal(web3.utils.toBN(1000));
      });
```

When validating the `removeLiquidity` function, we will check for the following examples:

1. `tokenA` should not be a `0x0` address.
2. `tokenB` should not be a `0x0` address.
3. `removeShare` should not be 0.
4. Successfull execution should reduce the liquidity of the pair.
5. Successfull execution should emit `RemovedLiquidity` event.

The section should look like this:

```javascript
      it("should not allow tokenA to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.removeLiquidity(NULL_ADDRESS, AUSD, 1, 0, 0),
          "DEX: tokenA is zero address"
        );
      });

      it("should not allow tokenB to be a 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.removeLiquidity(ACA, NULL_ADDRESS, 1, 0, 0),
          "DEX: tokenB is zero address"
        );
      });

      it("should not allow removeShare to be 0", async function () {
        await truffleAssert.reverts(
          instance.removeLiquidity(ACA, AUSD, 0, 0, 0),
          "DEX: removeShare is zero"
        );
      });

      it("should reduce the liquidity", async function () {
        const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        await instance.removeLiquidity(ACA, AUSD, 10, 1, 1);

        const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        expect(intialLiquidity[0].gt(finalLiquidity[0])).to.be.true;
        expect(intialLiquidity[1].gt(finalLiquidity[1])).to.be.true;
      });

      it("should emit RemovedLiquidity event", async function () {
        const tx = await instance.removeLiquidity(ACA, AUSD, 1, 0, 0, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const tokenA = tx.logs[0].args.tokenA;
        const tokenB = tx.logs[0].args.tokenB;
        const removeShare = tx.logs[0].args.removeShare;

        expect(event).to.equal("RemovedLiquidity");
        expect(sender).to.equal(deployer);
        expect(tokenA).to.deep.equal(ACA);
        expect(tokenB).to.deep.equal(AUSD);
        expect(removeShare).to.deep.equal(web3.utils.toBN(1));
      });
```

With that, our test is ready to be run.

<details>

<summary>Your test/DEX.js should look like this:</summary>

```javascript
const PrecompiledDEX = artifacts.require('@acala-network/contracts/build/contracts/DEX');
const PrecompiledToken = artifacts.require('@acala-network/contracts/build/contracts/Token');

const truffleAssert = require('truffle-assertions');
const { parseUnits } = require('ethers/lib/utils');

const { ACA, AUSD, LP_ACA_AUSD, DOT, RENBTC, DEX } = require('@acala-network/contracts/utils/MandalaAddress');
const NULL_ADDRESS = '0x0000000000000000000000000000000000000000';

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract('PrecompiledDEX', function (accounts) {
  let instance;
  let ACAinstance;
  let AUSDinstance;
  let deployer;

  beforeEach('setup development environment', async function () {
    deployer = accounts[0];
    instance = await PrecompiledDEX.at(DEX);
    ACAinstance = await PrecompiledToken.at(ACA);
    AUSDinstance = await PrecompiledToken.at(AUSD);
  });

  describe('Operation', function () {
    describe('getLiquidityPool', function () {
      it('should not allow tokenA to be a 0x0 address', async function () {
        await truffleAssert.reverts(instance.getLiquidityPool(NULL_ADDRESS, ACA), 'DEX: tokenA is zero address');
      });

      it('should not allow tokenB to be a 0x0 address', async function () {
        await truffleAssert.reverts(instance.getLiquidityPool(ACA, NULL_ADDRESS), 'DEX: tokenB is zero address');
      });

      it('should return 0 liquidity for nonexistent pair', async function () {
        const response = await instance.getLiquidityPool(ACA, DOT);

        const liquidityA = response[0];
        const liquidityB = response[1];

        expect(liquidityA.isZero()).to.be.true;
        expect(liquidityB.isZero()).to.be.true;
      });

      it('should return liquidity for existing pairs', async function () {
        const response = await instance.getLiquidityPool(ACA, AUSD);

        const liquidityA = response[0];
        const liquidityB = response[1];

        expect(liquidityA.gt(web3.utils.toBN('0'))).to.be.true;
        expect(liquidityB.gt(web3.utils.toBN('0'))).to.be.true;
      });
    });

    describe('getLiquidityTokenAddress', function () {
      it('should not allow tokenA to be a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.getLiquidityTokenAddress(NULL_ADDRESS, ACA),
          'DEX: tokenA is zero address'
        );
      });

      it('should not allow tokenB to be a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.getLiquidityTokenAddress(ACA, NULL_ADDRESS),
          'DEX: tokenB is zero address'
        );
      });

      it('should return liquidity token address for an existing pair', async function () {
        const response = await instance.getLiquidityTokenAddress(ACA, AUSD);

        expect(response).to.equal(LP_ACA_AUSD);
      });
    });

    describe('getSwapTargetAddress', function () {
      it('should not allow for the path to include a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.getSwapTargetAmount([NULL_ADDRESS, ACA, DOT, RENBTC], 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.getSwapTargetAmount([ACA, NULL_ADDRESS, DOT, RENBTC], 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.getSwapTargetAmount([ACA, DOT, NULL_ADDRESS, RENBTC], 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.getSwapTargetAmount([ACA, DOT, RENBTC, NULL_ADDRESS], 12345678990),
          'DEX: token is zero address'
        );
      });

      it('should not allow supplyAmount to be 0', async function () {
        await truffleAssert.reverts(instance.getSwapTargetAmount([ACA, AUSD], 0), 'DEX: supplyAmount is zero');
      });

      it('should return 0 for an incompatible path', async function () {
        const expected_target = await instance.getSwapTargetAmount([ACA, DOT], 100);
        expect(expected_target).to.deep.equal(web3.utils.toBN('0'));
      });

      it('should return a swap target amount', async function () {
        const response = await instance.getSwapTargetAmount([ACA, AUSD], 100);

        expect(response.gt(web3.utils.toBN('0'))).to.be.true;
      });
    });

    describe('getSwapSupplyAmount', function () {
      it('should not allow an address in the path to be a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([NULL_ADDRESS, ACA, DOT, RENBTC], 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([ACA, NULL_ADDRESS, DOT, RENBTC], 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([ACA, DOT, NULL_ADDRESS, RENBTC], 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.getSwapSupplyAmount([ACA, DOT, RENBTC, NULL_ADDRESS], 12345678990),
          'DEX: token is zero address'
        );
      });

      it('should not allow targetAmount to be 0', async function () {
        await truffleAssert.reverts(instance.getSwapSupplyAmount([ACA, AUSD], 0), 'DEX: targetAmount is zero');
      });

      it('should return 0 for an incompatible path', async function () {
        const expected_supply = await instance.getSwapSupplyAmount([ACA, DOT], 100);
        expect(expected_supply).to.deep.equal(web3.utils.toBN('0'));
      });

      it('should return the supply amount', async function () {
        const response = await instance.getSwapSupplyAmount([ACA, AUSD], 100);

        expect(response.gt(web3.utils.toBN('0'))).to.be.true;
      });
    });

    describe('swapWithExactSupply', function () {
      it('should not allow path to contain a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.swapWithExactSupply([NULL_ADDRESS, ACA, DOT, RENBTC], 12345678990, 1),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.swapWithExactSupply([ACA, NULL_ADDRESS, DOT, RENBTC], 12345678990, 1),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.swapWithExactSupply([ACA, DOT, NULL_ADDRESS, RENBTC], 12345678990, 1),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.swapWithExactSupply([ACA, DOT, RENBTC, NULL_ADDRESS], 12345678990, 1),
          'DEX: token is zero address'
        );
      });

      it('should not allow supplyAmount to be 0', async function () {
        await truffleAssert.reverts(instance.swapWithExactSupply([ACA, AUSD], 0, 1), 'DEX: supplyAmount is zero');
      });

      it('should allocate the tokens to the caller', async function () {
        const initalBalance = await ACAinstance.balanceOf(deployer);
        const initBal = await AUSDinstance.balanceOf(deployer);
        const path = [ACA, AUSD];
        const expected_target = await instance.getSwapTargetAmount(path, 100);

        await instance.swapWithExactSupply(path, 100, 1, { from: deployer });

        const finalBalance = await ACAinstance.balanceOf(deployer);
        const finBal = await AUSDinstance.balanceOf(deployer);

        // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
        expect(finalBalance.lt(initalBalance.sub(web3.utils.toBN(100)))).to.be.true;
        expect(finBal.eq(initBal.add(expected_target))).to.be.true;
      });

      it('should emit a Swaped event', async function () {
        const path = [ACA, AUSD];
        const expected_target = await instance.getSwapTargetAmount(path, 100);

        const tx = await instance.swapWithExactSupply(path, 100, 1, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const event_path = tx.logs[0].args.path;
        const supplyAmount = tx.logs[0].args.supplyAmount;
        const targetAmount = tx.logs[0].args.targetAmount;

        expect(event).to.equal('Swaped');
        expect(sender).to.equal(deployer);
        expect(event_path).to.deep.equal(path);
        expect(supplyAmount).to.deep.equal(web3.utils.toBN(100));
        expect(targetAmount).to.deep.equal(expected_target);
      });
    });

    describe('swapWithExactTarget', function () {
      it('should not allow a token in a path to be a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.swapWithExactTarget([NULL_ADDRESS, ACA, DOT, RENBTC], 1, 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, NULL_ADDRESS, DOT, RENBTC], 1, 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, DOT, NULL_ADDRESS, RENBTC], 1, 12345678990),
          'DEX: token is zero address'
        );

        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, DOT, RENBTC, NULL_ADDRESS], 1, 12345678990),
          'DEX: token is zero address'
        );
      });

      it('should not allow targetAmount to be 0', async function () {
        await truffleAssert.reverts(
          instance.swapWithExactTarget([ACA, AUSD], 0, 1234567890),
          'DEX: targetAmount is zero'
        );
      });

      it('should allocate tokens to the caller', async function () {
        const initalBalance = await ACAinstance.balanceOf(deployer);
        const initBal = await AUSDinstance.balanceOf(deployer);
        const path = [ACA, AUSD];
        const expected_supply = await instance.getSwapSupplyAmount(path, 100);

        await instance.swapWithExactTarget(path, 100, 1234567890, { from: deployer });

        const finalBalance = await ACAinstance.balanceOf(deployer);
        const finBal = await AUSDinstance.balanceOf(deployer);

        // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
        expect(finalBalance.lt(initalBalance.sub(expected_supply))).to.be.true;
        expect(finBal.eq(initBal.add(web3.utils.toBN(100)))).to.be.true;
      });

      it('should emit Swaped event', async function () {
        const path = [ACA, AUSD];
        const expected_supply = await instance.getSwapSupplyAmount(path, 100);

        const tx = await instance.swapWithExactTarget(path, 100, 1234567890, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const event_path = tx.logs[0].args.path;
        const supplyAmount = tx.logs[0].args.supplyAmount;
        const targetAmount = tx.logs[0].args.targetAmount;

        expect(event).to.equal('Swaped');
        expect(sender).to.equal(deployer);
        expect(event_path).to.deep.equal(path);
        expect(supplyAmount).to.deep.equal(expected_supply);
        expect(targetAmount).to.deep.equal(web3.utils.toBN(100));
      });
    });

    describe('addLiquidity', function () {
      it('should not allow tokenA to be 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.addLiquidity(NULL_ADDRESS, AUSD, 1000, 1000, 1),
          'DEX: tokenA is zero address'
        );
      });

      it('should not allow tokenB to be 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.addLiquidity(ACA, NULL_ADDRESS, 1000, 1000, 1),
          'DEX: tokenB is zero address'
        );
      });

      it('should not allow maxAmountA to be 0', async function () {
        await truffleAssert.reverts(instance.addLiquidity(ACA, AUSD, 0, 1000, 1), 'DEX: maxAmountA is zero');
      });

      it('should not allow maxAmountB to be 0', async function () {
        await truffleAssert.reverts(instance.addLiquidity(ACA, AUSD, 1000, 0, 1), 'DEX: maxAmountB is zero');
      });

      it('should increase liquidity', async function () {
        const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        await instance.addLiquidity(ACA, AUSD, parseUnits('2', 12), parseUnits('2', 12), 1);

        const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        expect(finalLiquidity[0].gt(intialLiquidity[0])).to.be.true;
        expect(finalLiquidity[1].gt(intialLiquidity[1])).to.be.true;
      });

      it('should emit AddedLiquidity event', async function () {
        const tx = await instance.addLiquidity(ACA, AUSD, 1000, 1000, 1, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const tokenA = tx.logs[0].args.tokenA;
        const tokenB = tx.logs[0].args.tokenB;
        const maxAmountA = tx.logs[0].args.maxAmountA;
        const maxAmountB = tx.logs[0].args.maxAmountB;

        expect(event).to.equal('AddedLiquidity');
        expect(sender).to.equal(deployer);
        expect(tokenA).to.deep.equal(ACA);
        expect(tokenB).to.deep.equal(AUSD);
        expect(maxAmountA).to.deep.equal(web3.utils.toBN(1000));
        expect(maxAmountB).to.deep.equal(web3.utils.toBN(1000));
      });
    });

    describe('removeLiquidity', function () {
      it('should not allow tokenA to be a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.removeLiquidity(NULL_ADDRESS, AUSD, 1, 0, 0),
          'DEX: tokenA is zero address'
        );
      });

      it('should not allow tokenB to be a 0x0 address', async function () {
        await truffleAssert.reverts(
          instance.removeLiquidity(ACA, NULL_ADDRESS, 1, 0, 0),
          'DEX: tokenB is zero address'
        );
      });

      it('should not allow removeShare to be 0', async function () {
        await truffleAssert.reverts(instance.removeLiquidity(ACA, AUSD, 0, 0, 0), 'DEX: removeShare is zero');
      });

      it('should reduce the liquidity', async function () {
        const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        await instance.removeLiquidity(ACA, AUSD, 10, 1, 1);

        const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        expect(intialLiquidity[0].gt(finalLiquidity[0])).to.be.true;
        expect(intialLiquidity[1].gt(finalLiquidity[1])).to.be.true;
      });

      it('should emit RemovedLiquidity event', async function () {
        const tx = await instance.removeLiquidity(ACA, AUSD, 1, 0, 0, { from: deployer });

        const event = tx.logs[0].event;
        const sender = tx.logs[0].args.sender;
        const tokenA = tx.logs[0].args.tokenA;
        const tokenB = tx.logs[0].args.tokenB;
        const removeShare = tx.logs[0].args.removeShare;

        expect(event).to.equal('RemovedLiquidity');
        expect(sender).to.equal(deployer);
        expect(tokenA).to.deep.equal(ACA);
        expect(tokenB).to.deep.equal(AUSD);
        expect(removeShare).to.deep.equal(web3.utils.toBN(1));
      });
    });
  });
});
```

</details>

{% hint style="info" %}
**NOTE: If you want to interact with other precompiled and predeployed smart contracts, you can take a look at the list of all of the smart contracts supported in the** [**`ADDRESS` utility**](https://github.com/AcalaNetwork/predeploy-contracts/blob/master/contracts/utils/MandalaAddress.js) **and tweak this example test.**
{% endhint %}

When you run the test with (for example) `yarn test-mandala`, your tests should pass with the following output:

```shell
yarn test-mandala


yarn run v1.22.17
$ truffle test --network mandala
Using network 'mandala'.


Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.


  Contract: PrecompiledDEX
    Operation
      getLiquidityPool
        ✓ should not allow tokenA to be a 0x0 address (67ms)
        ✓ should not allow tokenB to be a 0x0 address
        ✓ should return 0 liquidity for nonexistent pair
        ✓ should return liquidity for existing pairs
      getLiquidityTokenAddress
        ✓ should not allow tokenA to be a 0x0 address
        ✓ should not allow tokenB to be a 0x0 address
        ✓ should return liquidity token address for an existing pair
      getSwapTargetAddress
        ✓ should not allow for the path to include a 0x0 address (50ms)
        ✓ should not allow supplyAmount to be 0
        ✓ should revert for an incompatible path
        ✓ should return a swap target amount (46ms)
      getSwapSupplyAmount
        ✓ should not allow an address in the path to be a 0x0 address (75ms)
        ✓ should not allow targetAmount to be 0
        ✓ should revert for an incompatible path
        ✓ should return the supply amount
      swapWithExactSupply
        ✓ should not allow path to contain a 0x0 address (161ms)
        ✓ should not allow supplyAmount to be 0 (52ms)
        ✓ should allocate the tokens to the caller (1293ms)
        ✓ should emit a Swaped event (1199ms)
      swapWithExactTarget
        ✓ should not allow a token in a path to be a 0x0 address (327ms)
        ✓ should not allow targetAmount to be 0 (58ms)
        ✓ should allocate tokens to the caller (1253ms)
        ✓ should emit Swaped event (1141ms)
      addLiquidity
        ✓ should not allow tokenA to be 0x0 address (70ms)
        ✓ should not allow tokenB to be 0x0 address (56ms)
        ✓ should not allow maxAmountA to be 0 (40ms)
        ✓ should not allow maxAmountB to be 0 (51ms)
        ✓ should increase liquidity (1186ms)
        ✓ should emit AddedLiquidity event (1126ms)
      removeLiquidity
        ✓ should not allow tokenA to be a 0x0 address (64ms)
        ✓ should not allow tokenB to be a 0x0 address
        ✓ should not allow removeShare to be 0 (53ms)
        ✓ should reduce the liquidity (1260ms)
        ✓ should emit RemovedLiquidity event (1111ms)


  34 passing (16s)

✨  Done in 18.75s.
```

## User journey

Since there is no contract to deploy, let's add a simulation of a user interacting with a `DEX` and log all of the changes and information to the console. The script will be called `userJourney.js` and will reside in the `scripts/` folder:

```shell
touch scripts/userJourney.js
```

The empty user journey script together with the imports of `ACA`, `AUSD`, `DEX` and `DOT` from `@acala-network/contracts`, `formatUnits` and `parseUnits` from `ethers` and precompiled `DEX` and `Token` smart contracts from `@acala-network/contracts` should look like this:

```javascript
const DEXContract = artifacts.require("@acala-network/contracts/build/contracts/DEX");
const TokenContract = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { ACA, AUSD, DEX, DOT } = require("@acala-network/contracts/utils/Address");
const { formatUnits, parseUnits } = require("ethers/lib/utils");

module.exports = async function(callback) {
  try {
    
  }
  catch(error) {
    console.log(error)
  }

  callback()
}
```

We will pad the log to the console with empty strings in order to get a more verbose output. At the beginning of the script, we assign an address value to `deployer`, get the its initial native balance and instantiate the predeployed `DEX` and `Token` smart contracts with the help of the `ADDRESS` utility:

```javascript
    console.log("");
    console.log("");

    const accounts = await web3.eth.getAccounts();
    const deployer = accounts[0];
    
    console.log("Interacting with DEX using account:", deployer);

    console.log(`Initial account balance: ${formatUnits((await web3.eth.getBalance(deployer)).toString(), 12)} ACA`);

    console.log("");
    console.log("");

    console.log("Instantiating DEX and token smart contracts");

    const instance = await DEXContract.at(DEX);
    const ACAinstance = await TokenContract.at(ACA);
    const AUSDinstance = await TokenContract.at(AUSD);
    const DOTinstance = await TokenContract.at(DOT);

    console.log("DEX instantiated with address", instance.address);
    console.log("ACA token instantiated with address", ACAinstance.address);
    console.log("AUSD token instantiated with address", AUSDinstance.address);
    console.log("DOT token instantiated with address", DOTinstance.address);
```

Now that we have instantiated the token smart contracts, we can get the balance of the `deployer` for each of them and log the initial balances to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Getting inital token balances");

    const initialAcaBalance = await ACAinstance.balanceOf(deployer);
    const initialAusdBalance = await AUSDinstance.balanceOf(deployer);
    const initialDotBalance = await DOTinstance.balanceOf(deployer);

    const acaDecimals = (await ACAinstance.decimals()).toNumber();
    const ausdDecimals = (await AUSDinstance.decimals()).toNumber();
    const dotDecimals = (await DOTinstance.decimals()).toNumber();

    console.log("Inital %s ACA balance: %s ACA", deployer, formatUnits(initialAcaBalance.toString(), acaDecimals));
    console.log("Inital %s AUSD balance: %s AUSD", deployer, formatUnits(initialAusdBalance.toString(), ausdDecimals));
    console.log("Inital %s DOT balance: %s DOT", deployer, formatUnits(initialDotBalance.toString(), dotDecimals));
```

Let's query some information from `DEX`. First thing that we will query on our journey are the liquidity pools for 3 pairs. After we log the results to the console, we can query the liquidity pool tokens for the same three pairs and log those as well:

```javascript
    console.log("");
    console.log("");

    console.log("Getting liquidity pools");

    const initialAcaAusdLP = await instance.getLiquidityPool(ACA, AUSD);
    const initialAcaDotLP = await instance.getLiquidityPool(ACA, DOT);
    const initialDotAusdLP = await instance.getLiquidityPool(DOT, AUSD);

    console.log("Initial ACA - AUSD liquidity pool: %s ACA - %s AUSD", formatUnits(initialAcaAusdLP[0].toString(), 12), formatUnits(initialAcaAusdLP[1].toString(), acaDecimals));
    console.log("Initial ACA - DOT liquidity pool: %s ACA - %s DOT", formatUnits(initialAcaDotLP[0].toString(), 12), formatUnits(initialAcaDotLP[1].toString(), ausdDecimals));
    console.log("Initial DOT - AUSD liquidity pool: %s DOT - %s AUSD", formatUnits(initialDotAusdLP[0].toString(), 12), formatUnits(initialDotAusdLP[1].toString(), dotDecimals));
    console.log("");

    console.log("Getting liquidity pools");

    const initialAcaAusdLP = await instance.getLiquidityPool(ACA, AUSD);
    const initialAcaDotLP = await instance.getLiquidityPool(ACA, DOT);
    const initialDotAusdLP = await instance.getLiquidityPool(DOT, AUSD);

    console.log("Initial ACA - AUSD liquidity pool: %s ACA - %s AUSD", formatUnits(initialAcaAusdLP[0].toString(), 12), formatUnits(initialAcaAusdLP[1].toString(), acaDecimals));
    console.log("Initial ACA - DOT liquidity pool: %s ACA - %s DOT", formatUnits(initialAcaDotLP[0].toString(), 12), formatUnits(initialAcaDotLP[1].toString(), ausdDecimals));
    console.log("Initial DOT - AUSD liquidity pool: %s DOT - %s AUSD", formatUnits(initialDotAusdLP[0].toString(), 12), formatUnits(initialDotAusdLP[1].toString(), dotDecimals));

    console.log("");
    console.log("");

    console.log("Getting liquidity pool token addresses");

    const acaAusdLPTokenAddress = await instance.getLiquidityTokenAddress(ACA, AUSD);
    const acaDotLPTokenAddress = await instance.getLiquidityTokenAddress(ACA, DOT);
    const dotAusdLPTokenAddress = await instance.getLiquidityTokenAddress(DOT, AUSD);

    console.log("Liquidity pool token address for ACA - AUSD:", acaAusdLPTokenAddress);
    console.log("Liquidity pool token address for ACA - DOT:", acaDotLPTokenAddress);
    console.log("Liquidity pool token address for DOT - AUSD:", dotAusdLPTokenAddress);
```

Now that we have queried all of the informative values, we can query the swap target amounts, so we know what to expect after swapping with exact supply. In order to do both, we need to define the swap paths. First path will be used to swap ACA for AUSD and the second one will be used to swap ACA for DOT. We are able to do the former directly, while we need to swap ACA for AUSD in order to be able to then swap AUSD for DOT. Direct swap of ACA for DOT is not possible. After we get the information of how much of AUSD or DOT we can expect after swapping with exact supply, we can swap the tokens using the already defined paths. Finally we can log the results to the console as well as the comparisons between the expected and actual changes in balance of the tokens:

```javascript
    console.log("");
    console.log("");

    console.log("Getting expected swap target amounts");

    const path1 = [ACA, AUSD];
    const path2 = [ACA, AUSD, DOT];
    const supply = initialAcaBalance.div(web3.utils.toBN(1000));

    const expectedTarget1 = await instance.getSwapTargetAmount(path1, supply);
    const expectedTarget2 = await instance.getSwapTargetAmount(path2, supply);

    console.log("Expected target when using path ACA -> AUSD: %s AUSD", formatUnits(expectedTarget1.toString(), ausdDecimals));
    console.log("Expected target when using path ACA -> AUSD -> DOT: %s DOT", formatUnits(expectedTarget2.toString(), dotDecimals));

    console.log("");
    console.log("");

    console.log("Swapping with exact supply");

    await instance.swapWithExactSupply(path1, supply, 1);
    await instance.swapWithExactSupply(path2, supply, 1);

    const halfwayAcaBalance = await ACAinstance.balanceOf(deployer);
    const halfwayAusdBalance = await AUSDinstance.balanceOf(deployer);
    const halfwayDotBalance = await DOTinstance.balanceOf(deployer);

    console.log("Halfway %s ACA balance: %s ACA", deployer, formatUnits(halfwayAcaBalance.toString(), acaDecimals));
    console.log("Halfway %s AUSD balance: %s AUSD", deployer, formatUnits(halfwayAusdBalance.toString(), ausdDecimals));
    console.log("Halfway %s DOT balance: %s DOT", deployer, formatUnits(halfwayDotBalance.toString(), dotDecimals));

    console.log("%s AUSD balance increase was %s AUSD, while the expected increase was %s AUSD.", deployer, formatUnits(halfwayAusdBalance.sub(initialAusdBalance).toString(), 12), formatUnits(expectedTarget1.toString(), ausdDecimals));
    console.log("%s DOT balance increase was %s DOT, while the expected increase was %s DOT.", deployer, formatUnits(halfwayDotBalance.sub(initialDotBalance).toString(), 12), formatUnits(expectedTarget2.toString(), dotDecimals));
```

Another way of swapping the tokens is with exact target. Here we have to set the maximum amount of supply we are willing to swap and the exact target of the egress token we would like to receive. To estimate what our maximum supply should be, we can first estimate the expected supply needed in order to get the exact target out of the swap. In order to get 10 of AUSD and 10 of DOT, we need to use the `parseUnits` utility, to properly format the desired target values, which both have 12 decimal spaces. After we get the estimated required supplies for each swap, we can log those to the console as well as use them as the maximum supply when swapping with exact target. As it is possible, that the required maximum supply increases, we add 1 ACA when swapping using the first path and 10 ACA, when swapping with the second path. When the swap is executed, we can query the token balances once more and log them as well as the comparisons with the expected changes to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Getting expected supply amount");

    const targetAusd = web3.utils.toBN(parseUnits("10", ausdDecimals));
    const targetDot = web3.utils.toBN(parseUnits("10", dotDecimals));

    const expectedSupply1 = await instance.getSwapSupplyAmount(path1, targetAusd);
    const expectedSupply2 = await instance.getSwapSupplyAmount(path2, targetDot);

    console.log("Expected supply for getting %s AUSD in order to reach a total of %s AUSD is %s ACA.", formatUnits(targetAusd.toString(), ausdDecimals), formatUnits(targetAusd.add(halfwayAusdBalance).toString(), ausdDecimals), formatUnits(expectedSupply1.toString(), acaDecimals));
    console.log("Expected supply for getting %s DOT in order to reach a total of %s DOT is %s ACA.", formatUnits(targetDot.toString(), dotDecimals), formatUnits(targetDot.add(halfwayDotBalance).toString(), dotDecimals), formatUnits(expectedSupply2.toString(), acaDecimals));

    console.log("");
    console.log("");

    console.log("Swapping with exact target");

    await instance.swapWithExactTarget(path1, targetAusd, expectedSupply1.add(web3.utils.toBN(parseUnits("1", ausdDecimals))));
    await instance.swapWithExactTarget(path2, targetDot, expectedSupply2.add(web3.utils.toBN(parseUnits("1", dotDecimals))));

    const finalAcaBalance = await ACAinstance.balanceOf(deployer);
    const finalAusdBalance = await AUSDinstance.balanceOf(deployer);
    const finalDotBalance = await DOTinstance.balanceOf(deployer);

    console.log("Final %s ACA balance: %s ACA", deployer, formatUnits(finalAcaBalance.toString(), acaDecimals));
    console.log("Final %s AUSD balance: %s AUSD", deployer, formatUnits(finalAusdBalance.toString(), ausdDecimals));
    console.log("Final %s DOT balance: %s DOT", deployer, formatUnits(finalDotBalance.toString(), dotDecimals));

    console.log("AUSD balance has increased by %s AUSD, while the expected increase was %s AUSD.", formatUnits(finalAusdBalance.sub(halfwayAusdBalance).toString(), ausdDecimals), formatUnits(targetAusd.toString(), ausdDecimals));
    console.log("DOT balance has increased by %s DOT, while the expected increase was %s DOT.", formatUnits(finalDotBalance.sub(halfwayDotBalance).toString(), dotDecimals), formatUnits(targetDot.toString(), dotDecimals));

    console.log("Expected decrease of ACA balance was %s ACA, while the actual decrease was %s ACA.", formatUnits(expectedSupply1.add(expectedSupply2).toString(), acaDecimals), formatUnits(halfwayAcaBalance.sub(finalAcaBalance).toString(), acaDecimals));
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

```javascript
const DEXContract = artifacts.require("@acala-network/contracts/build/contracts/DEX");
const TokenContract = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { ACA, AUSD, DEX, DOT } = require("@acala-network/contracts/utils/Address");
const { formatUnits, parseUnits } = require("ethers/lib/utils");


module.exports = async function(callback) {
  try {
    console.log("");
    console.log("");

    const accounts = await web3.eth.getAccounts();
    const deployer = accounts[0];
    
    console.log("Interacting with DEX using account:", deployer);

    console.log(`Initial account balance: ${formatUnits((await web3.eth.getBalance(deployer)).toString(), 12)} ACA`);

    console.log("");
    console.log("");

    console.log("Instantiating DEX and token smart contracts");

    const instance = await DEXContract.at(DEX);
    const ACAinstance = await TokenContract.at(ACA);
    const AUSDinstance = await TokenContract.at(AUSD);
    const DOTinstance = await TokenContract.at(DOT);

    console.log("DEX instantiated with address", instance.address);
    console.log("ACA token instantiated with address", ACAinstance.address);
    console.log("AUSD token instantiated with address", AUSDinstance.address);
    console.log("DOT token instantiated with address", DOTinstance.address);

    console.log("");
    console.log("");

    console.log("Getting inital token balances");

    const initialAcaBalance = await ACAinstance.balanceOf(deployer);
    const initialAusdBalance = await AUSDinstance.balanceOf(deployer);
    const initialDotBalance = await DOTinstance.balanceOf(deployer);

    const acaDecimals = (await ACAinstance.decimals()).toNumber();
    const ausdDecimals = (await AUSDinstance.decimals()).toNumber();
    const dotDecimals = (await DOTinstance.decimals()).toNumber();

    console.log("Inital %s ACA balance: %s ACA", deployer, formatUnits(initialAcaBalance.toString(), acaDecimals));
    console.log("Inital %s AUSD balance: %s AUSD", deployer, formatUnits(initialAusdBalance.toString(), ausdDecimals));
    console.log("Inital %s DOT balance: %s DOT", deployer, formatUnits(initialDotBalance.toString(), dotDecimals));

    console.log("");
    console.log("");

    console.log("Getting liquidity pools");

    const initialAcaAusdLP = await instance.getLiquidityPool(ACA, AUSD);
    const initialAcaDotLP = await instance.getLiquidityPool(ACA, DOT);
    const initialDotAusdLP = await instance.getLiquidityPool(DOT, AUSD);

    console.log("Initial ACA - AUSD liquidity pool: %s ACA - %s AUSD", formatUnits(initialAcaAusdLP[0].toString(), 12), formatUnits(initialAcaAusdLP[1].toString(), acaDecimals));
    console.log("Initial ACA - DOT liquidity pool: %s ACA - %s DOT", formatUnits(initialAcaDotLP[0].toString(), 12), formatUnits(initialAcaDotLP[1].toString(), ausdDecimals));
    console.log("Initial DOT - AUSD liquidity pool: %s DOT - %s AUSD", formatUnits(initialDotAusdLP[0].toString(), 12), formatUnits(initialDotAusdLP[1].toString(), dotDecimals));

    console.log("");
    console.log("");

    console.log("Getting liquidity pool token addresses");

    const acaAusdLPTokenAddress = await instance.getLiquidityTokenAddress(ACA, AUSD);
    const acaDotLPTokenAddress = await instance.getLiquidityTokenAddress(ACA, DOT);
    const dotAusdLPTokenAddress = await instance.getLiquidityTokenAddress(DOT, AUSD);

    console.log("Liquidity pool token address for ACA - AUSD:", acaAusdLPTokenAddress);
    console.log("Liquidity pool token address for ACA - DOT:", acaDotLPTokenAddress);
    console.log("Liquidity pool token address for DOT - AUSD:", dotAusdLPTokenAddress);

    console.log("");
    console.log("");

    console.log("Getting expected swap target amounts");

    const path1 = [ACA, AUSD];
    const path2 = [ACA, AUSD, DOT];
    const supply = initialAcaBalance.div(web3.utils.toBN(1000));

    const expectedTarget1 = await instance.getSwapTargetAmount(path1, supply);
    const expectedTarget2 = await instance.getSwapTargetAmount(path2, supply);

    console.log("Expected target when using path ACA -> AUSD: %s AUSD", formatUnits(expectedTarget1.toString(), ausdDecimals));
    console.log("Expected target when using path ACA -> AUSD -> DOT: %s DOT", formatUnits(expectedTarget2.toString(), dotDecimals));

    console.log("");
    console.log("");

    console.log("Swapping with exact supply");

    await instance.swapWithExactSupply(path1, supply, 1);
    await instance.swapWithExactSupply(path2, supply, 1);

    const halfwayAcaBalance = await ACAinstance.balanceOf(deployer);
    const halfwayAusdBalance = await AUSDinstance.balanceOf(deployer);
    const halfwayDotBalance = await DOTinstance.balanceOf(deployer);

    console.log("Halfway %s ACA balance: %s ACA", deployer, formatUnits(halfwayAcaBalance.toString(), acaDecimals));
    console.log("Halfway %s AUSD balance: %s AUSD", deployer, formatUnits(halfwayAusdBalance.toString(), ausdDecimals));
    console.log("Halfway %s DOT balance: %s DOT", deployer, formatUnits(halfwayDotBalance.toString(), dotDecimals));

    console.log("%s AUSD balance increase was %s AUSD, while the expected increase was %s AUSD.", deployer, formatUnits(halfwayAusdBalance.sub(initialAusdBalance).toString(), 12), formatUnits(expectedTarget1.toString(), ausdDecimals));
    console.log("%s DOT balance increase was %s DOT, while the expected increase was %s DOT.", deployer, formatUnits(halfwayDotBalance.sub(initialDotBalance).toString(), 12), formatUnits(expectedTarget2.toString(), dotDecimals));

    console.log("");
    console.log("");

    console.log("Getting expected supply amount");

    const targetAusd = web3.utils.toBN(parseUnits("10", ausdDecimals));
    const targetDot = web3.utils.toBN(parseUnits("10", dotDecimals));

    const expectedSupply1 = await instance.getSwapSupplyAmount(path1, targetAusd);
    const expectedSupply2 = await instance.getSwapSupplyAmount(path2, targetDot);

    console.log("Expected supply for getting %s AUSD in order to reach a total of %s AUSD is %s ACA.", formatUnits(targetAusd.toString(), ausdDecimals), formatUnits(targetAusd.add(halfwayAusdBalance).toString(), ausdDecimals), formatUnits(expectedSupply1.toString(), acaDecimals));
    console.log("Expected supply for getting %s DOT in order to reach a total of %s DOT is %s ACA.", formatUnits(targetDot.toString(), dotDecimals), formatUnits(targetDot.add(halfwayDotBalance).toString(), dotDecimals), formatUnits(expectedSupply2.toString(), acaDecimals));

    console.log("");
    console.log("");

    console.log("Swapping with exact target");

    await instance.swapWithExactTarget(path1, targetAusd, expectedSupply1.add(web3.utils.toBN(parseUnits("1", ausdDecimals))));
    await instance.swapWithExactTarget(path2, targetDot, expectedSupply2.add(web3.utils.toBN(parseUnits("1", dotDecimals))));

    const finalAcaBalance = await ACAinstance.balanceOf(deployer);
    const finalAusdBalance = await AUSDinstance.balanceOf(deployer);
    const finalDotBalance = await DOTinstance.balanceOf(deployer);

    console.log("Final %s ACA balance: %s ACA", deployer, formatUnits(finalAcaBalance.toString(), acaDecimals));
    console.log("Final %s AUSD balance: %s AUSD", deployer, formatUnits(finalAusdBalance.toString(), ausdDecimals));
    console.log("Final %s DOT balance: %s DOT", deployer, formatUnits(finalDotBalance.toString(), dotDecimals));

    console.log("AUSD balance has increased by %s AUSD, while the expected increase was %s AUSD.", formatUnits(finalAusdBalance.sub(halfwayAusdBalance).toString(), ausdDecimals), formatUnits(targetAusd.toString(), ausdDecimals));
    console.log("DOT balance has increased by %s DOT, while the expected increase was %s DOT.", formatUnits(finalDotBalance.sub(halfwayDotBalance).toString(), dotDecimals), formatUnits(targetDot.toString(), dotDecimals));

    console.log("Expected decrease of ACA balance was %s ACA, while the actual decrease was %s ACA.", formatUnits(expectedSupply1.add(expectedSupply2).toString(), acaDecimals), formatUnits(halfwayAcaBalance.sub(finalAcaBalance).toString(), acaDecimals));

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

To use the script within the local development network or a public development network, you need to add the following scripts to `scripts` section of your `package.json`:

```json
    "user-journey-mandala": "truffle exec scripts/userJourney.js --network mandala",
    "user-journey-mandala:pubDev": "truffle exec scripts/userJourney.js --network mandalaPublicDev"
```

Running the `yarn user-journey-mandala` script should return the following output:

```shell
yarn user-journey-mandala


yarn run v1.22.17
$ truffle exec scripts/userJourney.js --network mandala
Using network 'mandala'.



Interacting with DEX using account: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Initial account balance: 9323913680447.913162 ACA


Instantiating DEX and token smart contracts
DEX instantiated with address 0x0000000000000000000000000000000000000803
ACA token instantiated with address 0x0000000000000000000100000000000000000000
AUSD token instantiated with address 0x0000000000000000000100000000000000000001
DOT token instantiated with address 0x0000000000000000000100000000000000000002


Getting inital token balances
Inital 0x75E480dB528101a381Ce68544611C169Ad7EB342 ACA balance: 9323913.680447913162 ACA
Inital 0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance: 10406596.492912849269 AUSD
Inital 0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance: 1000571170.9265896179 DOT


Getting liquidity pools
Initial ACA - AUSD liquidity pool: 1677078.322034843658 ACA - 1193187.339725485249 AUSD
Initial ACA - DOT liquidity pool: 0.0 ACA - 0.0 DOT
Initial DOT - AUSD liquidity pool: 14288.290734103821 DOT - 140021616.7361665482 AUSD


Getting liquidity pool token addresses
Liquidity pool token address for ACA - AUSD: 0x0000000000000000000200000000000000000001
Liquidity pool token address for ACA - DOT: 0x0000000000000000000200000000000000000002
Liquidity pool token address for DOT - AUSD: 0x0000000000000000000200000000010000000002


Getting expected swap target amounts
Expected target when using path ACA -> AUSD: 6590.427715075078 AUSD
Expected target when using path ACA -> AUSD -> DOT: 6686.9335368891 DOT


Swapping with exact supply
Halfway 0x75E480dB528101a381Ce68544611C169Ad7EB342 ACA balance: 9305265.7621934171 ACA
Halfway 0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance: 10413186.920627924347 AUSD
Halfway 0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance: 1000577784.7007857732 DOT
0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance increase was 6590.427715075078 AUSD, while the expected increase was 6590.427715075078 AUSD.
0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance increase was 66.137741961553 DOT, while the expected increase was 6686.9335368891 DOT.


Getting expected supply amount
Expected supply for getting 10.0 AUSD in order to reach a total of 10413196.920627924347 AUSD is 14.384105366502 ACA.
Expected supply for getting 10.0 DOT in order to reach a total of 1000577794.7007857732 DOT is 14.241871766023 ACA.


Swapping with exact target
Final 0x75E480dB528101a381Ce68544611C169Ad7EB342 ACA balance: 9305237.045081274186 ACA
Final 0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance: 10413196.920627924347 AUSD
Final 0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance: 1000577794.7007857732 DOT
AUSD balance has increased by 10.0 AUSD, while the expected increase was 10.0 AUSD.
DOT balance has increased by 10.0 DOT, while the expected increase was 10.0 DOT.
Expected decrease of ACA balance was 28.625977132525 ACA, while the actual decrease was 28.717112142914 ACA.


User journey completed!
✨  Done in 7.37s.
```

## Summary

We have built upon the knowledge on how to interact with the Acala EVM+ and gotten familiar with Acala EVM+ precompiles and predeploys. To run the test we can use the `yarn test-mandala` and `yarn test-madala:pubDev` and to run the user journey script we can use the `yarn user-journey-mandala` and `yarn user-journey-mandala:pubDev`. As we are using utilities only available in the Acala EVM+, we can no longer use a conventional development network like Ganache.
