---
description: >-
  A tutorial showcasing interaction with Acala EVM+'s precompiled and
  predeployed smart contracts.
---

# PrecompiledToken tutorial

## Table of contents

* [About](broken-reference)
* [Smart contract](broken-reference)
* [Test](broken-reference)
* [Script](broken-reference)
* [Summary](broken-reference)

## About

This example introduces the use of Acala EVM+ precompiles and predeploys that are present on every network at a fixed address (the address of a predeployed contract is the same on a local development network, public test network as well as the production network). As this example focuses on showcasing the precompiles and predeploys, it doesn't have a full smart contract. We will however interact with an `ERC20` smart contract that is already deployed to the network and we will get all of the required imports from the [`@acala-network/contracts`](https://github.com/AcalaNetwork/predeploy-contracts) dependency. The precompiles and predeploys are a specific feature of the Acala EVM+, so this and the following tutorial is no longer compatible with traditional EVM development networks (like Ganache).

Let's take a look!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial [https://github.com/AcalaNetwork/truffle-tutorials/tree/master/precompiled-token](https://github.com/AcalaNetwork/truffle-tutorials/tree/master/precompiled-token)
{% endhint %}

## Smart contract

The smart contract in this tutorial is only used to satisfy the Truffle's requirement to have a smart contract to compile. For this, we will create an empty smart contract that will inherit the `Token` from `@acala-network/contracts` dependency. In order to do so, we need to add the dependency to the project:

```shell
yarn add --dev @acala-network/contracts
```

Now that we have added the `@acala-network/contracts` dependency, we can create our empty smart contract. Your skeleton smart contract should look like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

