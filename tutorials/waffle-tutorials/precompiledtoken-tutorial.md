---
description: >-
  A tutorial showcasing interaction with Acala EVM+'s precompiled and
  predeployed smart contracts.
---

# PrecompiledToken tutorial

## Table of contents

* [About](precompiledtoken-tutorial.md#about)
* [Smart contract](precompiledtoken-tutorial.md#smart-contract)
* [Test](precompiledtoken-tutorial.md#test)
* [Script](precompiledtoken-tutorial.md#script)
* [Summary](precompiledtoken-tutorial.md#summary)

## About

This example introduces the use of Acala EVM+ precompiles and predeploys that are present on every network at a fixed address (the address of a predeployed contract is the same on a local development network, public test network as well as the production network). As this example focuses on showcasing the precompiles and predeploys, it doesn't have a smart contract. We will however interact with an `ERC20` smart contract that is already deployed to the network and we will get all of the required imports from the [`@acala-network/contracts`](https://github.com/AcalaNetwork/predeploy-contracts) dependency. The precompiles and predeploys are a specific feature of the Acala EVM+, so this and the following tutorials are no longer compatible with traditional EVM development networks (like Ganache). Let's take a look!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/waffle-tutorials/tree/master/precompiled-token](https://github.com/AcalaNetwork/waffle-tutorials/tree/master/precompiled-token)
{% endhint %}

## Smart contract

As mentioned in the introduction, this tutorial doesn't include the smart contract, so the `contracts` folder can be removed as well. We will however be using the `@acala-network/contracts` dependency in order to gain access to the precompiled resources of a `Token` smart contract. To add it to your project you can use:

```shell
yarn add --dev @acala-network/contracts
```

This allows us to access the `Token` smart contract artifacts, which can be found in `@acala-network/contracts/build/contracts/Token.json`.

## Test

Tests for this tutorial will validate the expected values returned by the ACA token predeployed smart contract. Your test file should be named `PrecompiledToken.test.ts`. Within it we import the `expect` and `use` from `chai` dependency, `solidity` from `ethereum-waffle` dependency and `Contract` from the `ethers` dependency. We are using `Contract` in stead of `ContractFactory`, because the contract is already deployed to the network. The `ADDRESS` utility of `@acala-network/contracts` dependency is imported and it holds the value of all of the address of the predeployed smart contracts in the Acala EVM+. Additionally we are importing the compiled `Token` smart contract from the `@acala-network/contracts` dependency, which we will use to instantiate the smart contract and `getTestProvider` utility, which allows us to overwrite the endpoint to which the tests are connecting if necessary.

{% hint style="info" %}
**NOTE: The ACA ERC20 token mirrors the balance of the native ACA currency, so you are able to transfer ACA within your smart contract the same way you would transfer a non-native ERC20 token.**
{% endhint %}

The test file with import statements and an empty test should look like this:

```typescript
import { expect, use } from 'chai';
import { solidity } from 'ethereum-waffle';
import { Contract } from 'ethers';
import ADDRESS from "@acala-network/contracts/utils/Address"

import { evmChai, Signer, TestProvider } from '@acala-network/bodhi';

import Token from '@acala-network/contracts/build/contracts/Token.json';
import { getTestProvider } from "../utils/setup";

use(solidity);
use(evmChai);

describe("PrecompiledToken", () => {

});
```

First thing to add to the `PrecompiledToken` describe block are the `provider`, `deployer`, `instance` and `deployerAddress` variables. Within the `before` action we assign the `TestProvider` to `provider`, `Signer` to the `deployer` variable. `deployerAddress` is used to store the address of `deployer` and deployed contract is instantiated to `instance`. The `after` action will disconnect from the `provider`:

```typescript
    let provider: TestProvider;
    let deployer: Signer;
    let instance: Contract;
    let deployerAddress: String;

    before(async () => {
      provider = await getTestProvider();
      [deployer] = await provider.getWallets();
      instance = new Contract(ADDRESS.ACA, Token.abi, deployer);
      deployerAddress = await deployer.getAddress();
    });

    after(async () => {
      provider.api.disconnect();
    });
```

You can see how we used the `ACA` from the `ADDRESS` utility in order to set the address of our predeployed smart contract.

Our test will only contain one section called `Precompiled token` in which we will be checking the following examples:

1. The token should return `Acala` when `name()` is called.
2. The token should return `ACA` when `symbol()` is called.
3. The total supply of the token should be greater than 0.
4. The balance of our development address should be greater than 0.

```typescript
    describe("Precompiled token", () => {
      it("should assing correct name to Token", async () => {
        expect(await instance.name()).to.equal("Acala");
      });

      it("should assign correct symbol", async () => {
        expect(await instance.symbol()).to.equal("ACA");
      });

      it("should have the total supply greater than 0", async () => {
        expect((await instance.totalSupply())).to.be.above(0);
      });

      it("should show balance of the deployer address higher than 0", async () => {
        expect((await instance.balanceOf(deployerAddress))).to.be.above(0);
      });
    });
```

With that, our test is ready to be run.

<details>

<summary>Your test/PrecompiledToken.test.ts should look like this:</summary>

```typescript
import { expect, use } from 'chai';
import { solidity } from 'ethereum-waffle';
import { Contract } from 'ethers';
import ADDRESS from "@acala-network/contracts/utils/Address"

import { evmChai, Signer, TestProvider } from '@acala-network/bodhi';

import Token from '@acala-network/contracts/build/contracts/Token.json';
import { getTestProvider } from "../utils/setup";

use(solidity);
use(evmChai);

describe("PrecompiledToken", () => {
    let provider: TestProvider;
    let deployer: Signer;
    let instance: Contract;
    let deployerAddress: String;

    before(async () => {
      provider = await getTestProvider();
      [deployer] = await provider.getWallets();
      instance = new Contract(ADDRESS.ACA, Token.abi, deployer);
      deployerAddress = await deployer.getAddress();
    });

    after(async () => {
      provider.api.disconnect();
    });

    describe("Precompiled token", () => {
      it("should assing correct name to Token", async () => {
        expect(await instance.name()).to.equal("Acala");
      });

      it("should assign correct symbol", async () => {
        expect(await instance.symbol()).to.equal("ACA");
      });

      it("should have the total supply greater than 0", async () => {
        expect((await instance.totalSupply())).to.be.above(0);
      });

      it("should show balance of the deployer address higher than 0", async () => {
        expect((await instance.balanceOf(deployerAddress))).to.be.above(0);
      });
    });
});
```

</details>

When you run the test with `yarn test`, your tests should pass with the following output:

```shell
yarn test


yarn run v1.22.17
$ export NODE_ENV=test && mocha -r ts-node/register/transpile-only --timeout 100000 --no-warnings test/**/*.test.ts

  PrecompiledToken
    Precompiled token
      ✔ should assing correct name to Token
      ✔ should assign correct symbol
      ✔ should have the total supply greater than 0
      ✔ should show balance of the deployer address higher than 0


  4 passing (579ms)

Done in 7.99s.
```

## Script

As the smart contract is already deployed to the network, we don't need a deploy script, but we can add a script that interacts with the predeployed smart contract and log some values from it to the console.

Let's name our script `getACAinfo.ts` and import `ACA` from the `ADDRESS` utility, `Contract` from `ethers`, `formatUnits` from `ethers/libs/utils` and `Token` precompile from `@acala-network/contracts`. The script along with the imports should look like this:

```typescript
import { use } from 'chai';
import { Contract } from 'ethers';
import { formatUnits } from 'ethers/lib/utils';
import ADDRESS from "@acala-network/contracts/utils/Address"

import { evmChai } from '@acala-network/bodhi';

import Token from '@acala-network/contracts/build/contracts/Token.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {

}

main()
```

Within the definition of the `main` function, we first retrieve the `wallet` and `provider` from the `setup()`. Then we output `Retrieve predeployed token information` to the console and instantiate the `Token` smart contract and save it to `instance`. The address of the deployed smart contract is logged into the terminal and we log the information about the token as well. Finally we disconnect from the provider:

```typescript
    const { wallet, provider } = await setup();

    console.log('Retrieve predeployed token information');

    const instance = new Contract(ADDRESS.ACA, Token.abi, wallet);

    console.log("Token address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();
    const decimals = await instance.decimals();
    const totalSupply = await instance.totalSupply();
    
    const walletAddress = await wallet.getAddress();

    const balance = await instance.balanceOf(await wallet.getAddress());

    console.log("Token name:", name);
    console.log("Token symbol:", symbol);
    console.log("Token decimal spaces:", decimals);
    console.log("Token total supply:", totalSupply.toString());
    console.log("Token balance of %s is: %s", walletAddress, balance.toString());

    provider.api.disconnect();
```

As we have the information about how many decimal spaces the token uses, we can add the output of the formatted values to the script. We will use the `formatUnits` utility form `ethers` that we imported. We have to pass the value to be formatted, as a string value, and the number of decimal spaces that the value includes. We can add these formatted balances to the `main()` function at the bottom of its definition, above the `provider.api.disconnect()`:

```typescript
    console.log("Formatted total supply of %s token is: %s %s", name, formatUnits(totalSupply.toString(), decimals), symbol);
    console.log("Formatted %s token balance of %s is: %s %s", name, walletAddress, formatUnits(balance.toString(), decimals), symbol);
```

<details>

<summary>Your src/getACAinfo.ts should look like this:</summary>

```typescript
import { use } from 'chai';
import { Contract } from 'ethers';
import { formatUnits } from 'ethers/lib/utils';
import ADDRESS from '@acala-network/contracts/utils/Address';

import { evmChai } from '@acala-network/bodhi';

import Token from '@acala-network/contracts/build/contracts/Token.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {
    const { wallet, provider } = await setup();

    console.log('Retrieve predeployed token information');

    const instance = new Contract(ADDRESS.ACA, Token.abi, wallet);

    console.log("Token address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();
    const decimals = await instance.decimals();
    const totalSupply = await instance.totalSupply();
    
    const walletAddress = await wallet.getAddress();

    const balance = await instance.balanceOf(await wallet.getAddress());

    console.log("Token name:", name);
    console.log("Token symbol:", symbol);
    console.log("Token decimal spaces:", decimals);
    console.log("Token total supply:", totalSupply.toString());
    console.log("Token balance of %s is: %s", walletAddress, balance.toString());

    console.log("Formatted total supply of %s token is: %s %s", name, formatUnits(totalSupply.toString(), decimals), symbol);
    console.log("Formatted %s token balance of %s is: %s %s", name, walletAddress, formatUnits(balance.toString(), decimals), symbol);

    provider.api.disconnect();
}

main()
```

</details>

{% hint style="info" %}
**NOTE: If you wish to modify the script to use another token precompile, the only thing you need to do is replace ACA address constant with another included within the ADDRESS utility (you could use AUSD for example).**
{% endhint %}

To use the script within the local development network or a public development network, you need to add the following script to `scripts` section of your `package.json`:

```json5
    "get-info": "ts-node --transpile-only src/getACAinfo.ts"
```

Running the `yarn get-info` script should return the following output:

```shell
yarn get-info


yarn run v1.22.17
$ ts-node --transpile-only src/getACAinfo.ts

Retrieve predeployed token information
Token address: 0x0000000000000000000100000000000000000000
Token name: Acala
Token symbol: ACA
Token decimal spaces: 12
Token total supply: 40110004600000000000
Token balance of 0x82A258cb20E2ADB4788153cd5eb5839615EcE9a0 is: 8999996847749727322
Formatted total supply of Acala token is: 40110004.6 ACA
Formatted Acala token balance of 0x82A258cb20E2ADB4788153cd5eb5839615EcE9a0 is: 8999996.847749727322 ACA
Done in 4.79s.
```

## Summary

We have built upon the knowledge on how to interact with the Acala EVM+ and gotten familiar with Acala EVM+ precompiles and predeploys. To run the test we can use the `yarn test` and to run the information script we can use the `yarn get-info`. As we are using utilities only available in the Acala EVM+, we can no longer use a conventional development network like Ganache.
