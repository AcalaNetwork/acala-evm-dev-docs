---
description: >-
  A tutorial on how to build, test and deploy a simple interactable smart
  contract in Acala EVM+.
---

# Echo tutorial

## Table of contents

* [About](broken-reference)
* [Smart contract](echo-tutorial.md#smart-contract)
* [Test](echo-tutorial.md#test)
* [Deploy script](echo-tutorial.md#deploy-script)
* [Summary](echo-tutorial.md#summary)

## About

This is an example that builds upon the [hello-world example](helloworld-tutorial.md) with additional functionalities, like support for events, interactable public functions and private variables. As the hello-world already contains an example on how to build the project, we will only focus on building the smart contract, test and deploy scripts. For the setup and naming, replace the hello-world with echo. Let's jump into it!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/truffle-tutorials/tree/master/echo](https://github.com/AcalaNetwork/truffle-tutorials/tree/master/echo)
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

Your test file should be called `x_echo.js` and the empty test along with the import statement should look like this:

```javascript
const Echo = artifacts.require("Echo");

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract("Echo", function (/* accounts */) {
  
});
```

To prepare for the testing, we have to define `instance` global variable. The `instance` will store the deployed Echo smart contract. Let's assign it a value in the `beforeEach` action:

```javascript
  let instance;

  beforeEach("setup development environment", async function () {
    instance = await Echo.deployed();
  });
```

Our test will be split into two sections, `Deployment` and `Operation`:

```javascript
  describe("Deployment", function () {

  });

  describe("Operation", function () {
    
  });
```

Within `Deployment` describe block we will ensure that the test suite works as expected and validate that the `echo` variable is set to `Deployed successfully!`:

```javascript
    it("should assert true", async function () {
      return assert.isTrue(true);
    });

    it("returns the right value after the contract is deployed", async function() {
      const echo = await instance.echo();

      expect(echo).to.equal("Deployed successfully!");
    });
```

We can now add the following test cases to our describe block:

1. The contract should update the `echo` variable when `scream()` is called.
2. When the `echo` variable is changed, the `NewEcho` should be emitted.
3. The `echoCount` should be incremented when new string is saved to the `echo` variable.

The test cases of the `Operation` describe block should look like this:

```javascript
    it("should update the echo variable", async function () {
      await instance.scream("Hello World!");

      expect(await instance.echo()).to.equal("Hello World!");
    });

    it("shold emit NewEcho event", async function () {
      const response = await instance.scream("Hello World!");

      expect(response.logs[0].event).to.equal("NewEcho");
    });

    it("should increment echo counter in the NewEcho event", async function () {
      const initialResponse = await instance.scream("Hello World!");

      const finalResponse = await instance.scream("Goodbye World!");

      expect(finalResponse.logs[0].args.message).to.equal("Goodbye World!");
      expect(finalResponse.logs[0].args.count.toNumber()).to.equal(initialResponse.logs[0].args.count.toNumber() + 1);
    });
```

With that, our test is ready to be run.

<details>

<summary>Your test/x_echo.js should look like this:</summary>

```javascript
const Echo = artifacts.require("Echo");

/*
* uncomment accounts to access the test accounts made available by the
* Ethereum client
* See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
*/
contract("Echo", function (/* accounts */) {
  let instance;

  beforeEach("setup development environment", async function () {
    instance = await Echo.deployed();
  });

  describe("Deployment", function () {
    it("should assert true", async function () {
      return assert.isTrue(true);
    });

    it("returns the right value after the contract is deployed", async function() {
      const echo = await instance.echo();

      expect(echo).to.equal("Deployed successfully!");
    });
  });

  describe("Operation", function () {
    it("should update the echo variable", async function () {
      await instance.scream("Hello World!");

      expect(await instance.echo()).to.equal("Hello World!");
    });

    it("shold emit NewEcho event", async function () {
      const response = await instance.scream("Hello World!");

      expect(response.logs[0].event).to.equal("NewEcho");
    });

    it("should increment echo counter in the NewEcho event", async function () {
      const initialResponse = await instance.scream("Hello World!");

      const finalResponse = await instance.scream("Goodbye World!");

      expect(finalResponse.logs[0].args.message).to.equal("Goodbye World!");
      expect(finalResponse.logs[0].args.count.toNumber()).to.equal(initialResponse.logs[0].args.count.toNumber() + 1);
    });
  });
});
```

</details>

****

{% hint style="warning" %}
**NOTE: You need to add a deployment script for `Echo` before you can run the tests.**
{% endhint %}

When you run the test with (for example) `yarn test-mandala`, your tests should pass with the following output:

```shell
yarn test-mandala


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ truffle test --network mandala
Using network 'mandala'.


Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.

Deploying Echo
Echo deployed at: 0xf80A32A835F79D7787E8a8ee5721D0fEaFd78108


  Contract: Echo
    Deployment
      ✓ should assert true
      ✓ returns the right value after the contract is deployed (1144ms)
    Operation
      ✓ should update the echo variable (5123ms)
      ✓ shold emit NewEcho event (9937ms)
      ✓ should increment echo counter in the NewEcho event (15529ms)


  5 passing (41s)

✨  Done in 82.05s.
```

## Deploy script

This deployment script will deploy the contract and output its address.

Within the `x_echo.js` we will import the `Echo` smart contract and have the blank migration ready. We do this by placing the following code within the file:

```javascript
const Echo = artifacts.require("Echo");

module.exports = async function (deployer) {

};
```

Within the script, we first log the `Deploying Echo` to the console, to signal the start of the deployment, then we deploy it and log its address to the console:

```javascript
  console.log("Deploying Echo");

  await deployer.deploy(Echo);

  console.log("Echo deployed at:", Echo.address);
```

<details>

<summary>Your script/x_echo.js should look like this:</summary>

```javascript
const Echo = artifacts.require("Echo");

module.exports = async function (deployer) {
  console.log("Deploying Echo");

  await deployer.deploy(Echo);

  console.log("Echo deployed at:", Echo.address);
};
```

</details>

Running the `yarn deploy-mandala` script should return the following output:

```shell
yarn deploy-mandala


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ truffle migrate --network mandala

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'mandala'
> Network id:      595
> Block gas limit: 15000000 (0xe4e1c0)


1_initial_migration.js
======================

   Replacing 'Migrations'
   ----------------------
   > transaction hash:    0xa2f826fac3b11aba0b1b19226716f4cc38a9a959679baf3435fd9710586b9cb4
   > Blocks: 1            Seconds: 4
   > contract address:    0xc374eB17f665914c714Ac4cdC8AF3a3474228cc5
   > block number:        203
   > block timestamp:     1638183162016
   > account:             0x75E480dB528101a381Ce68544611C169Ad7EB342
   > balance:             9999996.085190716982
   > gas used:            0 (0x0)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:                   0 ETH


1637792775_echo.js
==================
Deploying Echo

   Replacing 'Echo'
   ----------------
   > transaction hash:    0x4e6d554743d48596d482f9713c025895b437823d66e34af2e43258d1e3663f78
   > Blocks: 1            Seconds: 4
   > contract address:    0x2eA9df3bABe04451c9C3B06a2c844587c59d9C37
   > block number:        205
   > block timestamp:     1638183174001
   > account:             0x75E480dB528101a381Ce68544611C169Ad7EB342
   > balance:             9999994.702387153408
   > gas used:            0 (0x0)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH

Echo deployed at: 0x2eA9df3bABe04451c9C3B06a2c844587c59d9C37

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:                   0 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0 ETH


✨  Done in 52.94s.
```

## Summary

We have built upon the first example and added a smart contract with more functionalities and tested all of them. The tests were more detailed and covered more examples. We also ensured that we can interact with the smart contract and that we can modify its storage. We can compile smart contract `yarn build`, test it with `yarn test` or `yarn test-mandala` and deploy it with `yarn deploy` or `yarn deploy-mandala`.
