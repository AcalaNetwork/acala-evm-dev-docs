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

This example introduces the use of Acala EVM+ predeployed DEX that is present on every network at a fixed address (the address of a predeployed contract is the same on a local development network, public test network as well as the production network). As this example focuses on showcasing the interactions with the predeployed `DEX`, it doesn't have its own smart conract. We will get all of the required imports from the [`@acala-network/contracts`](https://github.com/AcalaNetwork/predeploy-contracts) dependency. The precompiles and predeploys are a specific feature of the Acala EVM+, so this tutorial is no longer compatible with traditional EVM development networks (like Ganache) or with the Hardhat's built in network emulator.

Let's take a look!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/DEX](https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/DEX)
{% endhint %}

## Smart contract

As mentioned in the introduction, this tutorial doesn't include the smart contract, so the `contracts` folder can be removed as well. We will however be using the `@acala-network/contracts` dependency in order to gain access to the precompiled resources of the `DEX` smart contract.

## Test

Tests for this tutorial will validate the expected values returned by the DEX predeployed smart contract. The test file in our case is called `DEX.js`. Within it we import the `expect` from `chai` dependency and `Contract` from the `ethers` dependency. We are using `Contract` in stead of `ContractFactory`, because the contract is already deployed to the network. The `ACA`, `AUSD`, `LP_ACA_AUSD`, `DOT`, `RENBTC` and `DEX`, which are exports from the `ADDRESS` utility of `@acala-network/contracts` dependency, are imported and they hold the values of the addresses of the corresponding smart contracts. Additionally we are importing the compiled `DEX` smart contract from the `@acala-network/contracts` dependency, which we will use to instantiate the smart contract. `Token` precompile is imported from `@acala-network/contracts` so that we can instantiate the predeployed token smart contracts and validate the balance changes after interacting with the DEX.

{% hint style="info" %}
**NOTE: Since the ACA ERC20 token mirrors the balance of the native ACA currency, we can not expect that sending **_**x**_** amount of ACA into the DEX, will decrease our ACA balance by that exact amount. Some of it will also be used to pay for the transaction fees. We have to keep this in mind while writing the tests and scripts.**
{% endhint %}

The test file with import statements and an empty test should look like this:

```javascript
const { expect } = require("chai");
const { Contract } = require("ethers");
const { ACA, AUSD, LP_ACA_AUSD, DOT, RENBTC, DEX } = require("@acala-network/contracts/utils/MandalaAddress");

const DEXContract = require("@acala-network/contracts/build/contracts/DEX.json");
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");
const { parseUnits } = require("@acala-network/eth-providers/node_modules/@ethersproject/units");
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

describe("DEX contract", function () {

});
```

To prepare for the testing, we have to define the global variables, `instance`, `ACAinstance`, `AUSDinstance`, `deployer` and `deployerAddress`. The `instance` will store the predeployed DEX smart contract instance, the `ACAinstance` and `AUSDinstance` variables will store their respective ERC20 predeployed token smart contracts. The `deployer` will store `Signer` and the `deployerAddress` will store its address. Let's assign these values in the `beforeEach` action:

