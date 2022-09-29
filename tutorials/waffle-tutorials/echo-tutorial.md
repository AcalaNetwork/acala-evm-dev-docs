---
description: >-
  A tutorial on how to build, test and deploy a simple interactable smart
  contract in Acala EVM+.
---

# Echo tutorial

## Table of contents

* [About](echo-tutorial.md#about)
* [Smart contract](echo-tutorial.md#smart-contract)
* [Test](echo-tutorial.md#test)
* [Deploy script](echo-tutorial.md#deploy-script)
* [Summary](echo-tutorial.md#summary)

## About

This is an example that builds upon the [hello-world example](helloworld-tutorial.md) with additional functionalities, like support for events, interactable public functions and private variables. As the hello-world already contains an example on how to build the project, we will only focus on building the smart contract, test and deploy scripts. For the setup and naming, replace the hello-world with echo. Let's jump into it!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/waffle-tutorials/tree/master/echo](https://github.com/AcalaNetwork/waffle-tutorials/tree/master/echo)
{% endhint %}

## Smart contract

In this tutorial we will be adding a simple smart contract that has a public variable called `echo`, which stores the latest string passed to a public function. Every time the `echo` variable is changed, the event, containing the latest value as well as the number of time it was changed, is emitted.

Your empty smart contract should look like this:

```solidity
pragma solidity =0.8.9;

contract Echo{
   
}
```

`echo` variable, used to store the string passed to it, is placed at the beginning of the example. Its visibility should be set to public, so that the compiler builds a getter function for it. `echoCount` variable is used to count the number of times the `echo` variable is changed. Additionally we will have a `NewEcho` event that will be emitted every time the `echo` variable is changed and it will contain the new value as well as the number of times the `echo` variable is changed. The content of the smart contract, including these two variables and the event, looks like this:

```solidity
    string public echo;
    uint echoCount;

    event NewEcho(string message, uint count);
```

The `constructor` function can set the initial value of the `echo` variable. Let's set it to `Deployed successfully!`, to signal that the smart contract is ready to use:

```solidity
    constructor() {
        echo = "Deployed successfully!";
    }
```

The last thing to add is a function that allows us to change the value of the `echo` variable. The function should assign the new value to the `echo` variable, increment the `echoCount`, emit the `NewEcho` event and return the input string. Let's call this function `scream()` as it will cause an echo:

```solidity
    function scream(string memory message) public returns(string memory){
        echo = message;
        echoCount += 1;
        emit NewEcho(message, echoCount);
        return message;
    }
```

This concludes our `Echo` smart contract.

<details>

<summary>Your contracts/Echo.sol should look like this:</summary>

```solidity
pragma solidity =0.8.9;

contract Echo{
    string public echo;
    uint echoCount;

    event NewEcho(string message, uint count);

    constructor() {
        echo = "Deployed successfully!";
    }

    function scream(string memory message) public returns(string memory){
        echo = message;
        echoCount += 1;
        emit NewEcho(message, echoCount);
        return message;
    }
}
```

</details>

As the Echo smart contract is ready to be compiled, we can use the `yarn build` command (like we did in the hello-world) to compile the smart contract, which will create the `build` directory and contain the compiled smart contract.

## Test

Your test file should be named `Echo.test.ts` and the empty test along with the import statement should look like this:

```typescript
import { expect, use } from 'chai';
import { deployContract, solidity } from 'ethereum-waffle';
import { Contract, ethers } from 'ethers';

import { evmChai, Signer, TestProvider } from '@acala-network/bodhi';
import { WsProvider } from '@polkadot/api';

import Echo from '../build/Echo.json';
import { getTestProvider } from '../utils/setup';

use(solidity);
use(evmChai);

const ECHO_ABI = require("../build/Echo.json").abi;

describe("Echo", () => {

});
```

In addition to the similar import statements to the ones in the hello-world the `ECHO_ABI` is also imported. It is used for validating the `NewEcho` event.

First thing to add to the `Echo` describe block are the `provider`, `wallet` and `instance` variables. Within the `before` action we assign the `TestProvider` to the `provider`, `Signer` to the `wallet` variable and deployed contract `instance`. The `after` action will disconnect from the `provider`:

```typescript
    let provider: TestProvider;
    let wallet: Signer;
    let instance: Contract;

    before(async () => {
      provider = await getTestProvider();
      [wallet] = await provider.getWallets();
      instance = await deployContract(wallet, Echo);
    });

    after(async () => {
      provider.api.disconnect();
    });
```

There are two describe blocks within `Echo` block. The `Deployment` block validates that the smart contract was deployed as expected and the `Operation` block validates the operation of the smart contract:

```typescript
    describe("Deployment", () => {
        
    });

    describe("Operation", () => {
        
    });
```

The deployment block only has one example that validates that the `echo` variable is assigned the `Deployed successfully!` value:

```typescript
      it("returns the right value after the contract is deployed", async () => {
        console.log(instance.address);
        expect(await instance.echo()).to.equal("Deployed successfully!");
      });
```

The test cases in the `Operation` should validate the following:

1. The contract should update the `echo` variable when `scream()` is called.
2. When the `echo` variable is changed, the `NewEcho` should be emitted.
3. The `echoCount` should be incremented when new string is saved to the `echo` variable.

The test cases of the `Operation` describe block should look like this:

```typescript
      it("should update the echo variable", async () => {
        await instance.scream("Hello World!");

        expect(await instance.echo()).to.equal("Hello World!");
      });

      it("should emit a NewEcho event", async () => {
        await expect(instance.scream("Hello World!")).to
          .emit(instance, "NewEcho");
      });

      it("should increment echo counter in the NewEcho event", async function () {
        let iface = new ethers.utils.Interface(ECHO_ABI);

        let current_block_number = Number(await provider.api.query.system.number());
        await instance.scream("Hello World!");

        let block_hash = await provider.api.rpc.chain.getBlockHash(current_block_number + 1);
        const data = await provider.api.derive.tx.events(block_hash);

        let event = data.events.filter(item => provider.api.events.evm.Executed.is(item.event));
        expect(event.length).above(0);

        let decode_log = iface.parseLog((event[event.length-1].event.data.toJSON() as any)[2][0]);

        const initialCount = decode_log.args.count;

        await expect(instance.scream("Goodbye World!")).to
          .emit(instance, "NewEcho")
          .withArgs("Goodbye World!", initialCount.toNumber() + 1);
      });
```

Let's take a look at the last example. We use the `ECHO_ABI` variable to import the interface of the `Echo` smart contract into `ethers`. Then we get the current block number and initiate the first `scream()` call. Next we retrieve the hash of the block that the transaction including the `scream()` call is in. We then retrieve the `NewEcho` event and get the `count` variable from it. Finally we use it to assert that the second call to the `scream()` increments the `echoCount` variable.

With that, our test is ready to be run.

<details>

<summary>Your test/Echo.test.ts should look like this:</summary>

```typescript
import { expect, use } from 'chai';
import { deployContract, solidity } from 'ethereum-waffle';
import { Contract, ethers } from 'ethers';

import { evmChai, Signer, TestProvider } from '@acala-network/bodhi';
import { WsProvider } from '@polkadot/api';

import Echo from '../build/Echo.json';
import { getTestProvider } from '../utils/setup';

use(solidity);
use(evmChai);

const ECHO_ABI = require("../build/Echo.json").abi;

describe("Echo", () => {
    let provider: TestProvider;
    let wallet: Signer;
    let instance: Contract;

    before(async () => {
        provider = await getTestProvider();
        [wallet] = await provider.getWallets();
        instance = await deployContract(wallet, Echo);
    });

    after(async () => {
        provider.api.disconnect();
    });

    describe("Deployment", () => {
        it("returns the right value after the contract is deployed", async () => {
            console.log(instance.address);
            expect(await instance.echo()).to.equal("Deployed successfully!");
        });
    });

    describe("Operation", () => {
        it("should update the echo variable", async () => {
            await instance.scream("Hello World!");

            expect(await instance.echo()).to.equal("Hello World!");
        });

        it("should emit a NewEcho event", async () => {
            await expect(instance.scream("Hello World!")).to
            .emit(instance, "NewEcho");
        });

        it("should increment echo counter in the NewEcho event", async function () {
            let iface = new ethers.utils.Interface(ECHO_ABI);

            let current_block_number = Number(await provider.api.query.system.number());
            await instance.scream("Hello World!");

            let block_hash = await provider.api.rpc.chain.getBlockHash(current_block_number + 1);
            const data = await provider.api.derive.tx.events(block_hash);

            let event = data.events.filter(item => provider.api.events.evm.Executed.is(item.event));
            expect(event.length).above(0);

            let decode_log = iface.parseLog((event[event.length-1].event.data.toJSON() as any)[2][0]);

            const initialCount = decode_log.args.count;

            await expect(instance.scream("Goodbye World!")).to
            .emit(instance, "NewEcho")
            .withArgs("Goodbye World!", initialCount.toNumber() + 1);
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
$ export NODE_ENV=test && mocha -r ts-node/register/transpile-only --timeout 50000 --no-warnings test/**/*.test.ts


  Echo
    Deployment
0x0230135fDeD668a3F7894966b14F42E65Da322e4
      ✔ returns the right value after the contract is deployed
    Operation
      ✔ should update the echo variable (5967ms)
      ✔ should emit a NewEcho event (6010ms)
      ✔ should increment echo counter in the NewEcho event (11956ms)


  4 passing (55s)

✨  Done in 81.09s.
```

## Deploy script

The `setup.ts` should remain the same as in the hello-world. The `deploy.ts` needs to have the same imports like he hello-world example, except for the smart contract we are importing:

```typescript
import { use } from 'chai';
import { ContractFactory } from 'ethers';

import { evmChai } from '@acala-network/bodhi';

import Echo from '../build/Echo.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {

}

main()
```

Within the definition of the `main` function, we first retrieve the `wallet` and `provider` from the `setup()`. Then we output `Deploy Echo` to the console and deploy the `Echo` smart contract and save it to `instance`. We retrieve the value stored in the `echo` variable, output it to the console, change it and output the new value. Finally we disconnect from the provider:

```typescript
    const { wallet, provider } = await setup();

    console.log('Deploy Echo');

    const instance = await ContractFactory.fromSolidity(Echo).connect(wallet).deploy();

    console.log("Echo address:", instance.address);

    const variable = await instance.echo();

    console.log("Deployment status:", variable);

    await instance.scream("Ready for use!");

    const ready = await instance.echo();

    console.log("Contract status:", ready);

    provider.api.disconnect();
```

<details>

<summary>Your src/deploy.ts should look like this:</summary>

```typescript
import { use } from 'chai';
import { ContractFactory } from 'ethers';

import { evmChai } from '@acala-network/bodhi';

import Echo from '../build/Echo.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {
    const { wallet, provider } = await setup();

    console.log('Deploy Echo');

    const instance = await ContractFactory.fromSolidity(Echo).connect(wallet).deploy();

    console.log("Echo address:", instance.address);

    const variable = await instance.echo();

    console.log("Deployment status:", variable);

    await instance.scream("Ready for use!");

    const ready = await instance.echo();

    console.log("Contract status:", ready);

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
Deploy Echo
Echo address: 0xe381a3D153293a81Dd26C3E6EAd18C74979e5Eb5
Deployment status: Deployed successfully!
Contract status: Ready for use!
✨  Done in 20.28s.
```

## Summary

We have built upon the first example and added a smart contract with more functionalities and tested all of them. The tests were more detailed and covered more examples. We also ensured that we can interact with the smart contract and that we can modify its storage. We can compile smart contract `yarn build`, test it with `yarn test` and deploy it with `yarn deploy`.
