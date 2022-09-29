---
description: >-
  A tutorial showcasing interaction with Acala EVM+'s precompiled and
  predeployed smart contracts.
---

# PrecompiledToken tutorial

### Table of contents

* [About](precompiledtoken-tutorial.md#about)
* [Smart contract](precompiledtoken-tutorial.md#smart-contract)
* [Test](precompiledtoken-tutorial.md#test)
* [Script](precompiledtoken-tutorial.md#script)
* [Summary](precompiledtoken-tutorial.md#summary)

### About

This example introduces the use of Acala EVM+ precompiles and predeploys that are present on every network at a fixed address (the address of a predeployed contract is the same on a local development network, public test network as well as the production network). As this example focuses on showcasing the precompiles and predeploys, it doesn't have a smart contract. We will however interact with an `ERC20` smart contract that is already deployed to the network and we will get all of the required imports from the [`@acala-network/contracts`](https://github.com/AcalaNetwork/predeploy-contracts) dependency. The precompiles and predeploys are a specific feature of the Acala EVM+, so this and the following tutorial is no longer compatible with traditional EVM development networks (like Ganache) or with the Hardhat's build in network emulator. Let's take a look!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/precompiled-token](https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/precompiled-token)
{% endhint %}

### Smart contract

As mentioned in the introduction, this tutorial doesn't include the smart contract, so the `contracts` folder can be removed as well. We will however be using the `@acala-network/contracts` dependency in order to gain access to the precompiled resources of a `Token` smart contract. To add it to your project you can use:

```shell
yarn add --dev @acala-network/contracts
```

This allows us to access the `Token` smart contract artifacts, which can be found in `@acala-network/contracts/build/contracts/Token.json`.

### Test

Tests for this tutorial will validate the expected values returned by the ACA token predeployed smart contract. The test file in our case is called `PrecompiledToken.js`. Within it we import the `expect` from `chai` dependency and `Contract` from the `ethers` dependency. We are using `Contract` in stead of `ContractFactory`, because the contract is already deployed to the network. The `ACA`, which is an export from the `ADDRESS` utility of `@acala-network/contracts` dependency, is imported and it holds the value of the address of the `ACA` token. We will use the `MandalaAddress` to access the addresses of the predeployed smart contracts of the Mandala network. Additionally we are importing the compiled `Token` smart contract from the `@acala-network/contracts` dependency, which we will use to instantiate the smart contract.

{% hint style="info" %}
**NOTE: The ACA ERC20 token mirrors the balance of the native ACA currency, so you are able to transfer ACA within your smart contract the same way you would transfer a non-native ERC20 token.**
{% endhint %}

The test file with import statements and an empty test should look like this:

```javascript
const { expect } = require("chai");
const { Contract } = require("ethers");
const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");

const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");

describe("PrecompiledToken contract", function () {

});
```

To prepare for the testing, we have to define four global variables, `instance`, `deployer` and `deployerAddress`. The `instance` will store the predeployed Token smart contract instance. The `deployer` will store a `Signer`. Let's assign them values in the `beforeEach` action:

```javascript
        let instance;
        let deployer;

        beforeEach(async function () {
                this.timeout(100000);
                [deployer] = await ethers.getSigners();
                instance = new Contract(ACA, TokenContract.abi, deployer);
        });
```

You can see how we used the `ACA` from the `ADDRESS` utility in order to set the address of our predeployed smart contract.

Our test will only contain one section called `Precompiled token` in which we will be checking the following examples:

1. The token should return `Acala` when `name()` is called.
2. The token should return `ACA` when `symbol()` is called.
3. The total supply of the token should be greater than 0.
4. The balance of our development address should be greater than 0.

Before the test examples are called, the timeout is increased to a 100s, just in case there is some network latency when communicating with the public development network:

```javascript
        describe("Precompiled token", function () {
                this.timeout(100000);

                it("should have the correct token name", async function () {
                        expect(await instance.name()).to.equal("Acala");
                });

                it("should have the correct token symbol", async function () {
                        expect(await instance.symbol()).to.equal("ACA");
                });

                it("should have the total supply greater than 0", async function () {
                        expect(await instance.totalSupply()).be.above(0);
                });

                it("should show balance of the deployer address higher than 0", async function () {
                        expect(await instance.balanceOf(await deployer.getAddress())).be.above(0);
                });
        });
```

With that, our test is ready to be run.

<details>

<summary>Your test/PrecompiledToken.js should look like this:</summary>

```javascript
    const { expect } = require("chai");
    const { Contract } = require("ethers");
    const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");

    const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");

    describe("PrecompiledToken contract", function () {
            let instance;
            let deployer;

            beforeEach(async function () {
                    this.timeout(100000);

                    [deployer] = await ethers.getSigners();
                    instance = new Contract(ACA, TokenContract.abi, deployer);
            });

            describe("Precompiled token", function () {
                    this.timeout(100000);

                    it("should have the correct token name", async function () {
                            expect(await instance.name()).to.equal("Acala");
                    });

                    it("should have the correct token symbol", async function () {
                            expect(await instance.symbol()).to.equal("ACA");
                    });

                    it("should have the total supply greater than 0", async function () {
                            expect(await instance.totalSupply()).be.above(0);
                    });

                    it("should show balance of the deployer address higher than 0", async function () {
                            expect(await instance.balanceOf(await deployer.getAddress())).be.above(0);
                    });
            });
    });
```

</details>

{% hint style="info" %}
**NOTE: If you want to interact with other precompiled and predeployed smart contracts, you can take a look at the list of all of the smart contracts supported in the** [**`ADDRESS` utility**](https://github.com/AcalaNetwork/predeploy-contracts/blob/master/contracts/utils/MandalaAddress.js) **and tweak this example test.**
{% endhint %}

When you run the test with (for example) `yarn test-mandala:pubDev`, your tests should pass with the following output:

```shell
yarn test-mandala:pubDev


yarn run v1.22.17
$ hardhat test test/PrecompiledToken.js --network mandalaPubDev


  PrecompiledToken contract
    Precompiled token
      ✓ should have the correct token name (2951ms)
      ✓ should have the correct token symbol (2652ms)
      ✓ should have the total supply greater than 0 (2259ms)
      ✓ should show balance of the deployer address higher than 0 (2035ms)


  4 passing (12s)

Done in 12.89s.
```

### Script

As the smart contract is already deployed to the network, we don't need a deploy script, but we can add a script that interacts with the predeployed smart contract and log some values from it to the console.

Let's name our script `getACAinfo.js` and import `ACA` from the `ADDRESS` utility, `Contract` from `ethers` and `Token` precompile from `@acala-network/contracts`. The script along with the imports should look like this:

```javascript
const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");
const { Contract } = require("ethers");

const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");

async function main() {

}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Our information script will reside in the definition (`async function main()`). First, we will get the address of the account which will be used to interact with the smart contract. Then we assign the predeployed smart contract to the `instance` variable using the `Contract` from `ethers`. Finally we get and output the `name`, `symbol`, `decimals`, `totalSupply` and own balance to the console:

```javascript
  const [deployer] = await ethers.getSigners();

  console.log("Getting contract info with the account:", deployer.address);

  console.log("Account balance:", (await deployer.getBalance()).toString());

  const instance = new Contract(ACA, TokenContract.abi, deployer);

  console.log("PrecompiledToken address:", instance.address);

  const name = await instance.name();
  const symbol = await instance.symbol();
  const decimals = await instance.decimals();
  const value = await instance.totalSupply();
  const balance = await instance.balanceOf(await deployer.getAddress());

  console.log("Token name:", name);
  console.log("Token symbol:", symbol);
  console.log("Token decimal spaces:", decimals.toString());
  console.log("Total supply:", value.toString());
  console.log("Our account token balance:", balance.toString());
```

As we have the information about how many decimal spaces the token uses, we can add a formatting function below the definition of the `main()` function and use it to format the `totalSupply` and own balance. Let's call it `balanceFormatting` and prepare it for formatting in case the balance is higher than 1 and lower than 1:

```javascript
function balanceFormatting(balance, decimals){
  const balanceLength = balance.length;
  let output = "";

  if(balanceLength > decimals){
    for(i = 0; i < (balanceLength - decimals); i++){
      output += balance[i];
    }
    output += ".";
    for(i = (balanceLength - decimals); i < balanceLength; i++){
      output += balance[i];
    }
  } else {
    output += "0."
    for(i = 0; i < (decimals - balanceLength); i++){
      output += "0";
    }
    output += balance;
  }
  return(output);
}
```

Finally we can add these formatted balances to the `main()` function at the bottom of its definition:

```javascript
  const formattedSuppy = balanceFormatting(value.toString(), decimals);
  const formattedBalance = balanceFormatting(balance.toString(), decimals);

  console.log("Total formatted supply: %s %s", formattedSuppy, symbol);
  console.log("Total formatted account token balance: %s %s", formattedBalance, symbol);
```

<details>

<summary>Your script/getACAinfo.js should look like this:</summary>

```javascript
    const { ACA } = require("@acala-network/contracts/utils/MandalaAddress");
    const { Contract } = require("ethers");

    const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");

    async function main() {
            const [deployer] = await ethers.getSigners();

            console.log("Getting contract info with the account:", deployer.address);

            console.log("Account balance:", (await deployer.getBalance()).toString());

            const instance = new Contract(ACA, TokenContract.abi, deployer);

            console.log("PrecompiledToken address:", instance.address);

            const name = await instance.name();
            const symbol = await instance.symbol();
            const decimals = await instance.decimals();
            const value = await instance.totalSupply();
            const balance = await instance.balanceOf(await deployer.getAddress());

            console.log("Token name:", name);
            console.log("Token symbol:", symbol);
            console.log("Token decimal spaces:", decimals.toString());
            console.log("Total supply:", value.toString());
            console.log("Our account token balance:", balance.toString());

            const formattedSuppy = balanceFormatting(value.toString(), decimals);
            const formattedBalance = balanceFormatting(balance.toString(), decimals);

            console.log("Total formatted supply: %s %s", formattedSuppy, symbol);
            console.log("Total formatted account token balance: %s %s", formattedBalance, symbol);
    }

    function balanceFormatting(balance, decimals){
            const balanceLength = balance.length;
            let output = "";

            if(balanceLength > decimals){
                    for(i = 0; i < (balanceLength - decimals); i++){
                            output += balance[i];
                    }
                    output += ".";
                    for(i = (balanceLength - decimals); i < balanceLength; i++){
                            output += balance[i];
                    }
            } else {
                    output += "0."
                    for(i = 0; i < (decimals - balanceLength); i++){
                            output += "0";
                    }
                    output += balance;
            }
            return(output);
    }

    main()
            .then(() => process.exit(0))
            .catch((error) => {
                    console.error(error);
                    process.exit(1);
            });
```

</details>

{% hint style="info" %}
**NOTE: If you wish to modify the script to use another token precompile, the only thing you need to do is replace ACA address constant with another included within the ADDRESS utility (you could use AUSD for example).**
{% endhint %}

To use the script within the local development network or a public development network, you need to add the following scripts to `scripts` section of your `package.json`:

```json5
    "get-info-mandala": "hardhat run scripts/getACAinfo.js --network mandala",
    "get-info-mandala:pubDev": "hardhat run scripts/getACAinfo.js --network mandalaPubDev"
```

Running the `yarn get-info-mandala:pubDev` script should return the following output:

```shell
yarn get-info-mandala:pubDev


yarn run v1.22.17
$ hardhat run scripts/getACAinfo.js --network mandalaPubDev
Getting contract info with the account: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Account balance: 98522154070475281000000
PrecompiledToken address: 0x0000000000000000000100000000000000000000
Token name: Acala
Token symbol: ACA
Token decimal spaces: 12
Total supply: 13320061695086400000
Our account token balance: 98522154070475281
Total formatted supply: 13320061.695086400000 ACA
Total formatted account token balance: 98522.154070475281 ACA
Done in 17.03s.
```

### Summary

We have built upon the knowledge on how to interact with the Acala EVM+ and gotten familiar with Acala EVM+ precompiles and predeploys. To run the test we can use the `yarn test-mandala` and `yarn test-madala:pubDev` and to run the information script we can use the `yarn get-info-mandala` and `yarn get-info-mandala:pubDev`. As we are using utilities only available in the Acala EVM+, we can no longer use a conventional development network like Ganache or Hardhat's emulated network.
