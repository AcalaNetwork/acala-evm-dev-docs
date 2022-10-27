---
description: >-
  Tutorial on how to build, test and deploy upgradeable smart contracts in the
  Acala EVM+.
---

# UpgradeableGreeter tutorial

## Table of contents

* [Intro](upgradeablegreeter-tutorial.md#intro)
* [Setting up](upgradeablegreeter-tutorial.md#setting-up)
* [Smart contract](upgradeablegreeter-tutorial.md#smart-contract)
* [Deploy script](upgradeablegreeter-tutorial.md#deploy-script)
* [Test](upgradeablegreeter-tutorial.md#test)
* [Conclusion](upgradeablegreeter-tutorial.md#conclusion)

## Intro

This tutorial addresses the use of the upgradeable smart contracts using the proxy-upgrade development pattern. Since Acala EVM+ charges storage rent to the user modifying the storage, we have to accommodate this when deploying the smart contracts (which modify more storage than an ordinary smart contract call). Since `upgrades` method of `@openzepelin/hardhat-upgrades` dependency, which is used to deploy and upgrade the upgradeable smart contracts, doesn't accept transaction parameters, we need to overwrite the provider it uses, to ensure that correct parameters are provided to it.

Let’s jump right in!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/upgradeable-greeter](https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/upgradeable-greeter)
{% endhint %}

## Setting up

The tutorial project will live in the `upgradeable-greeter/` folder. We can create it using `mkdir upgradeable-greeter`. As we will be using Hardhat development framework, we need to initiate the `yarn` project and add `hardhat` as a development dependency:

```shell
yarn init && yarn add --dev hardhat
```

{% hint style="info" %}
**NOTE: This example can use the default yarn project settings, which means that all of the prompts can be responded to with pressing `enter`.**
{% endhint %}

Now that the `hardhat` dependency is added to the project, we can initiate a simple hardhat project with `yarn hardhat`:

```shell
➜  advanced-escrow yarn hardhat
yarn run v1.22.17

888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

👷 Welcome to Hardhat v2.8.3 👷‍

? What do you want to do? … 
❯ Create a basic sample project
  Create an advanced sample project
  Create an advanced sample project that uses TypeScript
  Create an empty hardhat.config.js
  Quit
```

When the Hardhat prompt appears, selecting the first option will give us an adequate project skeleton that we can modify.

{% hint style="info" %}
**NOTE: Once again, the default settings from Hardhat are acceptable, so we only need to confirm them using the `enter` key.**
{% endhint %}

As we will be using the Mandala test network, we need to add it to `hardhat.config.js`. Networks are added in the `module.exports` section below the `solidity` compiler version configuration. We will be adding two networks to the configuration. The local development network, which we will call `mandala`, and the public test network, which we will call `mandalaPubDev`:

```javascript
 networks: {
   mandala: {
     url: 'http://127.0.0.1:8545',
     accounts: {
       mnemonic: 'fox sight canyon orphan hotel grow hedgehog build bless august weather swarm',
       path: "m/44'/60'/0'/0",
     },
     chainId: 595,
   },
   mandalaPubDev: {
     url: 'https://eth-rpc-mandala.aca-staging.network',
     accounts: {
       mnemonic: YOUR_MNEMONIC,
       path: "m/44'/60'/0'/0",
     },
     chainId: 595,
     timeout: 60000,
   },
 }
```

Let’s take a look at the network configurations:

* `url`: Used to specify the RPC endpoint of the network
* `accounts`: Section to describe how Hardhat should acquire or derive the EVM accounts
* `mnemonic`: Mnemonic used to derive the accounts. **Add your mnemonic here**
* `path`: Derivation path to create the accounts from the mnemonic
* `chainId`: Specific chain ID of the Mandala chain. The value of `595` is used for both, local development network as well as the public test network
* `timeout`: An override value for the built in transaction response timeout. It is needed only on the public test network

With that, our project is ready for development.

## Smart contract

Our original `Greeter` smart contract will accept the initializer parameter with the initial greeting. We will have the ability to get the current greeting and update it.

Hardhat has already created a smart contract within the `contracts/` folder when we ran its setup. This smart contract is named `Greeter`. We can just overw**r**ite it with our own if there are any differences:

```solidity
pragma solidity =0.8.9;

import "hardhat/console.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract Greeter is Initializable  {
    string private greeting;

    function initialize(string memory _greeting) public initializer {
        console.log("Deploying a Greeter with greeting:", _greeting);
        greeting = _greeting;
    }

    function greet() public view returns (string memory) {
        return greeting;
    }

    function setGreeting(string memory _greeting) public {
        console.log("Changing greeting from '%s' to '%s'", greeting, _greeting);
        greeting = _greeting;
    }
}
```

We are using OpenZeppelin's upgradeable smart contracts, so we need to add the `contracts-upgradeable` dependency:

```shell
yarn add --dev @openzeppelin/contracts-upgradeable
```

{% hint style="info" %}
**NOTE: You can read up on the upgradeable smart contracts in the official OpenZeppelin** [**documentation**](https://docs.openzeppelin.com/learn/upgrading-smart-contracts?pref=hardhat)**.**
{% endhint %}

This wraps up our `Greeter` smart contract.

<details>

<summary>Your <code>contracts/Greeter.sol</code> should look like this:</summary>

```solidity
//SPDX-License-Identifier: Unlicense
    pragma solidity =0.8.9;

    import "hardhat/console.sol";
    import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

    contract Greeter is Initializable  {
            string private greeting;

            function initialize(string memory _greeting) public initializer {
                    console.log("Deploying a Greeter with greeting:", _greeting);
                    greeting = _greeting;
            }

            function greet() public view returns (string memory) {
                    return greeting;
            }

            function setGreeting(string memory _greeting) public {
                    console.log("Changing greeting from '%s' to '%s'", greeting, _greeting);
                    greeting = _greeting;
            }
    }
```

</details>

In addition to the original `Greeter` smart contract, we will add an upgraded version of the smart contract called `GreeterV2`. To add it, use:

```shell
touch contracts/GreeterV2.sol
```

Add a pragma definition and the imports of `hardhat/console`, `@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol` and the original `Greeter` smart contract. We will be setting the inheritance of out `GreeterV2` to inherit `Greeter`:

```solidity
pragma solidity =0.8.9;

import "hardhat/console.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "./Greeter.sol";

contract GreeterV2 is Greeter  {
    string private greeting;
    function setGreetingV2(string memory _greeting) public {
        string memory newGreeting = string(abi.encodePacked(_greeting, " - V2"));
        console.log("<V2> Changing greeting from '%s' to '%s'", greeting, newGreeting);
        setGreeting(newGreeting);
    }
}
```

We have to define the greeting variable in order to avoid storage collision and we will be adding another `setGreeting`, which will append `- V2` to the end of the string we are adding:

```solidity
    string private greeting;

    function setGreetingV2(string memory _greeting) public {
        string memory newGreeting = string(abi.encodePacked(_greeting, " - V2"));
        console.log("<V2> Changing greeting from '%s' to '%s'", greeting, newGreeting);
        setGreeting(newGreeting);
    }
```

This concludes our `GreeterV2` smart contract.

<details>

<summary>Your <code>contracts/GreeterV2.sol</code> should look like this:</summary>

```solidity
//SPDX-License-Identifier: Unlicense
    pragma solidity =0.8.9;

    import "hardhat/console.sol";
    import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
    import "./Greeter.sol";

    contract GreeterV2 is Greeter  {
            string private greeting;

            function setGreetingV2(string memory _greeting) public {
                    string memory newGreeting = string(abi.encodePacked(_greeting, " - V2"));
                    console.log("<V2> Changing greeting from '%s' to '%s'", greeting, newGreeting);
                    setGreeting(newGreeting);
            }
    }
```

</details>

In order to be able to compile our smart contract with the `yarn build` command, we need to add the custom build command to `package.json`. We do this by adding a `”scripts”` section below the `"devDependencies”` section and defining the `"build”` command within it:

```json
 "scripts": {
   "build": "hardhat compile"
 }
```

With that, the smart contract can be compiled using:

```shell
yarn build
```

## Deploy script

### Override provider utility

In order to be able to deploy your smart contract to the Acala EVM+ using Hardhat, you need to pass custom transaction parameters to the deploy transactions. In most cases, we can override them within the call, but this is not the case with `upgrades` from OpenZeppelin. Because of this, we need to override the provider used by the Hardhat and connect a `Signer` to it. We will then be able to use this signer to communicate with the network and the default parameters will suffice for us to deploy an upgradeable smart contract.

First we need to add the dependency to the project:

```shell
yarn add --dev @acala-network/eth-providers
```

Now that we have the required dependency added to the project, we can create the utility:

```shell
mkdir utils && touch utils/overrideProvider.js
```

The `ethers` and `EvmRpcProvider` are imported at the top of the file and let's define the `providerOverrides()` below it:

```javascript
const { ethers } = require('hardhat');
const { EvmRpcProvider } = require('@acala-network/eth-providers');

async function providerOverrides() {

}
```

Within the `providerOverrides()` function, we set the parameters needed in order to create our new provider and instantiate the desired signer. We do this by adding an `ENDPOINT_URL`, which will point to the network node, and the `MNEMONIC`, which will hold the mnemonic used to instantiate the `Signer`. After we instantiate the provider, we set the overriden gas prices needed in order to be able to deploy an upgradeable smart contract. Once the gas price is overriden, we can finally create the `Signer` and connect it to our custom provider. In order to be able to use them, we need to return them at the end of the function:

```javascript
    const ENDPOINT_URL = process.env.ENDPOINT_URL || "ws://localhost:9944";
    const MNEMONIC = process.env.MNEMONIC || "fox sight canyon orphan hotel grow hedgehog build bless august weather swarm";

    const provider = EvmRpcProvider.from(ENDPOINT_URL);
    await provider.isReady();

    const gasPriceOverrides = (await provider._getEthGas()).gasPrice;

    provider.getFeeData = async () => ({
        maxFeePerGas: null,
        maxPriorityFeePerGas: null,
        gasPrice: gasPriceOverrides,
    });

    const signer = ethers.Wallet.fromMnemonic(MNEMONIC).connect(provider);

    return{
        provider: provider,
        signer: signer
    };
```

In order to be able to use the `providerOverrides` from our new utility, we have to export it at the bottom of the utility:

```javascript
module.exports = { providerOverrides };
```

This concludes the `overrideProvider` and we can move on to writing the deploy script where we will use it.

<details>

<summary>Your <code>utils/overrideProvider.js</code> should look like this:</summary>

```javascript
    const { ethers } = require('hardhat');
    const { EvmRpcProvider } = require('@acala-network/eth-providers');

    async function providerOverrides() {
            const ENDPOINT_URL = process.env.ENDPOINT_URL || "ws://localhost:9944";
            const MNEMONIC = process.env.MNEMONIC || "fox sight canyon orphan hotel grow hedgehog build bless august weather swarm";

            const provider = EvmRpcProvider.from(ENDPOINT_URL);
            await provider.isReady();

            const gasPriceOverrides = (await provider._getEthGas()).gasPrice;

            provider.getFeeData = async () => ({
                    maxFeePerGas: null,
                    maxPriorityFeePerGas: null,
                    gasPrice: gasPriceOverrides,
            });

            const signer = ethers.Wallet.fromMnemonic(MNEMONIC).connect(provider);

            return{
                    provider: provider,
                    signer: signer
            };
    }

    module.exports = { providerOverrides };
```

</details>

### Script

Now that we have our smart contract ready, we can deploy it.

Initiating Hardhat also created a `scripts` folder and within it a sample script. We will remove it and add our own deploy script instead:

```shell
rm scripts/sample-script.js && touch scripts/deploy.js
```

Let’s add a skeleton `main` function within the `deploy.js` and make sure it’s executed when the script is called:

```javascript
async function main() {
 
}
 
main()
 .then(() => process.exit(0))
 .catch((error) => {
   console.error(error);
   process.exit(1);
 });
```

Now that we have the skeleton deploy script, we can import the `providerOverrides` from the `overrideProvider` we added in the subsection above at the top of the file:

```javascript
const { providerOverrides } = require('../utils/overrideProvider');
```

We need to import `ethers` and `upgrades` from `hardhat`:

```javascript
const { ethers, upgrades } = require('hardhat');
```

{% hint style="warning" %}
**NOTE: Make sure to add `require('@openzeppelin/hardhat-upgrades');` to the `hardhat.config.js` in order to be able to use the `upgrades`.**
{% endhint %}

At the beginning of the `main` function definition, we will set the overriden provider, by invoking the `providerOverrides`:

```javascript
  const overrides = await providerOverrides();
```

Now that we have the overriden provider, we can deploy the upgradeable smart contract. We need to get the signer which will be used to deploy the smart contract, then we instantiate the smart contract within the contract factory and deploy it using the `upgrades`. Once the smart contract is successfully deployed, we will log its address to the console and get and output the greeting to the console. As well:

```javascript
  const overrides = await providerOverrides();

  const deployer = overrides.signer;

  console.log('Deploying contract with the account:', deployer.address);

  console.log('Account balance:', (await deployer.getBalance()).toString());

  const Greeter = await ethers.getContractFactory('Greeter', deployer);
  const instance = await upgrades.deployProxy(Greeter, ["Hello, Goku!"]);

  console.log('Greeter address:', instance.address);

  const greeting = await instance.greet();

  console.log('Greeting is:', greeting);
```

With that, our deploy script is ready to be run.

<details>

<summary>Your <code>scripts/deploy.js</code> should look like this:</summary>

```javascript
    const { providerOverrides } = require('../utils/overrideProvider');
    const { ethers, upgrades } = require('hardhat');

    async function main() {
            const overrides = await providerOverrides();

            const deployer = overrides.signer;

            console.log('Deploying contract with the account:', deployer.address);

            console.log('Account balance:', (await deployer.getBalance()).toString());

            const Greeter = await ethers.getContractFactory('Greeter', deployer);
            const instance = await upgrades.deployProxy(Greeter, ["Hello, Goku!"]);

            console.log('Greeter address:', instance.address);

            const greeting = await instance.greet();

            console.log('Greeting is:', greeting);
    }

    main()
            .then(() => process.exit(0))
            .catch((error) => {
                    console.error(error);
                    process.exit(1);
            });
```

</details>

In order to be able to run the `deploy.js` script, we need to add a script to the `package.json`. To add our custom script to the `package.json`, we need to place our custom script into the `"scripts”` section. Let’s add two scripts, one for the local development network and one for the public test network:

```json
   "deploy-mandala": "hardhat run scripts/deploy.js --network mandala",
   "deploy-mandala:pubDev": "hardhat run scripts/deploy.js --network mandalaPubDev"
```

To make sure each time the deploy script is called, a new proxy is deployed as well, we have to add a cleanup script to the `package.json`:

```json
    "clean": "rm -rf .openzeppelin/"
```

To ease the deploy procedure and not have to call two scripts, we can add the `clean` script to the deployment ones. The deploy scripts should be updated to look like this:

```json
    "deploy-mandala": "yarn clean && hardhat run scripts/deploy.js --network mandala",
    "deploy-mandala:pubDev": "yarn clean && hardhat run scripts/deploy.js --network mandalaPubDev"
```

With that, we are able to run the deploy script using `yarn deploy-mandala` or `yarn deploy-mandala:pubDev`. Using the former command should result in the following output:

```shell
yarn deploy-mandala

yarn run v1.22.19
$ yarn clean && hardhat run scripts/deploy.js --network mandala
$ rm -rf .openzeppelin/

  ------------------------------------------
  ⚡️ running in production (standard) mode ⚡️
  ------------------------------------------

Deploying contract with the account: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Account balance: 10000979176141851632000000
Greeter address: 0x721DbA5CE403DC1b04c7C2Bf8235761dDac1ebBb
Greeting is: Hello, Goku!
✨  Done in 3.29s.
```

Once the original smart contract is deployed, we can upgrade it. To do that, we need to add an upgrade script. Let's add it:

```shell
touch scripts/upgrade.js
```

Much like in the `deploy` script, we have to import the `providerOverrides`, `ethers` and `upgrades`. Additionally let's define the proxyAddress, which will hold the value of the address at which the proxy is deployed. Together with definition and invoking of `main` function, the file should look like this:

```javascript
const { providerOverrides } = require('../utils/overrideProvider');
const { ethers, upgrades } = require('hardhat');

const proxyAddress = '0x721DbA5CE403DC1b04c7C2Bf8235761dDac1ebBb';

async function main() {
        
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Within the `main` function, we first invoke the `providerOverrides` and instantiate our `Signer`. After we output its address and its balance to the console, we instantiate the upgraded smart contract to the `GreeterV2` and deploy it using the `upgrades` method. Finally we verify that the upgrade worked and output the greetings that demonstrate the expected operation:

```javascript
  const overrides = await providerOverrides();

  const deployer = overrides.signer;

  console.log('Upgrading contract with the account:', deployer.address);

  console.log('Account balance:', (await deployer.getBalance()).toString());

  const GreeterV2 = await ethers.getContractFactory('GreeterV2', deployer);
  const instance = await upgrades.upgradeProxy(proxyAddress, GreeterV2);

  console.log('GreeterV2 address:', instance.address);

  const originalGreeting = await instance.greet();

  await instance.setGreetingV2('Konichiwa, Kakarot!');

  const updatedGreeting = await instance.greet();

  console.log('Greeting is:', originalGreeting);
  console.log('Updated greeting:', updatedGreeting);
```

This concludes the `upgrade` script.

<details>

<summary>Your <code>scripts/upgrade.js</code> should look like this:</summary>

```javascript
    const { providerOverrides } = require('../utils/overrideProvider');
    const { ethers, upgrades } = require('hardhat');

    const proxyAddress = '0x721DbA5CE403DC1b04c7C2Bf8235761dDac1ebBb';

    async function main() {
            const overrides = await providerOverrides();

            const deployer = overrides.signer;

            console.log('Upgrading contract with the account:', deployer.address);

            console.log('Account balance:', (await deployer.getBalance()).toString());

            const GreeterV2 = await ethers.getContractFactory('GreeterV2', deployer);
            const instance = await upgrades.upgradeProxy(proxyAddress, GreeterV2);

            console.log('GreeterV2 address:', instance.address);

            const originalGreeting = await instance.greet();

            await instance.setGreetingV2('Konichiwa, Kakarot!');

            const updatedGreeting = await instance.greet();

            console.log('Greeting is:', originalGreeting);
            console.log('Updated greeting:', updatedGreeting);
    }

    main()
            .then(() => process.exit(0))
            .catch((error) => {
                    console.error(error);
                    process.exit(1);
            });
```

</details>

{% hint style="warning" %}
**NOTE: Remember to replace the `proxyAddress` with your own proxy address in order to be able to use the `upgrade` script.**
{% endhint %}

To be able to run the `upgrade` script, we need to add it to the `scripts` section of the `package.json`:

```json
    "upgrade-mandala": "hardhat run scripts/upgrade.js --network mandala",
    "upgrade-mandala:pubDev": "hardhat run scripts/upgrade.js --network mandalaPubDev"
```

Now we are able to upgrade our smart contract using either `upgrade-mandala` or the `upgrade-mandala:pubDev` scripts. Running the `upgrade-mandala` should return:

```shell
yarn upgrade-mandala

yarn run v1.22.19
$ hardhat run scripts/upgrade.js --network mandala

  ------------------------------------------
  ⚡️ running in production (standard) mode ⚡️
  ------------------------------------------

Upgrading contract with the account: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Account balance: 10000974390250157973000000
GreeterV2 address: 0x721DbA5CE403DC1b04c7C2Bf8235761dDac1ebBb
Greeting is: Hello, Goku!
Updated greeting: Konichiwa, Kakarot! - V2
✨  Done in 2.76s.
```

This concludes our scripts.

## Test

Initiating Hardhat also created a `test` folder and within it a sample test. We will remove it and add our own test instead:

```shell
rm test/sample-test.js && touch test/UpgradeableGreeter.js
```

At the beginning of the file, we will be importing all of the methods that we will require to successfully run the tests. We will be using `chai`'s `expect` and `hardhat`'s `ethers` and `upgrades`. To be able to deploy the upgradeable smart contract for testing, we have to import `providerOverrides` that we added to the `overrideProvider` utility before. As initiating the `EvmRpcProvider` generates a lot of output, our test console output would get very messy if we didn't silence it. To do this we use the `console.mute` dependency, that we have to add to the project by using:

```shell
yarn add --dev console.mute
```

The imports along with the empty test should look like this:

```javascript
const { expect } = require('chai');
const { ethers, upgrades } = require('hardhat');

const { providerOverrides } = require('../utils/overrideProvider');

require('console.mute');

describe('UpgradeableGreeter contract', function () {

});
```

To setup for each of the test examples we define the `Greeter` variable that will hold the contract factory for our original smart contract and `GreeterV2` that will hold the contract factory of our upgraded smart contract. The `instance` variable will hold the instance of the original smart contract that we will be testing against. We won't be setting the instance of the upgradeable smart contract before running the tests, because we will do that only when necessary. The `deployer` will hold the `Signer` connected to our overriden provider in order to successfully deploy the upgradeable smart contracts. We have to make sure we mute the console output before assigning the value to the `deployer` and unmute it afterwards, to make sure the console behaves as expected. All of the values are assigned in the `beforeEach` action:

```javascript
  let Greeter;
  let GreeterV2;
  let instance;
  let deployer;

  beforeEach(async function () {
    console.mute();
    deployer = await providerOverrides();
    console.resume();
    Greeter = await ethers.getContractFactory('Greeter', deployer);
    GreeterV2 = await ethers.getContractFactory('GreeterV2', deployer);
    instance = await upgrades.deployProxy(Greeter, ['Hello, Goku!']);
  });
```

Our test cases will be split into two groups. One will be called `Deployment` and it will verify that the deployed smart contract has expected values set before it is being used. The second one will be called `Upgrade` and it will validate the expected behaviour of our updated smart contract. The empty sections should look like this:

```javascript
  describe("Deployment", function () {

  });

  describe("Upgrade", function () {

  });
```

We will be verifying that the vale passed to the `initialize` is returned when calling `greet` and that the value can be updated by the `setGreeting`:

```javascript
    it('should return the greeting set at deployment', async function () {
      expect(await instance.greet()).to.equal('Hello, Goku!');
    });

    it('should return a new greeting when one is set', async function () {
      await instance.setGreeting('Hello, Kakarot!')
      expect(await instance.greet()).to.equal('Hello, Kakarot!');
    });
```

The `Upgrade` section will verify that upgrading the smart contract doesn't overwrite the `greeting` upon the upgrade, that upgrading the smart contract actually adds a new method and that the preexisting method is not overwritten:

```javascript
    it('should maintain the greeting after the upgrade', async function () {
      const upgradedInstance = await upgrades.upgradeProxy(instance.address, GreeterV2);

      expect(await upgradedInstance.greet()).to.equal('Hello, Goku!');
    });

    it('should add a new method', async function () {
      const upgradedInstance = await upgrades.upgradeProxy(instance.address, GreeterV2);

      await upgradedInstance.setGreetingV2('Konichiwa, Kakarot!');

      expect(await upgradedInstance.greet()).to.equal('Konichiwa, Kakarot! - V2');
    });

    it('should maintain the original method', async function () {
      const upgradedInstance = await upgrades.upgradeProxy(instance.address, GreeterV2);

      await upgradedInstance.setGreetingV2('Konichiwa, Kakarot!');

      const updatedGreeting = await instance.greet();

      await upgradedInstance.setGreeting('Goodbye, Goku!');

      const originalGreeting = await instance.greet();

      expect(updatedGreeting).to.equal('Konichiwa, Kakarot! - V2');
      expect(originalGreeting).to.equal('Goodbye, Goku!');
    });
```

This concludes our test.

<details>

<summary>Your <code>test/UpgradeableGreeter.js</code> should look like this:</summary>

```javascript
    const { expect } = require('chai');
    const { ethers, upgrades } = require('hardhat');

    const { providerOverrides } = require('../utils/overrideProvider');

    require('console.mute');

    describe('UpgradeableGreeter contract', function () {
            let Greeter;
            let GreeterV2;
            let instance;
            let deployer;

            beforeEach(async function () {
                    console.mute();
                    deployer = await providerOverrides();
                    console.resume();
                    Greeter = await ethers.getContractFactory('Greeter', deployer);
                    GreeterV2 = await ethers.getContractFactory('GreeterV2', deployer);
                    instance = await upgrades.deployProxy(Greeter, ['Hello, Goku!']);
            });

            describe('Deployment', function () {
                    it('should return the greeting set at deployment', async function () {
                            expect(await instance.greet()).to.equal('Hello, Goku!');
                    });

                    it('should return a new greeting when one is set', async function () {
                            await instance.setGreeting('Hello, Kakarot!')
                            expect(await instance.greet()).to.equal('Hello, Kakarot!');
                    });
            });

            describe('Upgrade', function () {
                    it('should maintain the greeting after the upgrade', async function () {
                            const upgradedInstance = await upgrades.upgradeProxy(instance.address, GreeterV2);

                            expect(await upgradedInstance.greet()).to.equal('Hello, Goku!');
                    });

                    it('should add a new method', async function () {
                            const upgradedInstance = await upgrades.upgradeProxy(instance.address, GreeterV2);

                            await upgradedInstance.setGreetingV2('Konichiwa, Kakarot!');

                            expect(await upgradedInstance.greet()).to.equal('Konichiwa, Kakarot! - V2');
                    });

                    it('should maintain the original method', async function () {
                            const upgradedInstance = await upgrades.upgradeProxy(instance.address, GreeterV2);

                            await upgradedInstance.setGreetingV2('Konichiwa, Kakarot!');

                            const updatedGreeting = await instance.greet();

                            await upgradedInstance.setGreeting('Goodbye, Goku!');

                            const originalGreeting = await instance.greet();

                            expect(updatedGreeting).to.equal('Konichiwa, Kakarot! - V2');
                            expect(originalGreeting).to.equal('Goodbye, Goku!');
                    });
            });
    });
```

</details>

As our test is ready to be run, we have to add the scripts to be able to run the test. We will be adding two scripts. One to run the tests on the local development network and on the public test network. We also have to make sure that we include the `clear` script to be run before running the test:

```json
    "test-mandala": "yarn clean && hardhat test test/UpgradeableGreeter.js --network mandala",
    "test-mandala:pubDev": "yarn clean && hardhat test test/UpgradeableGreeter.js --network mandalaPubDev"
```

Running the tests with `test-mandala` should give you the following output:

```shell
yarn test-mandala

yarn run v1.22.19
$ yarn clean && hardhat test test/UpgradeableGreeter.js --network mandala
$ rm -rf .openzeppelin/


  UpgradeableGreeter contract
    Deployment
      ✔ should return the greeting set at deployment
      ✔ should return a new greeting when one is set (138ms)
    Upgrade
      ✔ should maintain the greeting after the upgrade (641ms)
      ✔ should add a new method (984ms)
      ✔ should maintain the original method (1083ms)


  5 passing (11s)

✨  Done in 12.47s.
```

## Conclusion

We have successfully built an upgradeable `Greeter` smart contract that allows users set a greeting and retrieve it. We added scripts and commands to run those scripts. To compile the smart contract, use `yarn build`. In order to deploy the smart contract to a local development network use `yarn deploy-mandala` and to deploy it to a public test network use `yarn deploy-mandala:pubDev`. We also created a set of example tests to validate the expected behaviour on the local development network with `yarn test-mandala` and the public development network `yarn test-mandala:pubDev`.
