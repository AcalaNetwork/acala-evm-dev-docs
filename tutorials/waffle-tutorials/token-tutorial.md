---
description: A tutorial on how to build an ERC-20 compatible smart contract in Acala EVM+.
---

# Token tutorial

## Table of contents

* [About](broken-reference)
* [Smart contract](token-tutorial.md#smart-contract)
* [Test](token-tutorial.md#test)
* [Deploy script](token-tutorial.md#deploy-script)
* [Summary](token-tutorial.md#summary)

## About

This is an example that builds upon the [echo example](../truffle-tutorials/echo-tutorial.md). `Echo` was a simple example on building an intreactable state changing smart contract. `Token` is an example of [ERC20](https://eips.ethereum.org/EIPS/eip-20) token implementation in Acala EVM+. We won't be building an administrated or upgradeable token and we will use OpenZeppelin [ERC20 implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol). For the setup and naming, replace the `echo` with `token`. Let's jump into it!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/waffle-tutorials/tree/master/token](https://github.com/AcalaNetwork/waffle-tutorials/tree/master/token)
{% endhint %}

## Smart contract

In this tutorial we will be adding a simple smart contract that imports the ERC20 smart contract from `openzeppelin/contracts` and has a constructor that sets the initial balance of the sender (which also represents the total supply of the token) as well as the name of the token and its abbreviation.

To be able to import the `ERC20` from OpenZeppelin, we have to add `@openzeppelin/contracts` as a development dependency:

```shell
yarn add --dev @openzeppelin/contracts
```

Now that we have embedded the `@openzeppelin/contracts` dependency, we can focus on building our smart contract. Your empty smart contract should look like this:

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

<summary>Your contracts/Token.sol should look like this:</summary>

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

As the Token smart contract is ready to be compiled, we can use the `yarn build` command (like we did in the hello-world) to compile the smart contract, which will create the `build` directory and contain the compiled smart contract.

## Test

Your test file should be named `Token.test.ts` and the empty test along with the import statement should look like this:

```typescript
import { expect, use } from 'chai';
import { deployContract, solidity } from 'ethereum-waffle';
import { Contract, ethers } from 'ethers';

import { evmChai, Signer, TestProvider } from '@acala-network/bodhi';

import Token from '../build/Token.json';
import { getTestProvider } from '../utils/setup';

use(solidity);
use(evmChai);

const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

describe("Token", () => {

});
```

In addition to the similar import statements to the ones in the echo the `NULL_ADDRESS` is also defined. It is used for validating the incorrect calls.

First thing to add to the `Echo` describe block are the `provider`, `deployer`, `user`, `instance`, `deployerAddress` and `userAddress` variables. Within the `before` action we assign the `TestProvider` to `provider`, `Signer` to the `wallet` and `user` variables, `deployerAddress` and `userAddress` are used to store the addresses of `deployer` and `user` and deployed contract to `instance`. The `after` action will disconnect from the `provider`:

```typescript
    let provider: TestProvider;
    let deployer: Signer;
    let user: Signer;
    let instance: Contract;
    let deployerAddress: String;
    let userAddress: String;

    before(async () => {
      provider = await getTestProvider();
      [deployer, user] = await provider.getWallets();
      instance = await deployContract(deployer, Token, [1234567890]);
      deployerAddress = await deployer.getAddress();
      userAddress = await user.getAddress();
    });

    after(async () => {
      provider.api.disconnect();
    });
```

There are two describe blocks within `Token` block. The `Deployment` block validates that the smart contract was deployed as expected and the `Operation` block validates the operation of the smart contract:

```typescript
    describe("Deployment", () => {
        
    });

    describe("Operation", () => {
        
    });
```

`Deployment` block contains the examples that validate the expected initial state of the smart contract:

1. The `name` should equal `Token`.
2. The `symbol` should equal `TKN`.
3. The total supply of the smart contract should equal `1234567890`.
4. The initial balance of the `deployer` account should equal `1234567890`.
5. The `user` account should have `0` balance.
6. The allowances should be set to `0` when the smart contract is deployed.

```typescript
      it("should assing correct name to Token", async () => {
        expect(await instance.name()).to.equal("Token");
      });

      it("should assign correct symbol", async () => {
        expect(await instance.symbol()).to.equal("TKN");
      });

      it("should assign correct total supply", async () => {
        expect((await instance.totalSupply()).toNumber()).to.equal(1234567890);
      });

      it("should assign appropriate balance to the deployer", async () => {
        expect((await instance.balanceOf(deployerAddress)).toNumber()).to.equal(1234567890);
      });

      it("should not assign balance to a random address", async () => {
        expect((await instance.balanceOf(userAddress)).toNumber()).to.equal(0);
      });

      it("should set the allowances to 0", async () => {
        expect((await instance.allowance(deployerAddress, userAddress)).toNumber()).to.equal(0);
      });
```

The `Operation` block in itself is separated into two `describe` blocks, which are separated in itself:

1. `Transfer`: Contains the test cases to validate the transaction functionality of ERC20 tokens:

* `transfer()`: Validates the correct operation of the `transfer()` function.

1. `Allowances`: Contains the test cases to validate the allowances and connected functionality of ERC20 tokens:

* `approve()`: Validates the correct operation of the `approve()` function.
* `increaseAllowance()`: Validates the correct operation of the `increaseAllowance()` function.
* `decreaseAllowance()`: Validates the correct operation of the `decreaseAllowance()` function.
* `transferFrom()`: Validates the correct operation of the `transferFrom()` function.

The contents of the `Operation` block should look like this:

```typescript
      describe("Transfer", () => {
        describe("transfer()", () => {

        });
      });

      describe("Allowances", () => {
        describe("approve", () => {

        });

        describe("increaseAllowance", () => {

        });

        describe("decreaseAllowance()", () => {

        });

        describe("transferFrom()", () => {

        });
      });
```

The `transfer()` example validates the following:

1. When transferring tokens, the balances should be updated.
2. `Transfer` event should be emitted.
3. Transfers to `0x0` address should be reverted.
4. Trying to transfer more than own balance should be reverted.

These examples should look like this:

```typescript
          it("should update balances", async () => {
            const initialUserBalance = await instance.balanceOf(userAddress);
            const initialDeployerBalance = await instance.balanceOf(deployerAddress);

            await instance.connect(deployer).transfer(userAddress, 500);

            const finalUserBalance = await instance.balanceOf(userAddress);
            const finalDeployerBalance = await instance.balanceOf(deployerAddress);

            expect(initialDeployerBalance.toNumber() - finalDeployerBalance.toNumber()).to.equal(500);
            expect(finalUserBalance.toNumber() - initialUserBalance.toNumber()).to.equal(500);
          });

          it("should emit Transfer event", async () => {
            await expect(instance.connect(deployer).transfer(userAddress, 500)).to
              .emit(instance, "Transfer")
              .withArgs(deployerAddress, userAddress, 500);
          });

          it("should revert when trying to transfer to 0x0 address", async () => {
            await expect(instance.connect(deployer).transfer(NULL_ADDRESS, 500)).to
              .be.revertedWith("ERC20: transfer to the zero address");
          });

          it("should revert when trying to transfer more than own balance", async () => {
            await expect(instance.connect(user).transfer(deployerAddress, 1234567890)).to
              .be.revertedWith("ERC20: transfer amount exceeds balance");
          });
```

The `approve()` example validates the following:

1. Allowance should be granted for less funds than the owner has.
2. Allowance should be granted for more funds than the owner has.
3. `Approval` event should be emitted when giving allowance.
4. Call should be reverted if the address receiving the allowance is `0x0`.

These examples should look like this:

```typescript
          it("should grant allowance for an amount smaller than own balance", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            const allowance = await instance.allowance(deployerAddress, userAddress);

            expect(allowance.toNumber()).to.equal(1500);
          });

          it("should grant allowance for an amount greater than own balance", async () => {
            await instance.connect(deployer).approve(userAddress, 12345678900);

            const allowance = await instance.allowance(deployerAddress, userAddress);

            expect(allowance.toNumber()).to.equal(12345678900);
          });

          it("should emit Approval", async () => {
            await expect(instance.connect(deployer).approve(userAddress, 1500)).to
              .emit(instance, "Approval")
              .withArgs(deployerAddress, userAddress, 1500);
          });

          it("should revert when trying to grant allowance to 0x0 address", async () => {
            await expect(instance.connect(deployer).approve(NULL_ADDRESS, 1500)).to
              .be.revertedWith("ERC20: approve to the zero address");
          });
```

The `increaseAllowance()` example validates the following:

1. Owner should be able to increase allowance to an amount smaller that the total funds that they have.
2. Owner should be able to increase the balance to more than the amount of total funds that they have.
3. `Approval` event should be emitted.
4. The function can be called even if there was no preexisting allowance.

These examples should look like this:

```typescript
          it("should increase allowance for a total amount lower than own balance", async () => {
            await instance.connect(deployer).approve(userAddress, 1000);

            const initialAllowance = await instance.allowance(deployerAddress, userAddress);

            await instance.connect(deployer).increaseAllowance(userAddress, 500);

            const finalAllowance = await instance.allowance(deployerAddress, userAddress);

            expect(finalAllowance.toNumber() - initialAllowance.toNumber()).to.equal(500);
          });

          it("should increase allowance for a total amount higher than own balance", async () => {
            await instance.connect(deployer).approve(userAddress, 1000);

            const initialAllowance = await instance.allowance(deployerAddress, userAddress);

            await instance.connect(deployer).increaseAllowance(userAddress, 1234567890);

            const finalAllowance = await instance.allowance(deployerAddress, userAddress);

            expect(finalAllowance.toNumber() - initialAllowance.toNumber()).to.equal(1234567890);
          });

          it("should emit Approval event", async () => {
            await instance.connect(deployer).approve(userAddress, 1000);

            await expect(instance.connect(deployer).increaseAllowance(userAddress, 500)).to
              .emit(instance, "Approval")
              .withArgs(deployerAddress, userAddress, 1500);
          });

          it("should increase the allowance even if there is no preeexisting allowance", async () => {
            const initialAllowance = await instance.allowance(deployerAddress, userAddress);

            await instance.connect(deployer).increaseAllowance(userAddress, 500);

            const finalAllowance = await instance.allowance(deployerAddress, userAddress);

            expect(finalAllowance.toNumber() - initialAllowance.toNumber()).to.equal(500);
          });
```

The `decreaseAllowance()` example validates the following:

1. Owner should be able to decrease allowance.
2. `Approval` event should be emitted.
3. Call should be reverted when trying to decrease the allowance below 0.

These examples should look like this:

We can now add the following test cases to our describe block:

```typescript
          it("should allow owner to decrease alowance", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            const initialAllowance = await instance.allowance(deployerAddress, userAddress);

            await instance.connect(deployer).decreaseAllowance(userAddress, 500);

            const finalAllowance = await instance.allowance(deployerAddress, userAddress);

            expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(500);
          });

          it("should emit Approval event", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            await expect(instance.connect(deployer).decreaseAllowance(userAddress, 500)).to
              .emit(instance, "Approval")
              .withArgs(deployerAddress, userAddress, 1000);
          });

          it("should revert when trying to reduce the allowance below 0", async () => {
            await expect(instance.connect(deployer).decreaseAllowance(userAddress, 10000)).to
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

```typescript
          it("should allow transfer if the allowance is given", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            const initialBalance = await instance.balanceOf(userAddress);

            await instance.connect(user).transferFrom(deployerAddress, userAddress, 500);

            const finalBalance = await instance.balanceOf(userAddress);

            expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(500);
          });

          it("should emit Transfer", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 500)).to
              .emit(instance, "Transfer")
              .withArgs(deployerAddress, userAddress, 500);
          });

          it("should emit Approval", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 500)).to
              .emit(instance, "Approval")
              .withArgs(deployerAddress, userAddress, 1000);
          });

          it("should update allowance", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            const initialAllowance = await instance.allowance(deployerAddress, userAddress);

            await instance.connect(user).transferFrom(deployerAddress, userAddress, 500);

            const finalAllowance = await instance.allowance(deployerAddress, userAddress);

            expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(500);
          });

          it("should revert when trying to transfer amount higher than allowance", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 100000)).to
              .be.revertedWith("ERC20: transfer amount exceeds allowance");
          });

          it("should rever when trying to transfer to 0x0 address", async () => {
            await instance.connect(deployer).approve(userAddress, 1500);

            await expect(instance.connect(user).transferFrom(deployerAddress, NULL_ADDRESS, 500)).to
              .be.revertedWith("ERC20: transfer to the zero address");
          });

          it("should revert when owner doesn't have enough funds", async () => {
            await instance.connect(deployer).approve(userAddress, 12345678900);

            await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 12345678900)).to
              .be.revertedWith("ERC20: transfer amount exceeds balance");
          });

          it("should revert when allowance was not given", async () => {
            await expect(instance.connect(deployer).transferFrom(userAddress, deployerAddress, 1500)).to
              .be.revertedWith("ERC20: transfer amount exceeds allowance");
          });