contract PrecompiledToken {

}
```

Import of the `Token` from `@acala-network/contracts` is done between the `pragma` definition and the start od the `contract` block:

```solidity
import "@acala-network/contracts/token/Token.sol";
```

As we now have access to `Token.sol` from `@acala-network/contracts`, we can set the inheritance of our `PrecompiledToken` contract:

```solidity
contract PrecompiledToken is Token {
```

This concludes our `PrecompiledToken` smart contract.

<details>

<summary>Your contracts/PrecompiledToken.sol should look like this:</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

import "@acala-network/contracts/token/Token.sol";

contract PrecompiledToken is Token {
    
}
```

</details>

Tests for this tutorial will validate the expected values returned by the ACA token predeployed smart contract. The test file in our case is called `precompiled_token.js`. Within it we import the `Token` from `@acala-netowkr/contracts` dependency and assign it to `PrecompiledToken` variable. The `ACA`, which is an export from the `ADDRESS` utility of `@acala-network/contracts` dependency, is imported and it holds the value of the address of the `ACA` token. The `MandalaAddress` utility holds the values of the predeployed smart contracts in the local development and Mandala network and we will be using this one. There are also `AcalaNetwork` and `KaruraNetwork`, that hold the addresses of their respective networks.

{% hint style="info" %}
**NOTE: The ACA ERC20 token mirrors the balance of the native ACA currency, so you are able to transfer ACA within your smart contract the same way you would transfer a non-native ERC20 token.**
{% endhint %}

The test file with import statements and an empty test should look like this:

```javascript
const PrecompiledToken = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract("PrecompiledToken", function (accounts) {
  
});
```

To prepare for the testing, we have to define four global variables, `instance` and `deployer`. The `instance` will store the predeployed Token smart contract instance. The `deployer` will store the account that we will be using in our tests. Let's assign them values in the `beforeEach` action:

```javascript
  let instance;
  let deployer;

  beforeEach("setup development environment", async function () {
    deployer = accounts[0];
    instance = await PrecompiledToken.at(ACA);
  });
```

{% hint style="info" %}
**NOTE: You can see how we used the `ACA` from the `ADDRESS` utility in order to set the address of our predeployed smart contract.**
{% endhint %}

{% hint style="info" %}
**NOTE: We are using the `at` utility of the Truffle artifact in order to connect out instance to an already deployed smart contract.**
{% endhint %}

Our test will only contain one section called `Precompiled token` in which we will be checking the following examples:

1. The token should return `Acala` when `name()` is called.
2. The token should return `ACA` when `symbol()` is called.
3. The total supply of the token should be greater than 0 (or not equal to 0 in our case).
4. The balance of our development address should be greater than 0 (or not equal to 0 in our case).

```javascript
  describe("Precompiled token", function () {
    it("should have the correct token name", async function() {
      const name = await instance.name();

      expect(name).to.equal("Acala");
    });

    it("should have the correct token symbol", async function() {
      const symbol = await instance.symbol();

      expect(symbol).to.equal("ACA");
    });

    it("should have the total supply greater than 0", async function() {
      const totalSupply = await instance.totalSupply();

      expect(totalSupply).to.not.equal(0);
    });

    it("should show balance of the deployer address higher than 0", async function() {
      const balance = await instance.balanceOf(deployer);

      expect(balance).to.not.equal(0);
    });
  });
```

With that, our test is ready to be run.

<details>

<summary>Your test/precompiled_token.js should look like this:</summary>

```javascript
const PrecompiledToken = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");

/*
* uncomment accounts to access the test accounts made available by the
* Ethereum client
* See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
*/
contract("PrecompiledToken", function (accounts) {
  let instance;
  let deployer;

  beforeEach("setup development environment", async function () {
    deployer = accounts[0];
    instance = await PrecompiledToken.at(ACA);
  });

  describe("Precompiled token", function () {
    it("should have the correct token name", async function() {
      const name = await instance.name();

      expect(name).to.equal("Acala");
    });

    it("should have the correct token symbol", async function() {
      const symbol = await instance.symbol();

      expect(symbol).to.equal("ACA");
    });

    it("should have the total supply greater than 0", async function() {
      const totalSupply = await instance.totalSupply();

      expect(totalSupply).to.not.equal(0);
    });

    it("should show balance of the deployer address higher than 0", async function() {
      const balance = await instance.balanceOf(deployer);

      expect(balance).to.not.equal(0);
    });
  });
});
```

</details>

{% hint style="info" %}
**NOTE: If you want to interact with other precompiled and predeployed smart contracts, you can take a look at the list of all of the smart contracts supported in the** [**`ADDRESS` utility**](https://github.com/AcalaNetwork/predeploy-contracts/blob/master/contracts/utils/Address.js) **and tweak this example test.**
{% endhint %}

When you run the test with (for example) `yarn test-mandala`, your tests should pass with the following output:

```shell
yarn test-mandala


yarn run v1.22.17
$ truffle test --network mandala
Using network 'mandala'.


Compiling your contracts...
===========================
> Compiling ./contracts/PrecompiledToken.sol
> Artifacts written to /var/folders/_c/x274s0_x6qj1xtkv60pllc800000gp/T/test--68754-Yj7ZAe5DeXnT
> Compiled successfully using:
   - solc: 0.8.9+commit.e5eed63a.Emscripten.clang


  Contract: PrecompiledToken
    Precompiled token
      ✓ should have the correct token name (74ms)
      ✓ should have the correct token symbol
      ✓ should have the total supply greater than 0
      ✓ should show balance of the deployer address higher than 0


  4 passing (538ms)

✨  Done in 5.80s.
```

## Script

As the smart contract is already deployed to the network, we don't need a deploy script, but we can add a script that interacts with the predeployed smart contract and log some values from it to the console.

The Truffle scripts reside in the `scripts` directory. To create it along with the script called `getACAinfo.js` within it, we can use:

```shell
mkdir scripts && touch scripts/getACAinfo.js
```

Within the `getACAinfo.js` we will import the `Token` precompiled smart contract from the `@acala-network/contracts`, `ACA` address from the `ADDRESS` utility and `formatUnits` utility from `ethers`. The empty script will also consist of the callback definition and its execution:

```javascript
const PrecompiledToken = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");
const { formatUnits } = require("ethers/lib/utils")

module.exports = async function(callback) {
  try {
    
  }
  catch(error) {
    console.log(error)
  }

  callback()
}
```

First, we will get the address of the account which will be used to interact with the smart contract and log it to the console along with its balance. Then we assign the predeployed smart contract to the `instance` variable using the Truffle artifact imported from the `@acala-network/contracts` and the `ADDRESS` utility from its utilities. Finally we get and output the `name`, `symbol`, `decimals`, `totalSupply` and own balance to the console. In addition to outputting the raw total supply and balance of our address, we also use the `formatUtils` from `ethers` to format them to be presented in a user friendly way:

```javascript
    const accounts = await web3.eth.getAccounts();
    const deployer = accounts[0];

    console.log("Getting contract info with the account:", deployer);

    console.log("Account balance:", formatUnits((await web3.eth.getBalance(deployer)).toString(), 12));

    const instance = await PrecompiledToken.at(ACA);

    console.log("PrecompiledToken address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();
    const decimals = await instance.decimals();
    const value = await instance.totalSupply();
    const balance = await instance.balanceOf(deployer);
    
    console.log("Token name:", name);
    console.log("Token symbol:", symbol);
    console.log("Token decimal spaces:", decimals.toString());
    console.log("Total supply:", value.toString());
    console.log("Our account token balance:", balance.toString());

    console.log(`Total formatted supply: ${formatUnits(value.toString(), decimals.toNumber())} ${symbol}`);
    console.log(`Total formatted account token balance: ${formatUnits(balance.toString(), decimals.toNumber())} ${symbol}`);
```

This concludes our script.

<details>

<summary>Your script/getACAinfo.js should look like this:</summary>

```javascript
const PrecompiledToken = artifacts.require("@acala-network/contracts/build/contracts/Token");

const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");
const { formatUnits } = require("ethers/lib/utils")

module.exports = async function(callback) {
  try {
    const accounts = await web3.eth.getAccounts();
    const deployer = accounts[0];

    console.log("Getting contract info with the account:", deployer);

    console.log("Account balance:", formatUnits((await web3.eth.getBalance(deployer)).toString(), 12));

    const instance = await PrecompiledToken.at(ACA);

    console.log("PrecompiledToken address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();
    const decimals = await instance.decimals();
    const value = await instance.totalSupply();
    const balance = await instance.balanceOf(deployer);
    
    console.log("Token name:", name);
    console.log("Token symbol:", symbol);
    console.log("Token decimal spaces:", decimals.toString());
    console.log("Total supply:", value.toString());
    console.log("Our account token balance:", balance.toString());

    console.log(`Total formatted supply: ${formatUnits(value.toString(), decimals.toNumber())} ${symbol}`);
    console.log(`Total formatted account token balance: ${formatUnits(balance.toString(), decimals.toNumber())} ${symbol}`);
  }
  catch(error) {
    console.log(error)
  }

  callback()
}
```

</details>

{% hint style="info" %}
**NOTE: If you wish to modify the script to use another token predeploy, the only thing you need to do is replace ACA address constant with another included within the ADDRESS utility (you could use AUSD for example).**
{% endhint %}

To use the script within the local development network or a public development network, we need to add the following scripts to `scripts` section of your `package.json`:

```shell
    "get-info-mandala": "truffle exec scripts/getACAinfo.js --network mandala",
    "get-info-mandala:pubDev": "truffle exec scripts/getACAinfo.js --network mandalaPublicDev",
```

Running the `yarn get-info-mandala` script should return the following output:

```shell
yarn get-info-mandala

yarn run v1.22.17
$ truffle exec scripts/getACAinfo.js --network mandala
Using network 'mandala'.

Getting contract info with the account: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Account balance: 10000913998512.569816
PrecompiledToken address: 0x0000000000000000000100000000000000000000
Token name: Acala
Token symbol: ACA
Token decimal spaces: 12
Total supply: 140110004600000000000
Our account token balance: 10000913998512569816
Total formatted supply: 140110004.6 ACA
Total formatted account token balance: 10000913.998512569816 ACA
✨  Done in 1.91s.
```

## Summary

We have built upon the knowledge on how to interact with the Acala EVM+ and gotten familiar with Acala EVM+ precompiles and predeploys. To run the test we can use the `yarn test-mandala` and `yarn test-madala:pubDev` and to run the information script we can use the `yarn get-info-mandala` and `yarn get-info-mandala:pubDev`. As we are using utilities only available in the Acala EVM+, we can no longer use a conventional development networks like Ganache.