```javascript
        let instance;
        let ACAinstance;
        let AUSDinstance;
        let deployer;
        let deployerAddress;

        beforeEach(async function () {
                [deployer] = await ethers.getSigners();
                deployerAddress = await deployer.getAddress();
                instance = new Contract(DEX, DEXContract.abi, deployer);
                ACAinstance = new Contract(ACA, TokenContract.abi, deployer);
                AUSDinstance = new Contract(AUSD, TokenContract.abi, deployer);
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

Before we add the inner describe blocks within the `Operation` describe block, we should increase the timeout for this test to 50s, to make sure that the tests can be run on the public test network in addition to the local development network:

```javascript
        describe("Operation", function () {
                this.timeout(50000);

                describe("getLiquidityPool", function () {
                        
                });

                describe("getLiquidityTokenAddress", function (){
                        
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
                                await expect(instance.getLiquidityPool(NULL_ADDRESS, ACA)).to
                                        .be.revertedWith("DEX: tokenA is zero address");
                        });

                        it("should not allow tokenB to be a 0x0 address", async function () {
                                await expect(instance.getLiquidityPool(ACA, NULL_ADDRESS)).to
                                        .be.revertedWith("DEX: tokenB is zero address");
                        });

                        it("should return 0 liquidity for nonexistent pair", async function () {
                                const response = await instance.getLiquidityPool(ACA, DOT);

                                const liquidityA = response[0];
                                const liquidityB = response[1];

                                expect(liquidityA).to.equal(0);
                                expect(liquidityB).to.equal(0);
                        });

                        it("should return liquidity for existing pairs", async function () {
                                const response = await instance.getLiquidityPool(ACA, AUSD);

                                const liquidityA = response[0];
                                const liquidityB = response[1];

                                expect(liquidityA).to.be.above(0);
                                expect(liquidityB).to.be.above(0);
                        });
```

When validating the `getLiquidityTokenAddress` function, we will check for the following examples:

1. `tokenA` should not be a `0x0` address.
2. `tokenB` should not be a `0x0` address.
3. Liquidity token address should be returned for the existing pairs.

The section should look like this:

```javascript
                        it("should not allow tokenA to be a 0x0 address", async function () {
                                await expect(instance.getLiquidityTokenAddress(NULL_ADDRESS, ACA)).to
                                        .be.revertedWith("DEX: tokenA is zero address");
                        });

                        it("should not allow tokenB to be a 0x0 address", async function () {
                                await expect(instance.getLiquidityTokenAddress(ACA, NULL_ADDRESS)).to
                                        .be.revertedWith("DEX: tokenB is zero address");
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
                                let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

                                await expect(instance.getSwapTargetAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                                
                                path = [ACA, NULL_ADDRESS, DOT, RENBTC];

                                await expect(instance.getSwapTargetAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, NULL_ADDRESS, RENBTC];

                                await expect(instance.getSwapTargetAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, RENBTC, NULL_ADDRESS];

                                await expect(instance.getSwapTargetAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        });

                        it("should not allow supplyAmount to be 0", async function () {
                                await expect(instance.getSwapTargetAmount([ACA, DOT], 0)).to
                                        .be.revertedWith("DEX: supplyAmount is zero");
                        });

                        it("should return 0 for an incompatible path", async function () {
                                const response = await instance.getSwapTargetAmount([ACA, DOT], 100);

                                expect(response.toString()).to.equal('0');
                        })

                        it("should return a swap target amount", async function () {
                                const response = await instance.getSwapTargetAmount([ACA, AUSD], 100);

                                expect(response).to.be.above(0);
                        });
```

When validating the `getSwapSupplyAmount` function, we will check for the following examples:

1. `path` should not include the `0x0` address.
2. `targetAmount` should not be 0.
3. Getting swap supply amount should revert for an incompatible path.
4. Swap supply amount should be returned when all parameters are correct.

The section should look like this:

```javascript
                        it("should not allow an address in the path to be a 0x0 address", async function () {
                                let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

                                await expect(instance.getSwapSupplyAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                                
                                path = [ACA, NULL_ADDRESS, DOT, RENBTC];

                                await expect(instance.getSwapSupplyAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, NULL_ADDRESS, RENBTC];

                                await expect(instance.getSwapSupplyAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, RENBTC, NULL_ADDRESS];

                                await expect(instance.getSwapSupplyAmount(path, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        });

                        it("should not allow targetAmount to be 0", async function () {
                                await expect(instance.getSwapSupplyAmount([ACA, AUSD], 0)).to
                                        .be.revertedWith("DEX: targetAmount is zero");
                        });

                        it("should return 0 for an incompatible path", async function () {
                                const response = await instance.getSwapSupplyAmount([ACA, DOT], 100);

                                expect(response.toString()).to.equal('0');
                        });

                        it("should return the supply amount", async function () {
                                const response = await instance.getSwapSupplyAmount([ACA, AUSD], 100);

                                expect(response).to.be.above(0);
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
                                let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

                                await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to
                                        .be.revertedWith("DEX: token is zero address");
                                
                                path = [ACA, NULL_ADDRESS, DOT, RENBTC];

                                await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, NULL_ADDRESS, RENBTC];

                                await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, RENBTC, NULL_ADDRESS];

                                await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to
                                        .be.revertedWith("DEX: token is zero address");
                        });

                        it("should not allow supplyAmount to be 0", async function () {
                                await expect(instance.swapWithExactSupply([ACA, AUSD], 0, 1)).to
                                        .be.revertedWith("DEX: supplyAmount is zero");
                        });

                        it("should allocate the tokens to the caller", async function () {
                                const initalBalance = await ACAinstance.balanceOf(deployerAddress);
                                const initBal = await AUSDinstance.balanceOf(deployerAddress);
                                const path = [ACA, AUSD];
                                const expected_target = await instance.getSwapTargetAmount(path, 100);

                                await instance.connect(deployer).swapWithExactSupply(path, 100, 1);

                                const finalBalance = await ACAinstance.balanceOf(deployerAddress);
                                const finBal = await AUSDinstance.balanceOf(deployerAddress);

                                // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
                                expect(finalBalance).to.be.below(initalBalance.sub(100));
                                expect(finBal).to.equal(initBal.add(expected_target));
                        });

                        it("should emit a Swaped event", async function () {
                                const path = [ACA, AUSD];
                                const expected_target = await instance.getSwapTargetAmount(path, 100);

                                await expect(instance.connect(deployer).swapWithExactSupply(path, 100, 1)).to
                                        .emit(instance, "Swaped")
                                        .withArgs(deployerAddress, path, 100, expected_target);
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
                                let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

                                await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                                
                                path = [ACA, NULL_ADDRESS, DOT, RENBTC];

                                await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, NULL_ADDRESS, RENBTC];

                                await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        
                                path = [ACA, DOT, RENBTC, NULL_ADDRESS];

                                await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to
                                        .be.revertedWith("DEX: token is zero address");
                        });

                        it("should not allow targetAmount to be 0", async function () {
                                await expect(instance.swapWithExactTarget([ACA, AUSD], 0, 1234567890)).to
                                        .be.revertedWith("DEX: targetAmount is zero");
                        });

                        it("should allocate tokens to the caller", async function () {
                                const initalBalance = await ACAinstance.balanceOf(deployerAddress);
                                const initBal = await AUSDinstance.balanceOf(deployerAddress);
                                const path = [ACA, AUSD];
                                const expected_supply = await instance.getSwapSupplyAmount(path, 100);

                                await instance.connect(deployer).swapWithExactTarget(path, 100, 1234567890);

                                const finalBalance = await ACAinstance.balanceOf(deployerAddress);
                                const finBal = await AUSDinstance.balanceOf(deployerAddress);

                                // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
                                expect(finalBalance).to.be.below(initalBalance.sub(expected_supply));
                                expect(finBal).to.equal(initBal.add(100));
                        });

                        it("should emit Swaped event", async function () {
                                const path = [ACA, AUSD];
                                const expected_supply = await instance.getSwapSupplyAmount(path, 100);

                                await expect(instance.connect(deployer).swapWithExactTarget(path, 100, 1234567890)).to
                                        .emit(instance, "Swaped")
                                        .withArgs(deployerAddress, path, expected_supply, 100);
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
                                await expect(instance.addLiquidity(NULL_ADDRESS, AUSD, 1000, 1000, 1)).to
                                        .be.revertedWith("DEX: tokenA is zero address");
                        });

                        it("should not allow tokenB to be 0x0 address", async function () {
                                await expect(instance.addLiquidity(ACA, NULL_ADDRESS, 1000, 1000, 1)).to
                                        .be.revertedWith("DEX: tokenB is zero address");
                        });

                        it("should not allow maxAmountA to be 0", async function () {
                                await expect(instance.addLiquidity(ACA, AUSD, 0, 1000, 1)).to
                                        .be.revertedWith("DEX: maxAmountA is zero");
                        });

                        it("should not allow maxAmountB to be 0", async function () {
                                await expect(instance.addLiquidity(ACA, AUSD, 1000, 0, 1)).to
                                        .be.revertedWith("DEX: maxAmountB is zero");
                        });
                        
                        it("should increase liquidity", async function () {
                                const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

                                await instance.addLiquidity(ACA, AUSD, parseUnits("2", 12), parseUnits("2", 12), 1);

                                const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

                                expect(finalLiquidity[0]).to.be.above(intialLiquidity[0]);
                                expect(finalLiquidity[1]).to.be.above(intialLiquidity[1]);
                        });

                        it("should emit AddedLiquidity event", async function () {
                                await expect(instance.connect(deployer).addLiquidity(ACA, AUSD, 1000, 1000, 1)).to
                                        .emit(instance, "AddedLiquidity")
                                        .withArgs(deployerAddress, ACA, AUSD, 1000, 1000);
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
                                await expect(instance.removeLiquidity(NULL_ADDRESS, AUSD, 1, 0, 0)).to
                                        .be.revertedWith("DEX: tokenA is zero address");
                        });

                        it("should not allow tokenB to be a 0x0 address", async function () {
                                await expect(instance.removeLiquidity(ACA, NULL_ADDRESS, 1, 0, 0)).to
                                        .be.revertedWith("DEX: tokenB is zero address");
                        });

                        it("should not allow removeShare to be 0", async function () {
                                await expect(instance.removeLiquidity(ACA, AUSD, 0, 0, 0)).to
                                        .be.revertedWith("DEX: removeShare is zero");
                        });

                        it("should reduce the liquidity", async function () {
                                const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

                                await instance.removeLiquidity(ACA, AUSD, 10, 1, 1);

                                const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

                                expect(finalLiquidity[0]).to.be.below(intialLiquidity[0]);
                                expect(finalLiquidity[1]).to.be.below(intialLiquidity[1]);
                        });

                        it("should emit RemovedLiquidity event", async function () {
                                await expect(instance.connect(deployer).removeLiquidity(ACA, AUSD, 1, 0, 0)).to
                                        .emit(instance, "RemovedLiquidity")
                                        .withArgs(deployerAddress, ACA, AUSD, 1);
                        });
```

With that, our test is ready to be run.

<details>

<summary>Your test/DEX.js should look like this:</summary>

```javascript
const { expect } = require('chai');
const { Contract } = require('ethers');
const { ACA, AUSD, LP_ACA_AUSD, DOT, RENBTC, DEX } = require('@acala-network/contracts/utils/MandalaAddress');

const DEXContract = require('@acala-network/contracts/build/contracts/DEX.json');
const TokenContract = require('@acala-network/contracts/build/contracts/Token.json');
const { parseUnits } = require('@ethersproject/units');
const NULL_ADDRESS = '0x0000000000000000000000000000000000000000';

describe('DEX contract', function () {
  let instance;
  let ACAinstance;
  let AUSDinstance;
  let deployer;
  let deployerAddress;

  beforeEach(async function () {
    [deployer] = await ethers.getSigners();
    deployerAddress = await deployer.getAddress();
    instance = new Contract(DEX, DEXContract.abi, deployer);
    ACAinstance = new Contract(ACA, TokenContract.abi, deployer);
    AUSDinstance = new Contract(AUSD, TokenContract.abi, deployer);
  });

  describe('Operation', function () {
    this.timeout(50000);

    describe('getLiquidityPool', function () {
      it('should not allow tokenA to be a 0x0 address', async function () {
        await expect(instance.getLiquidityPool(NULL_ADDRESS, ACA)).to.be.revertedWith('DEX: tokenA is zero address');
      });

      it('should not allow tokenB to be a 0x0 address', async function () {
        await expect(instance.getLiquidityPool(ACA, NULL_ADDRESS)).to.be.revertedWith('DEX: tokenB is zero address');
      });

      it('should return 0 liquidity for nonexistent pair', async function () {
        const response = await instance.getLiquidityPool(ACA, DOT);

        const liquidityA = response[0];
        const liquidityB = response[1];

        expect(liquidityA).to.equal(0);
        expect(liquidityB).to.equal(0);
      });

      it('should return liquidity for existing pairs', async function () {
        const response = await instance.getLiquidityPool(ACA, AUSD);

        const liquidityA = response[0];
        const liquidityB = response[1];

        expect(liquidityA).to.be.above(0);
        expect(liquidityB).to.be.above(0);
      });
    });

    describe('getLiquidityTokenAddress', function () {
      it('should not allow tokenA to be a 0x0 address', async function () {
        await expect(instance.getLiquidityTokenAddress(NULL_ADDRESS, ACA)).to.be.revertedWith(
          'DEX: tokenA is zero address'
        );
      });

      it('should not allow tokenB to be a 0x0 address', async function () {
        await expect(instance.getLiquidityTokenAddress(ACA, NULL_ADDRESS)).to.be.revertedWith(
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
        let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

        await expect(instance.getSwapTargetAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');

        path = [ACA, NULL_ADDRESS, DOT, RENBTC];

        await expect(instance.getSwapTargetAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');

        path = [ACA, DOT, NULL_ADDRESS, RENBTC];

        await expect(instance.getSwapTargetAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');

        path = [ACA, DOT, RENBTC, NULL_ADDRESS];

        await expect(instance.getSwapTargetAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');
      });

      it('should not allow supplyAmount to be 0', async function () {
        await expect(instance.getSwapTargetAmount([ACA, DOT], 0)).to.be.revertedWith('DEX: supplyAmount is zero');
      });

      it('should return 0 for an incompatible path', async function () {
        const response = await instance.getSwapTargetAmount([ACA, DOT], 100);
        expect(response.toString()).to.equal("0");
      });

      it('should return a swap target amount', async function () {
        const response = await instance.getSwapTargetAmount([ACA, AUSD], 100);

        expect(response).to.be.above(0);
      });
    });

    describe('getSwapSupplyAmount', function () {
      it('should not allow an address in the path to be a 0x0 address', async function () {
        let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

        await expect(instance.getSwapSupplyAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');

        path = [ACA, NULL_ADDRESS, DOT, RENBTC];

        await expect(instance.getSwapSupplyAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');

        path = [ACA, DOT, NULL_ADDRESS, RENBTC];

        await expect(instance.getSwapSupplyAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');

        path = [ACA, DOT, RENBTC, NULL_ADDRESS];

        await expect(instance.getSwapSupplyAmount(path, 12345678990)).to.be.revertedWith('DEX: token is zero address');
      });

      it('should not allow targetAmount to be 0', async function () {
        await expect(instance.getSwapSupplyAmount([ACA, AUSD], 0)).to.be.revertedWith('DEX: targetAmount is zero');
      });

      it('should return 0 for an incompatible path', async function () {
        const response = await instance.getSwapSupplyAmount([ACA, DOT], 100);
        expect(response.toString()).to.equal("0");
      });

      it('should return the supply amount', async function () {
        const response = await instance.getSwapSupplyAmount([ACA, AUSD], 100);

        expect(response).to.be.above(0);
      });
    });

    describe('swapWithExactSupply', function () {
      it('should not allow path to contain a 0x0 address', async function () {
        let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

        await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to.be.revertedWith(
          'DEX: token is zero address'
        );

        path = [ACA, NULL_ADDRESS, DOT, RENBTC];

        await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to.be.revertedWith(
          'DEX: token is zero address'
        );

        path = [ACA, DOT, NULL_ADDRESS, RENBTC];

        await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to.be.revertedWith(
          'DEX: token is zero address'
        );

        path = [ACA, DOT, RENBTC, NULL_ADDRESS];

        await expect(instance.swapWithExactSupply(path, 12345678990, 1)).to.be.revertedWith(
          'DEX: token is zero address'
        );
      });

      it('should not allow supplyAmount to be 0', async function () {
        await expect(instance.swapWithExactSupply([ACA, AUSD], 0, 1)).to.be.revertedWith('DEX: supplyAmount is zero');
      });

      it('should allocate the tokens to the caller', async function () {
        const initalBalance = await ACAinstance.balanceOf(deployerAddress);
        const initBal = await AUSDinstance.balanceOf(deployerAddress);
        const path = [ACA, AUSD];
        const expected_target = await instance.getSwapTargetAmount(path, 100);

        await instance.connect(deployer).swapWithExactSupply(path, 100, 1);

        const finalBalance = await ACAinstance.balanceOf(deployerAddress);
        const finBal = await AUSDinstance.balanceOf(deployerAddress);

        // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
        expect(finalBalance).to.be.below(initalBalance.sub(100));
        expect(finBal).to.equal(initBal.add(expected_target));
      });

      it('should emit a Swaped event', async function () {
        const path = [ACA, AUSD];
        const expected_target = await instance.getSwapTargetAmount(path, 100);

        await expect(instance.connect(deployer).swapWithExactSupply(path, 100, 1))
          .to.emit(instance, 'Swaped')
          .withArgs(deployerAddress, path, 100, expected_target);
      });
    });

    describe('swapWithExactTarget', function () {
      it('should not allow a token in a path to be a 0x0 address', async function () {
        let path = [NULL_ADDRESS, ACA, DOT, RENBTC];

        await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to.be.revertedWith(
          'DEX: token is zero address'
        );

        path = [ACA, NULL_ADDRESS, DOT, RENBTC];

        await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to.be.revertedWith(
          'DEX: token is zero address'
        );

        path = [ACA, DOT, NULL_ADDRESS, RENBTC];

        await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to.be.revertedWith(
          'DEX: token is zero address'
        );

        path = [ACA, DOT, RENBTC, NULL_ADDRESS];

        await expect(instance.swapWithExactTarget(path, 1, 12345678990)).to.be.revertedWith(
          'DEX: token is zero address'
        );
      });

      it('should not allow targetAmount to be 0', async function () {
        await expect(instance.swapWithExactTarget([ACA, AUSD], 0, 1234567890)).to.be.revertedWith(
          'DEX: targetAmount is zero'
        );
      });

      it('should allocate tokens to the caller', async function () {
        const initalBalance = await ACAinstance.balanceOf(deployerAddress);
        const initBal = await AUSDinstance.balanceOf(deployerAddress);
        const path = [ACA, AUSD];
        const expected_supply = await instance.getSwapSupplyAmount(path, 100);

        await instance.connect(deployer).swapWithExactTarget(path, 100, 1234567890);

        const finalBalance = await ACAinstance.balanceOf(deployerAddress);
        const finBal = await AUSDinstance.balanceOf(deployerAddress);

        // The following assertion needs to check for the balance to be below the initialBalance - 100, because some of the ACA balance is used to pay for the transaction fee.
        expect(finalBalance).to.be.below(initalBalance.sub(expected_supply));
        expect(finBal).to.equal(initBal.add(100));
      });

      it('should emit Swaped event', async function () {
        const path = [ACA, AUSD];
        const expected_supply = await instance.getSwapSupplyAmount(path, 100);

        await expect(instance.connect(deployer).swapWithExactTarget(path, 100, 1234567890))
          .to.emit(instance, 'Swaped')
          .withArgs(deployerAddress, path, expected_supply, 100);
      });
    });

    describe('addLiquidity', function () {
      it('should not allow tokenA to be 0x0 address', async function () {
        await expect(instance.addLiquidity(NULL_ADDRESS, AUSD, 1000, 1000, 1)).to.be.revertedWith(
          'DEX: tokenA is zero address'
        );
      });

      it('should not allow tokenB to be 0x0 address', async function () {
        await expect(instance.addLiquidity(ACA, NULL_ADDRESS, 1000, 1000, 1)).to.be.revertedWith(
          'DEX: tokenB is zero address'
        );
      });

      it('should not allow maxAmountA to be 0', async function () {
        await expect(instance.addLiquidity(ACA, AUSD, 0, 1000, 1)).to.be.revertedWith('DEX: maxAmountA is zero');
      });

      it('should not allow maxAmountB to be 0', async function () {
        await expect(instance.addLiquidity(ACA, AUSD, 1000, 0, 1)).to.be.revertedWith('DEX: maxAmountB is zero');
      });

      it('should increase liquidity', async function () {
        const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        await instance.addLiquidity(ACA, AUSD, parseUnits('2', 12), parseUnits('2', 12), 1);

        const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        expect(finalLiquidity[0]).to.be.above(intialLiquidity[0]);
        expect(finalLiquidity[1]).to.be.above(intialLiquidity[1]);
      });

      it('should emit AddedLiquidity event', async function () {
        await expect(instance.connect(deployer).addLiquidity(ACA, AUSD, 1000, 1000, 1))
          .to.emit(instance, 'AddedLiquidity')
          .withArgs(deployerAddress, ACA, AUSD, 1000, 1000);
      });
    });

    describe('removeLiquidity', function () {
      it('should not allow tokenA to be a 0x0 address', async function () {
        await expect(instance.removeLiquidity(NULL_ADDRESS, AUSD, 1, 0, 0)).to.be.revertedWith(
          'DEX: tokenA is zero address'
        );
      });

      it('should not allow tokenB to be a 0x0 address', async function () {
        await expect(instance.removeLiquidity(ACA, NULL_ADDRESS, 1, 0, 0)).to.be.revertedWith(
          'DEX: tokenB is zero address'
        );
      });

      it('should not allow removeShare to be 0', async function () {
        await expect(instance.removeLiquidity(ACA, AUSD, 0, 0, 0)).to.be.revertedWith('DEX: removeShare is zero');
      });

      it('should reduce the liquidity', async function () {
        const intialLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        await instance.removeLiquidity(ACA, AUSD, 10, 1, 1);

        const finalLiquidity = await instance.getLiquidityPool(ACA, AUSD);

        expect(finalLiquidity[0]).to.be.below(intialLiquidity[0]);
        expect(finalLiquidity[1]).to.be.below(intialLiquidity[1]);
      });

      it('should emit RemovedLiquidity event', async function () {
        await expect(instance.connect(deployer).removeLiquidity(ACA, AUSD, 1, 0, 0))
          .to.emit(instance, 'RemovedLiquidity')
          .withArgs(deployerAddress, ACA, AUSD, 1);
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
$ hardhat test test/DEX.js --network mandala


  DEX contract
    Operation
      getLiquidityPool
        ✓ should not allow tokenA to be a 0x0 address (49ms)
        ✓ should not allow tokenB to be a 0x0 address
        ✓ should return 0 liquidity for nonexistent pair
        ✓ should return liquidity for existing pairs
      getLiquidityTokenAddress
        ✓ should not allow tokenA to be a 0x0 address
        ✓ should not allow tokenB to be a 0x0 address
        ✓ should return liquidity token address for an existing pair
      getSwapTargetAddress
        ✓ should not allow for the path to include a 0x0 address (75ms)
        ✓ should not allow supplyAmount to be 0
        ✓ should return 0 for an incompatible path
        ✓ should return a swap target amount
      getSwapSupplyAmount
        ✓ should not allow an address in the path to be a 0x0 address (54ms)
        ✓ should not allow targetAmount to be 0
        ✓ should return 0 for an incompatible path
        ✓ should return the supply amount
      swapWithExactSupply
        ✓ should not allow path to contain a 0x0 address (77ms)
        ✓ should not allow supplyAmount to be 0
        ✓ should allocate the tokens to the caller (1225ms)
        ✓ should emit a Swaped event (5491ms)
      swapWithExactTarget
        ✓ should not allow a token in a path to be a 0x0 address (140ms)
        ✓ should not allow targetAmount to be 0
        ✓ should allocate tokens to the caller (1284ms)
        ✓ should emit Swaped event (5400ms)
      addLiquidity
        ✓ should not allow tokenA to be 0x0 address
        ✓ should not allow tokenB to be 0x0 address
        ✓ should not allow maxAmountA to be 0
        ✓ should not allow maxAmountB to be 0
        ✓ should increase liquidity (1380ms)
        ✓ should emit AddedLiquidity event (9275ms)
      removeLiquidity
        ✓ should not allow tokenA to be a 0x0 address (63ms)
        ✓ should not allow tokenB to be a 0x0 address
        ✓ should not allow removeShare to be 0
        ✓ should reduce the liquidity (1169ms)
        ✓ should emit RemovedLiquidity event (5348ms)


  34 passing (32s)

✨  Done in 31.99s.
```

## User journey

Since there is no contract to deploy, let's add a simulation of a user interacting with a `DEX` and log all of the changes and information ot the console. The script will be called `userJourney.js` and will reside in the `scripts/` folder:

```shell
touch scripts/userJourney.js
```

The empty user journey script together with the imports of `ACA`, `AUSD`, `DEX` and `DOT` from `@acala-network/contracts`, `Contract`, `formatUnits` and `parseUnits` from `ethers` and precompiled `DEX` and `Token` smart contracts from `@acala-network/contracts` should look like this:

```javascript
const { ACA, AUSD, DEX, DOT } = require("@acala-network/contracts/utils/MandalaAddress");
const { Contract } = require("ethers");

const DEXContract = require("@acala-network/contracts/build/contracts/DEX.json");
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json")
const { formatUnits, parseUnits } = require("ethers/lib/utils");

async function main () {
        
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

We will pad the log to the console with empty strings in order to get a more verbose output. At the beginning of the script, we assign a Signer value to `deployer`, get the Signer's initial native balance and instantiate the predeployed `DEX` and `Token` smart contracts with the help of the `ADDRESS` utility:

```javascript
    console.log("");
    console.log("");

    const [deployer] = await ethers.getSigners();

    console.log("Interacting with DEX using account:", deployer.address);
    
    const initialBalance = await deployer.getBalance();

    console.log("Initial account balance: %s ACA", formatUnits(initialBalance.toString(), 12));

    console.log("");
    console.log("");

    console.log("Instantiating DEX and token smart contracts");

    const instance = new Contract(DEX, DEXContract.abi, deployer);
    const ACAinstance = new Contract(ACA, TokenContract.abi, deployer);
    const AUSDinstance = new Contract(AUSD, TokenContract.abi, deployer);
    const DOTinstance = new Contract(DOT, TokenContract.abi, deployer);

    console.log("DEX instantiated with address", instance.address);
    console.log("ACA token instantiated with address", ACAinstance.address);
    console.log("AUSD token instantiated with address", AUSDinstance.address);
    console.log("DOT token instantiated with address", DOTinstance.address);
```

Now that we have instatiated the token smart contracts, we can get the balance of the `deployer` for each of them and log the initial balances to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Getting inital token balances");

    const initialAcaBalance = await ACAinstance.balanceOf(deployer.address);
    const initialAusdBalance = await AUSDinstance.balanceOf(deployer.address);
    const initialDotBalance = await DOTinstance.balanceOf(deployer.address);

    console.log("Inital %s ACA balance: %s ACA", deployer.address, formatUnits(initialAcaBalance.toString(), 12));
    console.log("Inital %s AUSD balance: %s AUSD", deployer.address, formatUnits(initialAusdBalance.toString(), 12));
    console.log("Inital %s DOT balance: %s DOT", deployer.address, formatUnits(initialDotBalance.toString(), 12));
```

Let's query some information from `DEX`. First thing that we will query on our journey are the liquidity pools for 3 pairs. After we log the results to the console, we can query the liquidity pool tokens for the same three pairs and log those as well:

```javascript
    console.log("");
    console.log("");

    console.log("Getting liquidity pools");

    const initialAcaAusdLP = await instance.getLiquidityPool(ACA, AUSD);
    const initialAcaDotLP = await instance.getLiquidityPool(ACA, DOT);
    const initialDotAusdLP = await instance.getLiquidityPool(DOT, AUSD);

    console.log("Initial ACA - AUSD liquidity pool: %s ACA - %s AUSD", formatUnits(initialAcaAusdLP[0].toString(), 12), formatUnits(initialAcaAusdLP[1].toString(), 12));
    console.log("Initial ACA - DOT liquidity pool: %s ACA - %s DOT", formatUnits(initialAcaDotLP[0].toString(), 12), formatUnits(initialAcaDotLP[1].toString(), 12));
    console.log("Initial DOT - AUSD liquidity pool: %s DOT - %s AUSD", formatUnits(initialDotAusdLP[0].toString(), 12), formatUnits(initialDotAusdLP[1].toString(), 12));

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
    const supply = initialAcaBalance.div(1000);

    const expectedTarget1 = await instance.getSwapTargetAmount(path1, supply);
    const expectedTarget2 = await instance.getSwapTargetAmount(path2, supply);

    console.log("Expected target when using path ACA -> AUSD: %s AUSD", formatUnits(expectedTarget1.toString(), 12));
    console.log("Expected target when using path ACA -> AUSD -> DOT: %s DOT", formatUnits(expectedTarget2.toString(), 12));

    console.log("");
    console.log("");

    console.log("Swapping with exact supply");

    await instance.swapWithExactSupply(path1, supply, 1);
    await instance.swapWithExactSupply(path2, supply, 1);

    const halfwayAcaBalance = await ACAinstance.balanceOf(deployer.address);
    const halfwayAusdBalance = await AUSDinstance.balanceOf(deployer.address);
    const halfwayDotBalance = await DOTinstance.balanceOf(deployer.address);

    console.log("Halfway %s ACA balance: %s ACA", deployer.address, formatUnits(halfwayAcaBalance.toString(), 12));
    console.log("Halfway %s AUSD balance: %s AUSD", deployer.address, formatUnits(halfwayAusdBalance.toString(), 12));
    console.log("Halfway %s DOT balance: %s DOT", deployer.address, formatUnits(halfwayDotBalance.toString(), 12));

    console.log("%s AUSD balance increase was %s AUSD, while the expected increase was %s AUSD.", deployer.address, formatUnits(halfwayAusdBalance.sub(initialAusdBalance).toString(), 12), formatUnits(expectedTarget1.toString(), 12));
    console.log("%s DOT balance increase was %s DOT, while the expected increase was %s DOT.", deployer.address, formatUnits(halfwayDotBalance.sub(initialDotBalance).toString(), 12), formatUnits(expectedTarget2.toString(), 12));
```

Another way of swapping the tokens is with exact target. Here we have to set the maximum amount of supply we are willing to swap and the exact tartget of the egress token we would like to receive. To estimate what our maximum supply should be, we can first estimate the expected supply needed in order to get the exact target out of the swap. In order to get 10 of AUSD and 10 of DOT, we need to use the `parseUnits` utility, to properly format the desired target values, which both have 12 decimal spaces. After we get the estimated required supplies for each swap, we can log those to the console as well as use them as the maximum supply when swapping with exact target. As it is possible, that the required maximum supply increases, we add 1 ACA when swapping using the first path and 10 ACA, when swapping with the second path. When the swap is executed, we can query the token balances once more and log them as well as the comparisons with the expected changes to the console:

```javascript
    console.log("");
    console.log("");

    console.log("Getting expected supply amount");

    const targetAusd = parseUnits("10", 12);
    const targetDot = parseUnits("10", 12);

    const expectedSupply1 = await instance.getSwapSupplyAmount(path1, targetAusd);
    const expectedSupply2 = await instance.getSwapSupplyAmount(path2, targetDot);
    
    console.log("Expected supply for getting %s AUSD in order to reach a total of %s AUSD is %s ACA.", formatUnits(targetAusd.toString(), 12), formatUnits(targetAusd.add(halfwayAusdBalance).toString(), 12), formatUnits(expectedSupply1.toString(), 12));
    console.log("Expected supply for getting %s DOT in order to reach a total of %s DOT is %s ACA.", formatUnits(targetDot.toString(), 12), formatUnits(targetDot.add(halfwayDotBalance).toString(), 12), formatUnits(expectedSupply2.toString(), 12));

    console.log("");
    console.log("");

    console.log("Swapping with exact target");

    await instance.swapWithExactTarget(path1, targetAusd, expectedSupply1.add(parseUnits("1", 12)));
    await instance.swapWithExactTarget(path2, targetAusd, expectedSupply2.add(parseUnits("10", 12)));

    const finalAcaBalance = await ACAinstance.balanceOf(deployer.address);
    const finalAusdBalance = await AUSDinstance.balanceOf(deployer.address);
    const finalDotBalance = await DOTinstance.balanceOf(deployer.address);

    console.log("Final %s ACA balance: %s ACA", deployer.address, formatUnits(finalAcaBalance.toString(), 12));
    console.log("Final %s AUSD balance: %s AUSD", deployer.address, formatUnits(finalAusdBalance.toString(), 12));
    console.log("Final %s DOT balance: %s DOT", deployer.address, formatUnits(finalDotBalance.toString(), 12));

    console.log("AUSD balance has increased by %s AUSD, while the expected increase was %s AUSD.", formatUnits(finalAusdBalance.sub(halfwayAusdBalance), 12), formatUnits(targetAusd.toString(), 12));
    console.log("DOT balance has increased by %s DOT, while the expected increase was %s DOT.", formatUnits(finalDotBalance.sub(halfwayDotBalance), 12), formatUnits(targetDot.toString(), 12));

    console.log("Expected decrease of AUSD balance was %s AUSD, while the actual decrease was %s AUSD.", formatUnits(expectedSupply1.add(expectedSupply2).toString(), 12), formatUnits(halfwayAcaBalance.sub(finalAcaBalance).toString(), 12));
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
    const { ACA, AUSD, DEX, DOT } = require("@acala-network/contracts/utils/MandalaAddress");
    const { Contract } = require("ethers");

    const DEXContract = require("@acala-network/contracts/build/contracts/DEX.json");
    const TokenContract = require("@acala-network/contracts/build/contracts/Token.json")
    const { formatUnits, parseUnits } = require("ethers/lib/utils");

    async function main () {
            console.log("");
            console.log("");

            const [deployer] = await ethers.getSigners();

            console.log("Interacting with DEX using account:", deployer.address);
            
            const initialBalance = await deployer.getBalance();

            console.log("Initial account balance: %s ACA", formatUnits(initialBalance.toString(), 12));

            console.log("");
            console.log("");

            console.log("Instantiating DEX and token smart contracts");

            const instance = new Contract(DEX, DEXContract.abi, deployer);
            const ACAinstance = new Contract(ACA, TokenContract.abi, deployer);
            const AUSDinstance = new Contract(AUSD, TokenContract.abi, deployer);
            const DOTinstance = new Contract(DOT, TokenContract.abi, deployer);

            console.log("DEX instantiated with address", instance.address);
            console.log("ACA token instantiated with address", ACAinstance.address);
            console.log("AUSD token instantiated with address", AUSDinstance.address);
            console.log("DOT token instantiated with address", DOTinstance.address);

            console.log("");
            console.log("");

            console.log("Getting inital token balances");

            const initialAcaBalance = await ACAinstance.balanceOf(deployer.address);
            const initialAusdBalance = await AUSDinstance.balanceOf(deployer.address);
            const initialDotBalance = await DOTinstance.balanceOf(deployer.address);

            console.log("Inital %s ACA balance: %s ACA", deployer.address, formatUnits(initialAcaBalance.toString(), 12));
            console.log("Inital %s AUSD balance: %s AUSD", deployer.address, formatUnits(initialAusdBalance.toString(), 12));
            console.log("Inital %s DOT balance: %s DOT", deployer.address, formatUnits(initialDotBalance.toString(), 12));

            console.log("");
            console.log("");

            console.log("Getting liquidity pools");

            const initialAcaAusdLP = await instance.getLiquidityPool(ACA, AUSD);
            const initialAcaDotLP = await instance.getLiquidityPool(ACA, DOT);
            const initialDotAusdLP = await instance.getLiquidityPool(DOT, AUSD);

            console.log("Initial ACA - AUSD liquidity pool: %s ACA - %s AUSD", formatUnits(initialAcaAusdLP[0].toString(), 12), formatUnits(initialAcaAusdLP[1].toString(), 12));
            console.log("Initial ACA - DOT liquidity pool: %s ACA - %s DOT", formatUnits(initialAcaDotLP[0].toString(), 12), formatUnits(initialAcaDotLP[1].toString(), 12));
            console.log("Initial DOT - AUSD liquidity pool: %s DOT - %s AUSD", formatUnits(initialDotAusdLP[0].toString(), 12), formatUnits(initialDotAusdLP[1].toString(), 12));

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
            const supply = initialAcaBalance.div(1000);

            const expectedTarget1 = await instance.getSwapTargetAmount(path1, supply);
            const expectedTarget2 = await instance.getSwapTargetAmount(path2, supply);

            console.log("Expected target when using path ACA -> AUSD: %s AUSD", formatUnits(expectedTarget1.toString(), 12));
            console.log("Expected target when using path ACA -> AUSD -> DOT: %s DOT", formatUnits(expectedTarget2.toString(), 12));

            console.log("");
            console.log("");

            console.log("Swapping with exact supply");

            await instance.swapWithExactSupply(path1, supply, 1);
            await instance.swapWithExactSupply(path2, supply, 1);

            const halfwayAcaBalance = await ACAinstance.balanceOf(deployer.address);
            const halfwayAusdBalance = await AUSDinstance.balanceOf(deployer.address);
            const halfwayDotBalance = await DOTinstance.balanceOf(deployer.address);

            console.log("Halfway %s ACA balance: %s ACA", deployer.address, formatUnits(halfwayAcaBalance.toString(), 12));
            console.log("Halfway %s AUSD balance: %s AUSD", deployer.address, formatUnits(halfwayAusdBalance.toString(), 12));
            console.log("Halfway %s DOT balance: %s DOT", deployer.address, formatUnits(halfwayDotBalance.toString(), 12));

            console.log("%s AUSD balance increase was %s AUSD, while the expected increase was %s AUSD.", deployer.address, formatUnits(halfwayAusdBalance.sub(initialAusdBalance).toString(), 12), formatUnits(expectedTarget1.toString(), 12));
            console.log("%s DOT balance increase was %s DOT, while the expected increase was %s DOT.", deployer.address, formatUnits(halfwayDotBalance.sub(initialDotBalance).toString(), 12), formatUnits(expectedTarget2.toString(), 12));

            console.log("");
            console.log("");

            console.log("Getting expected supply amount");

            const targetAusd = parseUnits("10", 12);
            const targetDot = parseUnits("10", 12);

            const expectedSupply1 = await instance.getSwapSupplyAmount(path1, targetAusd);
            const expectedSupply2 = await instance.getSwapSupplyAmount(path2, targetDot);
            
            console.log("Expected supply for getting %s AUSD in order to reach a total of %s AUSD is %s ACA.", formatUnits(targetAusd.toString(), 12), formatUnits(targetAusd.add(halfwayAusdBalance).toString(), 12), formatUnits(expectedSupply1.toString(), 12));
            console.log("Expected supply for getting %s DOT in order to reach a total of %s DOT is %s ACA.", formatUnits(targetDot.toString(), 12), formatUnits(targetDot.add(halfwayDotBalance).toString(), 12), formatUnits(expectedSupply2.toString(), 12));

            console.log("");
            console.log("");

            console.log("Swapping with exact target");

            await instance.swapWithExactTarget(path1, targetAusd, expectedSupply1.add(parseUnits("1", 12)));
            await instance.swapWithExactTarget(path2, targetAusd, expectedSupply2.add(parseUnits("10", 12)));

            const finalAcaBalance = await ACAinstance.balanceOf(deployer.address);
            const finalAusdBalance = await AUSDinstance.balanceOf(deployer.address);
            const finalDotBalance = await DOTinstance.balanceOf(deployer.address);

            console.log("Final %s ACA balance: %s ACA", deployer.address, formatUnits(finalAcaBalance.toString(), 12));
            console.log("Final %s AUSD balance: %s AUSD", deployer.address, formatUnits(finalAusdBalance.toString(), 12));
            console.log("Final %s DOT balance: %s DOT", deployer.address, formatUnits(finalDotBalance.toString(), 12));

            console.log("AUSD balance has increased by %s AUSD, while the expected increase was %s AUSD.", formatUnits(finalAusdBalance.sub(halfwayAusdBalance), 12), formatUnits(targetAusd.toString(), 12));
            console.log("DOT balance has increased by %s DOT, while the expected increase was %s DOT.", formatUnits(finalDotBalance.sub(halfwayDotBalance), 12), formatUnits(targetDot.toString(), 12));

            console.log("Expected decrease of AUSD balance was %s AUSD, while the actual decrease was %s AUSD.", formatUnits(expectedSupply1.add(expectedSupply2).toString(), 12), formatUnits(halfwayAcaBalance.sub(finalAcaBalance).toString(), 12));

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


Interacting with DEX using account: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Initial account balance: 10000999000000.0 ACA


Instantiating DEX and token smart contracts
DEX instantiated with address 0x0000000000000000000000000000000000000803
ACA token instantiated with address 0x0000000000000000000100000000000000000000
AUSD token instantiated with address 0x0000000000000000000100000000000000000001
DOT token instantiated with address 0x0000000000000000000100000000000000000002


Getting inital token balances
Inital 0x75E480dB528101a381Ce68544611C169Ad7EB342 ACA balance: 10001000.0 ACA
Inital 0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance: 10000000.0 AUSD
Inital 0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance: 10000000.0 DOT


Getting liquidity pools
Initial ACA - AUSD liquidity pool: 1000000.0 ACA - 2000000.0 AUSD
Initial ACA - DOT liquidity pool: 0.0 ACA - 0.0 DOT
Initial DOT - AUSD liquidity pool: 20000.0 DOT - 1000000.0 AUSD


Getting liquidity pool token addresses
Liquidity pool token address for ACA - AUSD: 0x0000000000000000000200000000000000000001
Liquidity pool token address for ACA - DOT: 0x0000000000000000000200000000000000000002
Liquidity pool token address for DOT - AUSD: 0x0000000000000000000200000000010000000002


Getting expected swap target amounts
Expected target when using path ACA -> AUSD: 19784.332751266429 AUSD
Expected target when using path ACA -> AUSD -> DOT: 387.629643512645 DOT


Swapping with exact supply
Halfway 0x75E480dB528101a381Ce68544611C169Ad7EB342 ACA balance: 9980997.910299541571 ACA
Halfway 0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance: 10019784.332751266429 AUSD
Halfway 0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance: 10000380.176464723134 DOT
0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance increase was 19784.332751266429 AUSD, while the expected increase was 19784.332751266429 AUSD.
0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance increase was 380.176464723134 DOT, while the expected increase was 387.629643512645 DOT.


Getting expected supply amount
Expected supply for getting 10.0 AUSD in order to reach a total of 10019794.332751266429 AUSD is 5.207151566085 ACA.
Expected supply for getting 10.0 DOT in order to reach a total of 10000390.176464723134 DOT is 271.029934786494 ACA.


Swapping with exact target
Final 0x75E480dB528101a381Ce68544611C169Ad7EB342 ACA balance: 9980721.58074658888 ACA
Final 0x75E480dB528101a381Ce68544611C169Ad7EB342 AUSD balance: 10019794.332751266429 AUSD
Final 0x75E480dB528101a381Ce68544611C169Ad7EB342 DOT balance: 10000390.176464723134 DOT
AUSD balance has increased by 10.0 AUSD, while the expected increase was 10.0 AUSD.
DOT balance has increased by 10.0 DOT, while the expected increase was 10.0 DOT.
Expected decrease of AUSD balance was 276.237086352579 AUSD, while the actual decrease was 276.329552952691 AUSD.


User journey completed!
✨  Done in 8.72s.
```

## Summary

We have built upon the knowledge on how to interact with the Acala EVM+ and gotten familiar with Acala EVM+ precompiles and predeploys. To run the test we can use the `yarn test-mandala` and `yarn test-madala:pubDev` and to run the user journey script we can use the `yarn user-journey-mandala` and `yarn user-journey-mandala:pubDev`. As we are using utilities only available in the Acala EVM+, we can no longer use a conventional development network like Ganache or Hardhat's emulated network.