```

With that, our test is ready to be run.

<details>

<summary>Your test/Token.test.ts should look like this:</summary>

```typescript
import { expect, use } from 'chai';
import { deployContract, solidity } from 'ethereum-waffle';
import { Contract } from 'ethers';

import { evmChai, Signer, TestProvider } from '@acala-network/bodhi';

import Token from '../build/Token.json';
import { getTestProvider } from '../utils/setup';

use(solidity);
use(evmChai);

const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

describe("Token", () => {
    let provider: TestProvider;
    let deployer: Signer;
    let user: Signer;
    let instance: Contract;
    let deployerAddress: String;
    let userAddress: String;

    before(async () => {
        provider = await getTestProvider();
        [deployer, user] = await provider.getWallets();
        instance = await deployContract(deployer, Token, [1234567890]);
        deployerAddress = await deployer.getAddress();
        userAddress = await user.getAddress();
    });

    after(async () => {
      provider.api.disconnect();
    });

    describe("Deployment", () => {
        it("should assing correct name to Token", async () => {
            expect(await instance.name()).to.equal("Token");
        });

        it("should assign correct symbol", async () => {
            expect(await instance.symbol()).to.equal("TKN");
        });

        it("should assign correct total supply", async () => {
            expect((await instance.totalSupply()).toNumber()).to.equal(1234567890);
        });

        it("should assign appropriate balance to the deployer", async () => {
            expect((await instance.balanceOf(deployerAddress)).toNumber()).to.equal(1234567890);
        });

        it("should not assign balance to a random address", async () => {
            expect((await instance.balanceOf(userAddress)).toNumber()).to.equal(0);
        });

        it("should set the allowances to 0", async () => {
            expect((await instance.allowance(deployerAddress, userAddress)).toNumber()).to.equal(0);
        });
    });

    describe("Operation", () => {
        describe("Transfer", () => {
            describe("transfer()", () => {
                it("should update balances", async () => {
                    const initialUserBalance = await instance.balanceOf(userAddress);
                    const initialDeployerBalance = await instance.balanceOf(deployerAddress);

                    await instance.connect(deployer).transfer(userAddress, 500);

                    const finalUserBalance = await instance.balanceOf(userAddress);
                    const finalDeployerBalance = await instance.balanceOf(deployerAddress);

                    expect(initialDeployerBalance.toNumber() - finalDeployerBalance.toNumber()).to.equal(500);
                    expect(finalUserBalance.toNumber() - initialUserBalance.toNumber()).to.equal(500);
                });

                it("should emit Transfer event", async () => {
                    await expect(instance.connect(deployer).transfer(userAddress, 500)).to
                    .emit(instance, "Transfer")
                    .withArgs(deployerAddress, userAddress, 500);
                });

                it("should revert when trying to transfer to 0x0 address", async () => {
                    await expect(instance.connect(deployer).transfer(NULL_ADDRESS, 500)).to
                    .be.revertedWith("ERC20: transfer to the zero address");
                });

                it("should revert when trying to transfer more than own balance", async () => {
                    await expect(instance.connect(user).transfer(deployerAddress, 1234567890)).to
                    .be.revertedWith("ERC20: transfer amount exceeds balance");
                });
            });
        });

        describe("Allowances", () => {
            describe("approve()", () => {
                it("should grant allowance for an amount smaller than own balance", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    const allowance = await instance.allowance(deployerAddress, userAddress);

                    expect(allowance.toNumber()).to.equal(1500);
                });

                it("should grant allowance for an amount greater than own balance", async () => {
                    await instance.connect(deployer).approve(userAddress, 12345678900);

                    const allowance = await instance.allowance(deployerAddress, userAddress);

                    expect(allowance.toNumber()).to.equal(12345678900);
                });

                it("should emit Approval", async () => {
                    await expect(instance.connect(deployer).approve(userAddress, 1500)).to
                    .emit(instance, "Approval")
                    .withArgs(deployerAddress, userAddress, 1500);
                });

                it("should revert when trying to grant allowance to 0x0 address", async () => {
                    await expect(instance.connect(deployer).approve(NULL_ADDRESS, 1500)).to
                    .be.revertedWith("ERC20: approve to the zero address");
                });
            });

            describe("increaseAllowance()", () => {
                it("should increase allowance for a total amount lower than own balance", async () => {
                    await instance.connect(deployer).approve(userAddress, 1000);

                    const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                    await instance.connect(deployer).increaseAllowance(userAddress, 500);

                    const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                    expect(finalAllowance.toNumber() - initialAllowance.toNumber()).to.equal(500);
                });

                it("should increase allowance for a total amount higher than own balance", async () => {
                    await instance.connect(deployer).approve(userAddress, 1000);

                    const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                    await instance.connect(deployer).increaseAllowance(userAddress, 1234567890);

                    const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                    expect(finalAllowance.toNumber() - initialAllowance.toNumber()).to.equal(1234567890);
                });

                it("should emit Approval event", async () => {
                    await instance.connect(deployer).approve(userAddress, 1000);

                    await expect(instance.connect(deployer).increaseAllowance(userAddress, 500)).to
                    .emit(instance, "Approval")
                    .withArgs(deployerAddress, userAddress, 1500);
                });

                it("should increase the allowance even if there is no preeexisting allowance", async () => {
                    const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                    await instance.connect(deployer).increaseAllowance(userAddress, 500);

                    const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                    expect(finalAllowance.toNumber() - initialAllowance.toNumber()).to.equal(500);
                });
            });

            describe("decreaseAllowance()", () => {
                it("should allow owner to decrease alowance", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                    await instance.connect(deployer).decreaseAllowance(userAddress, 500);

                    const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                    expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(500);
                });

                it("should emit Approval event", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    await expect(instance.connect(deployer).decreaseAllowance(userAddress, 500)).to
                    .emit(instance, "Approval")
                    .withArgs(deployerAddress, userAddress, 1000);
                });

                it("should revert when trying to reduce the allowance below 0", async () => {
                    await expect(instance.connect(deployer).decreaseAllowance(userAddress, 10000)).to
                    .be.revertedWith("ERC20: decreased allowance below zero");
                });
            });

            describe("transferFrom()", () => {
                it("should allow transfer if the allowance is given", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    const initialBalance = await instance.balanceOf(userAddress);

                    await instance.connect(user).transferFrom(deployerAddress, userAddress, 500);

                    const finalBalance = await instance.balanceOf(userAddress);

                    expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(500);
                });

                it("should emit Transfer", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 500)).to
                    .emit(instance, "Transfer")
                    .withArgs(deployerAddress, userAddress, 500);
                });

                it("should emit Approval", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 500)).to
                    .emit(instance, "Approval")
                    .withArgs(deployerAddress, userAddress, 1000);
                });

                it("should update allowance", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    const initialAllowance = await instance.allowance(deployerAddress, userAddress);

                    await instance.connect(user).transferFrom(deployerAddress, userAddress, 500);

                    const finalAllowance = await instance.allowance(deployerAddress, userAddress);

                    expect(initialAllowance.toNumber() - finalAllowance.toNumber()).to.equal(500);
                });

                it("should revert when trying to transfer amount higher than allowance", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 100000)).to
                    .be.revertedWith("ERC20: transfer amount exceeds allowance");
                });

                it("should rever when trying to transfer to 0x0 address", async () => {
                    await instance.connect(deployer).approve(userAddress, 1500);

                    await expect(instance.connect(user).transferFrom(deployerAddress, NULL_ADDRESS, 500)).to
                    .be.revertedWith("ERC20: transfer to the zero address");
                });

                it("should revert when owner doesn't have enough funds", async () => {
                    await instance.connect(deployer).approve(userAddress, 12345678900);

                    await expect(instance.connect(user).transferFrom(deployerAddress, userAddress, 12345678900)).to
                    .be.revertedWith("ERC20: transfer amount exceeds balance");
                });

                it("should revert when allowance was not given", async () => {
                    await expect(instance.connect(deployer).transferFrom(userAddress, deployerAddress, 1500)).to
                    .be.revertedWith("ERC20: transfer amount exceeds allowance");
                });
            });
        });
    });
});
```

</details>

When you run the test with `yarn test`, your tests should pass with the following output:

```shell
yarn test


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ export NODE_ENV=test && mocha -r ts-node/register/transpile-only --timeout 100000 --no-warnings test/**/*.test.ts


  Token
    Deployment
      ✔ should assing correct name to Token (78ms)
      ✔ should assign correct symbol (74ms)
      ✔ should assign correct total supply (42ms)
      ✔ should assign appropriate balance to the deployer (40ms)
      ✔ should not assign balance to a random address (50ms)
      ✔ should set the allowances to 0 (41ms)
    Operation
      Transfer
        transfer()
          ✔ should update balances (365ms)
          ✔ should emit Transfer event (232ms)
          ✔ should revert when trying to transfer to 0x0 address (72ms)
          ✔ should revert when trying to transfer more than own balance (69ms)
      Allowances
        approve()
          ✔ should grant allowance for an amount smaller than own balance (197ms)
          ✔ should grant allowance for an amount greater than own balance (187ms)
          ✔ should emit Approval (172ms)
          ✔ should revert when trying to grant allowance to 0x0 address (64ms)
        increaseAllowance()
          ✔ should increase allowance for a total amount lower than own balance (370ms)
          ✔ should increase allowance for a total amount higher than own balance (376ms)
          ✔ should emit Approval event (323ms)
          ✔ should increase the allowance even if there is no preeexisting allowance (217ms)
        decreaseAllowance()
          ✔ should allow owner to decrease alowance (332ms)
          ✔ should emit Approval event (287ms)
          ✔ should revert when trying to reduce the allowance below 0 (64ms)
        transferFrom()
          ✔ should allow transfer if the allowance is given (364ms)
          ✔ should emit Transfer (285ms)
          ✔ should emit Approval (273ms)
          ✔ should update allowance (332ms)
          ✔ should revert when trying to transfer amount higher than allowance (191ms)
          ✔ should rever when trying to transfer to 0x0 address (162ms)
          ✔ should revert when owner doesn't have enough funds (180ms)
          ✔ should revert when allowance was not given (66ms)


  29 passing (7s)

✨  Done in 25.72s.
```

## Deploy script

The `setup.ts` should remain the same as in the hello-world. The `deploy.ts` needs to have the same imports like the hello-world example, except for the smart contract we are importing:

```typescript
import { use } from 'chai';
import { ContractFactory } from 'ethers';

import { evmChai } from '@acala-network/bodhi';

import Token from '../build/Token.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {

}

main()
```

Within the definition of the `main` function, we first retrieve the `wallet` and `provider` from the `setup()`. Then we output `Deploy Token` to the console and deploy the `Token` smart contract and save it to `instance`. The address of the deployed smart contract is logged into the terminal and we log the information about the token as well. Finally we disconnect from the provider:

```typescript
    const { wallet, provider } = await setup();

    console.log('Deploy Token');

    const instance = await ContractFactory.fromSolidity(Token).connect(wallet).deploy(1234567890);

    console.log("Token address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();
    const totalSupply = await instance.totalSupply();

    console.log("Token name:", name);
    console.log("Token symbol:", symbol);
    console.log("Token total supply:", totalSupply.toNumber());

    provider.api.disconnect();
```

<details>

<summary>Your src/deploy.ts should look like this:</summary>

```typescript
import { use } from 'chai';
import { ContractFactory } from 'ethers';

import { evmChai } from '@acala-network/bodhi';

import Token from '../build/Token.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {
    const { wallet, provider } = await setup();

    console.log('Deploy Token');

    const instance = await ContractFactory.fromSolidity(Token).connect(wallet).deploy(1234567890);

    console.log("Token address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();
    const totalSupply = await instance.totalSupply();

    console.log("Token name:", name);
    console.log("Token symbol:", symbol);
    console.log("Token total supply:", totalSupply.toNumber());

    provider.api.disconnect();
}

main()
```

</details>

Running the `yarn deploy` script should return the following output:

```shell
yarn deploy


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ ts-node --transpile-only src/deploy.ts
Deploy Token
Token address: 0x546411ddd9722De71dA1B836327b37D840F16059
Token name: Token
Token symbol: TKN
Token total supply: 1234567890
```

## Summary

We have built upon the previous examples, added an ERC20 smart contract and tested all of its functionalities. The tests were more detailed and covered more examples. We also ensured that we can interact with the smart contract and that its storage is modified as expected. We can compile smart contract with `yarn build`, test it with `yarn test` and deploy it with `yarn deploy`.
