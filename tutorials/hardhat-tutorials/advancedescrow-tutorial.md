---
description: >-
  Tutorial on how to build an Escrow smart contract using Acala EVM+'s built in
  predeployed tokens, DEX and Schedule (an on-chain automation tool).
---

# AdvancedEscrow tutorial

## Table of contents

* [Intro](advancedescrow-tutorial.md#intro)
* [Setting up](advancedescrow-tutorial.md#setting-up)
* [Smart contract](advancedescrow-tutorial.md#smart-contract)
* [Deploy script](advancedescrow-tutorial.md#deploy-script)
* [User journey](advancedescrow-tutorial.md#user-journey)
* [Conclusion](advancedescrow-tutorial.md#conclusion)

## Intro

This tutorial dives into Acala EVM+ smart contract development using Hardhat development framework. We will start with the setup, build the smart contract and write deployment and user journey scripts. The smart contract will allow users to initiate escrows in one currency, and for beneficiaries to specify if they desire to be paid in another currency. Another feature we will familiarise ourselves with is the on-chain automation using a predeployed smart contract called `Schedule`. Using it will allow us to set the automatic completion of escrow after a certain number of blocks are included in the blockchain.

Let’s jump right in!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/advanced-escrow](https://github.com/AcalaNetwork/hardhat-tutorials/tree/master/advanced-escrow)
{% endhint %}

## Setting up

The tutorial project will live in the `advanced-escrow/` folder. We can create it using `mkdir advanced-escrow`. As we will be using Hardhat development framework, we need to initiate the `yarn` project and add `hardhat` as a development dependency:

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

Let’s take a look at the network configurations:&#x20;

* `url`: Used to specify the RPC endpoint of the network
* `accounts`: Section to describe how Hardhat should acquire or derive the EVM accounts
* `mnemonic`: Mnemonic used to derive the accounts. **Add your mnemonic here**
* `path`: Derivation path to create the accounts from the mnemonic
* `chainId`: Specific chain ID of the Mandala chain. The value of `595` is used for both, local development network as well as the public test network
* `timeout`: An override value for the built in transaction response timeout. It is needed only on the public test network

With that, our project is ready for development.

## Smart contract

The `AdvancedEscrow` smart contract, which we will add in the following section, will still leave some areas that could be improved. `Advanced` is referring to the use of the predeployed smart contracts in the Acala EVM+ rather than its function.

When two parties enter into an escrow agreement, using the `AdvancedEscrow` smart contract, the party paying for the service: first transfers the tokens from one of the predeployed ERC20 smart contracts into the escrow smart contract. The party then initiates the escrow within the smart contract. Initiation of escrow requires both the contract address of the token being escrowed, and the wallet address of the beneficiary of escrow.

Upon initiation of the escrow, the smart contract exchanges the tokens coming into escrow for AUSD. Then it sets the deadline after which, AUSD is released to the beneficiary. The beneficiary also has the ability to specify which tokens they want to receive from escrow and the smart contract exchanges the AUSD it is holding in escrow for the desired tokens upon completion of escrow.

We also allow for the escrow to be completed before the deadline, with the ability for the initiating party to release the funds to the beneficiary manually.

Hardhat has already created a smart contract within the `contracts/` folder when we ran it**s** setup. This smart contract is named `Greeter`. We will remove it and add our own called `AdvancedEscrow`:

```shell
rm contracts/Greeter.sol && touch contracts/AdvancedEscrow.sol
```

Now that we have our smart contract file ready, we can place an empty smart contract within it:

```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;
 
contract AdvancedEscrow {
 
}
```

We will be using precompiled smart contracts available in `@acala-network/contracts` and `@openzeppelin/contracts` dependencies. To be able to do this, we need to add the dependencies to the project:

```shell
yarn add --dev @acala-network/contracts@4.2.0 @openzeppelin/contracts@4.4.2
```

As we will be using predeployed IDEX and IScheduler as well as the precompiled Token contracts, we need to import them after the `pragma` statement:

```solidity
import "@acala-network/contracts/dex/IDEX.sol";
import "@acala-network/contracts/token/Token.sol";
import "@acala-network/contracts/schedule/ISchedule.sol";
```

As each of the predeployed smart contracts has a predetermined address, we can use one of the `Address` utilities of `@acala-network/contracts` dependency to set them in our smart contract. There are the `AcalaAddress`, the `KaruraAddress` and the `MandalaAddress` utilities. We can use the `MandalaAddress` in this example:

```solidity
import "@acala-network/contracts/utils/MandalaAddress.sol";
```

Now that we have sorted out all of the imports, we need to make sure that our `AdvancedEscrow` smart contract inherits the `ADDRESS` smart contract utility in order to be able to access the addresses of the predeployed contracts stored within it. We have to add the inheritance statement to the contract definition line:

```solidity
contract AdvancedEscrow is ADDRESS {
```

We can finally start working on the actual smart contract. We will be interacting with the predeployed DEX and Schedule smart contracts, so we can define them at the beginning of the smart contract:

```solidity
   IDEX public dex = IDEX(ADDRESS.DEX);
   ISchedule public schedule = ISchedule(ADDRESS.SCHEDULE);
```

Our smart contract will support one active escrow at the time, but will allow reuse. Let’s add a counter to be able to check the previous escrows, as well as the Escrow structure:

```solidity
   uint256 public numberOfEscrows;
 
   mapping(uint256 => Escrow) public escrows;
 
   struct Escrow {
       address initiator;
       address beneficiary;
       address ingressToken;
       address egressToken;
       uint256 AusdValue;
       uint256 deadline;
       bool completed;
   }
```

As you can see, we added a counter for `numberOfEscrows`, a mapping to list said `escrows` and a struct to keep track of the information included inside an escrow. The `Escrow` structure holds the following information:

* `initiator`: The account that initiated and funded the escrow
* `beneficiary`: The account that is to receive the escrowed funds
* `ingressToken`: Address of the token that was used to fund the escrow
* `egressToken`: Address of the token that will be used to pay out of the escrow
* `AusdValue`: Value of the escrow in AUSD
* `deadline`: Block number of the block after which, the escrow will be paid out
* `completed`: As an escrow can only be active or fulfilled, this can be represented as by a boolean value

The constructor in itself will only be used to set the value of `numberOfEscrows` to 0. While Solidity is a null-state language, it’s still better to be explicit where we can:

```solidity
   constructor() {
       numberOfEscrows = 0;
   }
```

Now we can add the event that will notify listeners of the change in the smart contract called `EscrowUpdate`:

```solidity
   event EscrowUpdate(
       address indexed initiator,
       address indexed beneficiary,
       uint256 AusdValue,
       bool fulfilled
   );
```

The event contains information about the current state of the latest escrow:&#x20;

* `initiator:` Address of the account that initiated the escrow
* `beneficiary`: Address of the account to which the escrow should be released to
* `AusdValue`: Value of the escrow represented in the AUSD currency
* `fulfilled`: As an escrow can only be active or fulfilled, this can be represented as by a boolean value.

Let’s start writing the logic of the escrow. As we said, there should only be one escrow active at any given time and the initiator should transfer the tokens to the smart contract before initiating the escrow. When initiating escrow, the initiator should pass the address of the token they allocated to the smart contract as the function call parameter in order for the smart contract to be able to swap that token for AUSD. All of the escrows are held in AUSD, but they can be paid out in an alternative currency. None of the addresses passed to the function should be `0x0` and the period in which the escrow should automatically be completed, expressed in the number of blocks, should not be 0 as well.

Once all of the checks are passed and the ingress tokens are swapped for AUSD, the completion of escrow should be scheduled with the predeployed `Schedule`. Afterwards, the escrow information should be saved to the storage and `EscrowUpdate` should be emitted.

All of this happens within `initiateEscrow` function:

```solidity
   function initiateEscrow(
       address beneficiary_,
       address ingressToken_,
       uint256 ingressValue,
       uint256 period
   )
       public returns (bool)
   {
       // Check to make sure the latest escrow is completed
       // Additional check is needed to ensure that the first escrow can be initiated and that the
       // guard statement doesn't underflow
       require(
           numberOfEscrows == 0 || escrows[numberOfEscrows - 1].completed,
           "Escrow: current escrow not yet completed"
       );
       require(beneficiary_ != address(0), "Escrow: beneficiary_ is 0x0");
       require(ingressToken_ != address(0), "Escrow: ingressToken_ is 0x0");
       require(period != 0, "Escrow: period is 0");
 
       uint256 contractBalance = Token(ingressToken_).balanceOf(address(this));
       require(
           contractBalance >= ingressValue,
           "Escrow: contract balance is less than ingress value"
       );
 
       Token AUSDtoken = Token(ADDRESS.AUSD);
       uint256 initalAusdBalance = AUSDtoken.balanceOf(address(this));
      
       address[] memory path = new address[](2);
       path[0] = ingressToken_;
       path[1] = ADDRESS.AUSD;
       require(dex.swapWithExactSupply(path, ingressValue, 1), "Escrow: Swap failed");
      
       uint256 finalAusdBalance = AUSDtoken.balanceOf(address(this));
      
       schedule.scheduleCall(
           address(this),
           0,
           1000000,
           5000,
           period,
           abi.encodeWithSignature("completeEscrow()")
       );
 
       Escrow storage currentEscrow = escrows[numberOfEscrows];
       currentEscrow.initiator = msg.sender;
       currentEscrow.beneficiary = beneficiary_;
       currentEscrow.ingressToken = ingressToken_;
       currentEscrow.AusdValue = finalAusdBalance - initalAusdBalance;
       currentEscrow.deadline = block.number + period;
       numberOfEscrows += 1;
      
       emit EscrowUpdate(msg.sender, beneficiary_, currentEscrow.AusdValue, false);
      
       return true;
   }
```

As you might have noticed, we didn’t set the `egressToken` value of the escrow. This is up to the beneficiary. Default payout is AUSD; but the beneficiary should be able to set a different token if they wish. As this is completely their prerogative, they are the only party that can change this value. To be able to do so, we need to add an additional `setEgressToken` function. Only the latest escrow’s egress token value can be modified and only if the latest escrow is still active:

```solidity
   function setEgressToken(address egressToken_) public returns (bool) {
       require(!escrows[numberOfEscrows - 1].completed, "Escrow: already completed");
       require(
           escrows[numberOfEscrows - 1].beneficiary == msg.sender,
           "Escrow: sender is not beneficiary"
       );
 
       escrows[numberOfEscrows - 1].egressToken = egressToken_;
 
       return true;
   }
```

Another thing that you might have noticed is that we scheduled a call of `completeEscrow` in the `scheduleCall` call to the `Schedule` predeployed smart contract. We need to add this function as well. The function should only be able to be run if the current escrow is still active and only by the `AdvancedEscrow` smart contract or by the initiator of the escrow. The smart contract is able to call the `completeEscrow` function, because it passed a pre-signed transaction for this call to the `Schedule` smart contract. The function should swap the AUSD held in escrow for the desired egress token, if one is specified. Otherwise, the AUSD is released to the beneficiary. Once the funds are allocated to the beneficiary, the escrow should be marked as completed and `EscrowUpdate` event, notifying the listeners of the completion, should be emitted:

```solidity
   function completeEscrow() public returns (bool) {
       Escrow storage currentEscrow = escrows[numberOfEscrows - 1];
       require(!currentEscrow.completed, "Escrow: escrow already completed");
       require(
           msg.sender == currentEscrow.initiator || msg.sender == address(this),
           "Escrow: caller is not initiator or this contract"
       );
 
       if(currentEscrow.egressToken != address(0)){
           Token token = Token(currentEscrow.egressToken);
           uint256 initialBalance = token.balanceOf(address(this));
          
           address[] memory path = new address[](2);
           path[0] = ADDRESS.AUSD;
           path[1] = currentEscrow.egressToken;
           require(
               dex.swapWithExactSupply(path, currentEscrow.AusdValue, 1),
               "Escrow: Swap failed"
           );
          
           uint256 finalBalance = token.balanceOf(address(this));
 
           token.transfer(currentEscrow.beneficiary, finalBalance - initialBalance);
       } else {
           Token AusdToken = Token(ADDRESS.AUSD);
           AusdToken.transfer(currentEscrow.beneficiary, currentEscrow.AusdValue);
       }
 
       currentEscrow.completed = true;
 
       emit EscrowUpdate(
           currentEscrow.initiator,
           currentEscrow.beneficiary,
           currentEscrow.AusdValue,
           true
       );
 
       return true;
   }
```

This wraps up our `AdvancedEscrow` smart contract.

<details>

<summary>Your <code>contracts/AdvancedEscrow.sol</code> should look like this:</summary>

```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "@acala-network/contracts/dex/IDEX.sol";
import "@acala-network/contracts/token/Token.sol";
import "@acala-network/contracts/schedule/ISchedule.sol";
 
import "@acala-network/contracts/utils/MandalaAddress.sol";

contract AdvancedEscrow is ADDRESS {
    IDEX dex = IDEX(ADDRESS.DEX);
    ISchedule schedule = ISchedule(ADDRESS.SCHEDULE);

    uint256 public numberOfEscrows;

    mapping(uint256 => Escrow) public escrows;

    struct Escrow {
        address initiator;
        address beneficiary;
        address ingressToken;
        address egressToken;
        uint256 AusdValue;
        uint256 deadline;
        bool completed;
    }

    constructor() {
        numberOfEscrows = 0;
    }

    event EscrowUpdate(
        address indexed initiator,
        address indexed beneficiary,
        uint256 AusdValue,
        bool fulfilled
    );

    function initiateEscrow(
        address beneficiary_,
        address ingressToken_,
        uint256 ingressValue,
        uint256 period
    )
        public returns (bool)
    {
        // Check to make sure the latest escrow is completed
        // Additional check is needed to ensure that the first escrow can be initiated and that the
        // guard statement doesn't underflow
        require(
            numberOfEscrows == 0 || escrows[numberOfEscrows - 1].completed,
            "Escrow: current escrow not yet completed"
        );
        require(beneficiary_ != address(0), "Escrow: beneficiary_ is 0x0");
        require(ingressToken_ != address(0), "Escrow: ingressToken_ is 0x0");
        require(period != 0, "Escrow: period is 0");

        uint256 contractBalance = Token(ingressToken_).balanceOf(address(this));
        require(
            contractBalance >= ingressValue,
            "Escrow: contract balance is less than ingress value"
        );

        Token AUSDtoken = Token(ADDRESS.AUSD);
        uint256 initalAusdBalance = AUSDtoken.balanceOf(address(this));
        
        address[] memory path = new address[](2);
        path[0] = ingressToken_;
        path[1] = ADDRESS.AUSD;
        require(dex.swapWithExactSupply(path, ingressValue, 1), "Escrow: Swap failed");
        
        uint256 finalAusdBalance = AUSDtoken.balanceOf(address(this));
        
        schedule.scheduleCall(
            address(this),
            0,
            1000000,
            5000,
            period,
            abi.encodeWithSignature("completeEscrow()")
        );

        Escrow storage currentEscrow = escrows[numberOfEscrows];
        currentEscrow.initiator = msg.sender;
        currentEscrow.beneficiary = beneficiary_;
        currentEscrow.ingressToken = ingressToken_;
        currentEscrow.AusdValue = finalAusdBalance - initalAusdBalance;
        currentEscrow.deadline = block.number + period;
        numberOfEscrows += 1;
        
        emit EscrowUpdate(msg.sender, beneficiary_, currentEscrow.AusdValue, false);
        
        return true;
    }

    function setEgressToken(address egressToken_) public returns (bool) {
        require(!escrows[numberOfEscrows - 1].completed, "Escrow: already completed");
        require(
            escrows[numberOfEscrows - 1].beneficiary == msg.sender,
            "Escrow: sender is not beneficiary"
        );

        escrows[numberOfEscrows - 1].egressToken = egressToken_;

        return true;
    }

    function completeEscrow() public returns (bool) {
        Escrow storage currentEscrow = escrows[numberOfEscrows - 1];
        require(!currentEscrow.completed, "Escrow: escrow already completed");
        require(
            msg.sender == currentEscrow.initiator || msg.sender == address(this),
            "Escrow: caller is not initiator or this contract"
        );

        if(currentEscrow.egressToken != address(0)){
            Token token = Token(currentEscrow.egressToken);
            uint256 initialBalance = token.balanceOf(address(this));
            
            address[] memory path = new address[](2);
            path[0] = ADDRESS.AUSD;
            path[1] = currentEscrow.egressToken;
            require(
                dex.swapWithExactSupply(path, currentEscrow.AusdValue, 1),
                "Escrow: Swap failed"
            );
            
            uint256 finalBalance = token.balanceOf(address(this));

            token.transfer(currentEscrow.beneficiary, finalBalance - initialBalance);
        } else {
            Token AusdToken = Token(ADDRESS.AUSD);
            AusdToken.transfer(currentEscrow.beneficiary, currentEscrow.AusdValue);
        }

        currentEscrow.completed = true;

        emit EscrowUpdate(
            currentEscrow.initiator,
            currentEscrow.beneficiary,
            currentEscrow.AusdValue,
            true
        );

        return true;
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

### Transaction helper utility

In order to be able to deploy your smart contract to the Acala EVM+ using Hardhat, you need to pass custom transaction parameters to the deploy transactions. We could add them directly to the script, but this becomes cumbersome and repetitive as our project grows. To avoid the repetitiveness, we will create a custom transaction helper utility, which will use `calcEthereumTransactionParams` from `@acala-network/eth-providers` dependency.

First we need to add the dependency to the project:

```shell
yarn add --dev @acala-network/eth-providers
```

Now that we have the required dependency added to the project, we can create the utility:

```shell
mkdir utils && touch utils/transactionHelpers.js
```

The `calcEthereumTransactionParams` is imported at the top of the file and let's define the `txParams()` below it:

```javascript
const { calcEthereumTransactionParams } = require("@acala-network/eth-providers");
async function txParams() {

}
```

Within the `txParams()` function, we set the parameters needed to be passed to the `calcEthereumTransactionParams` and then assign its return values to the `ethParams`. At the end of the function we return the gas price and gas limit needed to deploy a smart contract:

```javascript
    const txFeePerGas = '199999946752';
    const storageByteDeposit = '100000000000000';
    const blockNumber = await ethers.provider.getBlockNumber();

    const ethParams = calcEthereumTransactionParams({
      gasLimit: '31000000',
      validUntil: (blockNumber + 100).toString(),
      storageLimit: '64001',
      txFeePerGas,
      storageByteDeposit
    });

    return {
        txGasPrice: ethParams.txGasPrice,
        txGasLimit: ethParams.txGasLimit
    };
```

In order to be able to use the `txParams` from our new utility, we have to export it at the bottom of the utility:

```javascript
module.exports = { txParams };
```

This concludes the `transactionHelper` and we can move on to writing the deploy script where we will use it.

<details>

<summary>Your utils/transactionHelper.js should look like this:</summary>

```javascript
const { txParams } = require("../utils/transactionHelper");

async function main() {
  const ethParams = await txParams();

  const [deployer] = await ethers.getSigners();

  console.log("Deploying contract with the account:", deployer.address);

  console.log("Account balance:", (await deployer.getBalance()).toString());

  const Token = await ethers.getContractFactory("Token");
  const instance = await Token.deploy(
    1234567890,
    {
      gasPrice: ethParams.txGasPrice,
      gasLimit: ethParams.txGasLimit,
    }
  );

  console.log("Token address:", instance.address);

  const value = await instance.totalSupply();

  console.log("Total supply:", value.toNumber());
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

</details>

### Script

Now that we have our smart contract ready, we can deploy it, so we can use it.

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

Now that we have the skeleton deploy script, we can import the `txParams` from the `transactionHelper` we added in the subsection above at the top of the file:

```javascript
const { txParams } = require("../utils/transactionHelper");
```

At the beginning of the `main` function definition, we will set the transaction parameters, by invoking the `txParams`:

```javascript
const ethParams = await txParams();
```

Now that we have the deploy transaction parameters set, we can deploy the smart contract. We need to get the signer which will be used to deploy the smart contract, then we instantiate the smart contract within the contract factory and deploy it, passing the transaction parameters to the deploy transaction. Once the smart contract is successfully deployed, we will log its address to the console:

```javascript
 const [deployer] = await ethers.getSigners();
 
 const AdvancedEscrow = await ethers.getContractFactory("AdvancedEscrow");
 const instance = await AdvancedEscrow.deploy({
   gasPrice: ethParams.txGasPrice,
   gasLimit: ethParams.txGasLimit,
 });
 
 console.log("AdvancedEscrow address:", instance.address);
```

With that, our deploy script is ready to be run.

<details>

<summary>Your <code>scripts/deploy.js</code> should look like this:</summary>

```javascript
const ethParams = await txParams();

async function main() {
    const ethParams = await txParams();
    
    const [deployer] = await ethers.getSigners();
    
    const AdvancedEscrow = await ethers.getContractFactory("AdvancedEscrow");
    const instance = await AdvancedEscrow.deploy({
        gasPrice: ethParams.txGasPrice,
        gasLimit: ethParams.txGasLimit,
    });
    
    console.log("AdvancedEscrow address:", instance.address);
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
    console.error(error);
    process.exit(1);
    });
```

</details>

In order to be able to run the `deploy.js` script, we need to add a script to the `package. Json`. To add our custom script to the `package.json`, we need to place our custom script into the `"scripts”` section. Let’s add two scripts, one for the local development network and one for the public test network:

```json
   "deploy-mandala": "hardhat run scripts/deploy.js --network mandala",
   "deploy-mandala:pubDev": "hardhat run scripts/deploy.js --network mandalaPubDev"
```

With that, we are able to run the deploy script using `yarn deploy-mandala` or `yarn deploy-mandala:pubDev`. Using the latter command should result in the following output:

```shell
yarn deploy-mandala:pubDev
yarn run v1.22.17
$ hardhat run scripts/deploy.js --network mandalaPubDev
AdvancedEscrow address: 0xcfd3fef59055a7525607694FBcA16B6D92D97Eee
✨  Done in 33.18s.
```

{% hint style="info" %}
NOTE: You might encounter some warnings before the deploy script is executed. This is nothing to be alarmed about, just some redundant or outdated dependencies of the dependencies that we imported.
{% endhint %}

## Test

Initiating Hardhat also created a `test` folder and within it a sample test. We will remove it and add our own test instead:

```shell
rm test/sample-test.js && touch test/AdvancedEscrow.js
```

At the beginning of the file, we will be importing all of the constants and methods that we will require to successfully run the tests. The predeploy smart contract addresses are imported from `@acala-network/contracts` `ADDRESS` utility. We will be using `chai`'s `expect` and `ethers`' `Contract`, `ContractFactory` and `BigNumber`. We also require the compiled artifacts of the smart contracts that we will be using within the test. This is why we assign them to the `AdvancedEscrowContract`, `TokenContract` and `DEXContract` constants. In order to be able to test the block based deadlines, we need the ability to force the block generation within tests. While we could use the `loop` helper utility, we need to have reliable tests that don't fail if `loop` is not running. For this reason, we import the `ApiPromise` and `WsProvider` from `@polkadot/api`. As initiating the `ApiPromise` generates a lot of output, our test output would get very messy if we didn't silence it. To do this we use the `console.mute` dependency, that we have to add to the project by using:

```shell
yarn add --dev console.mute
```

To be able to easily validate things dependent on `0x0` address, we assign it to the `NULL_ADDRESS` constant. Lastly we configure the `ENDPOINT_URL` constant to be used by the `provider`. And instantiate the `WsProvider` to the `provider` constant. All of the imports and the empty test look like this:

```javascript
const { ACA, AUSD, DEX, DOT } = require("@acala-network/contracts/utils/MandalaAddress");
const { expect } = require("chai");
const { Contract, ContractFactory, BigNumber } = require("ethers");

const AdvancedEscrowContract = require("../artifacts/contracts/AdvancedEscrow.sol/AdvancedEscrow.json");
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");
const DEXContract = require("@acala-network/contracts/build/contracts/DEX.json")
const { ApiPromise, WsProvider } = require("@polkadot/api");

require('console.mute');

const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";
const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
const provider = new WsProvider(ENDPOINT_URL);

describe("AdvancedEscrow contract", function () {

});
```

To setup for each of the test examples we define the `AdvancedEscrow` variable that will hold the contract factory for our smart contract. The `instance` variable will hold the instance of the smart contract that we will be testing against and the `ACAinstance`, `AUSDinstance`, `DOTinstance` and `DEXinstance` hold the instances of the predeployed smart contracts. The `deployer` and `user` hold the Signers and `deployerAddress` and `userAddress` variables hold the addresses of these signers. Finally the `api` variable holds the `ApiPromise`, which we will use to force the generation of blocks. As creation of `ApiPromise` generates a lot of console output, especially when being run before each of the test examples. This is why we have to mute the console output before we create it and resume it after, to keep the expected behaviour of the console. All of the values are assigned in the `beforeEach` action:

```javascript
  let AdvancedEscrow;
  let instance;
  let ACAinstance;
  let AUSDinstance;
  let DOTinstance;
  let DEXinstance;
  let deployer;
  let user;
  let deployerAddress;
  let userAddress;
  let api;

  beforeEach(async function () {
    [deployer, user] = await ethers.getSigners();
    deployerAddress = await deployer.getAddress();
    userAddress = await user.getAddress();
    AdvancedEscrow = new ContractFactory(AdvancedEscrowContract.abi, AdvancedEscrowContract.bytecode, deployer);
    instance = await AdvancedEscrow.deploy();
    await instance.deployed();
    ACAinstance = new Contract(ACA, TokenContract.abi, deployer);
    AUSDinstance = new Contract(AUSD, TokenContract.abi, deployer);
    DOTinstance = new Contract(DOT, TokenContract.abi, deployer);
    DEXinstance = new Contract(DEX, DEXContract.abi, deployer);
    console.mute();
    api = await ApiPromise.create({ provider });
    console.resume();
  });
```

Our test cases will be split into two groups. One will be called `Deployment` and it will verify that the deployed smart contract has expected values set before it is being used. The second one will be called `Operation` and it will validate the expected behaviour of our smart contract. The empty sections should look like this:

```javascript
  describe("Deployment", function () {

  });

  describe("Operation", function () {

  });
```

We will only have one example within the `Deployment` section and it will verify that the number of escrows in a newly deployed smart contract is set to `0`:

```javascript
    it("should set the initial number of escrows to 0", async function () {
      expect(await instance.numberOfEscrows()).to.equal(0);
    });
```

The `Operation` section will hold more test examples. We will be checking for the following cases:

1. Initiating an escrow with a beneficiary of `0x0` should revert.
2. Initiating an escrow with a token address of `0x0` should revert.
3. Initiating an escrow with a duration of `0` blocks should revert.
4. Initiating an escrow with the value of escrow being higher than the balance of the smart contract should revert.
5. Upon successfully initiating an escrow, `EscrowUpdate` should be emitted.
6. Upon successfully initating an escrow, the values defining it should correspont to those passed upon initiation.
7. Initiating an escrow before the current escrow is completed should revert.
8. Trying to set the egress token should revert if the escrow has already been completed.
9. Trying to set the egress token should revert when it is not called by the beneficiary.
10. When egress token is successfully set, the escrow value should be updated.
11. Completing an escrow that was already completed should revert.
12. Completing an escrow while not being the initiator should revert.
13. Escrow should be paid out in AUSD when no egress token is set.
14. Escrow should be paid out in the desired egress token when one is set.
15. When escrow is paid out in egress token, that should not impact the AUSD balance of the beneficiary.
16. When escrow is completed, `EscrowUpdate` is emitted.
17. Escrow should be completed automatically when the desired number of blocks has passed.

These are the examples outlined above:

```javascript
    it("should revert when beneficiary is 0x0", async function () {
      await expect(instance.initiateEscrow(NULL_ADDRESS, ACA, 10_000, 10)).to
        .be.revertedWith("Escrow: beneficiary_ is 0x0");
    });

    it("should revert when ingress token is 0x0", async function () {
      await expect(instance.initiateEscrow(userAddress, NULL_ADDRESS, 10_000, 10)).to
        .be.revertedWith("Escrow: ingressToken_ is 0x0");
    });

    it("should revert when period is 0", async function () {
      await expect(instance.initiateEscrow(userAddress, ACA, 10_000, 0)).to
        .be.revertedWith("Escrow: period is 0");
    });

    it("should revert when balance of the contract is lower than ingressValue", async function () {
      const balance = await ACAinstance.balanceOf(instance.address);

      expect(balance).to.be.below(BigNumber.from("10000"));

      await expect(instance.initiateEscrow(userAddress, ACA, 10_000, 10)).to
        .be.revertedWith("Escrow: contract balance is less than ingress value");
    });

    it("should initate escrow and emit EscrowUpdate when initating escrow", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));

      await expect(instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1)).to
        .emit(instance, "EscrowUpdate")
        .withArgs(deployerAddress, userAddress, expectedValue, false);
    });

    it("should set the values of current escrow when initiating the escrow", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

      const blockNumber = await ethers.provider.getBlockNumber();
      const currentId = await instance.numberOfEscrows();
      const escrow = await instance.escrows(currentId.sub(1));

      expect(escrow.initiator).to.equal(deployerAddress);
      expect(escrow.beneficiary).to.equal(userAddress);
      expect(escrow.ingressToken).to.equal(ACA);
      expect(escrow.egressToken).to.equal(NULL_ADDRESS);
      expect(escrow.AusdValue).to.equal(expectedValue);
      expect(escrow.deadline).to.equal(blockNumber + 1);
      expect(escrow.completed).to.be.false;
    });

    it("should revert when initiating a new escrow when there is a preexiting active escrow", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
      
      await expect(instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1)).to
        .be.revertedWith("Escrow: current escrow not yet completed");
    });

    it("should revert when trying to set the egress token after the escrow has already been completed", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

      await expect(instance.connect(user).setEgressToken(DOT)).to
        .be.revertedWith("Escrow: already completed");
    });

    it("should revert when trying to set the egress token while not being the beneficiary", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

      await expect(instance.connect(deployer).setEgressToken(DOT)).to
        .be.revertedWith("Escrow: sender is not beneficiary");
    });

    it("should update the egress token", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);
      const blockNumber = await ethers.provider.getBlockNumber();

      await instance.connect(user).setEgressToken(DOT);

      const currentId = await instance.numberOfEscrows();
      const escrow = await instance.escrows(currentId.sub(1));

      expect(escrow.initiator).to.equal(deployerAddress);
      expect(escrow.beneficiary).to.equal(userAddress);
      expect(escrow.ingressToken).to.equal(ACA);
      expect(escrow.egressToken).to.equal(DOT);
      expect(escrow.AusdValue).to.equal(expectedValue);
      expect(escrow.deadline).to.equal(blockNumber + 10);
      expect(escrow.completed).to.be.false;
    });

    it("should revert when trying to complete an already completed escrow", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

      await expect(instance.connect(deployer).completeEscrow()).to
        .be.revertedWith("Escrow: escrow already completed");
    });

    it("should revert when trying to complete an escrow when not being the initiator", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

      await expect(instance.connect(user).completeEscrow()).to
        .be.revertedWith("Escrow: caller is not initiator or this contract");
    });

    it("should pay out the escrow in AUSD if no egress token is set", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);
      const initalBalance = await AUSDinstance.balanceOf(userAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

      await instance.connect(deployer).completeEscrow();
      const finalBalance = await AUSDinstance.balanceOf(userAddress);

      expect(finalBalance).to.be.above(initalBalance);
    });

    it("should pay out the escrow in set token when egress token is set", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);
      const initalBalance = await DOTinstance.balanceOf(userAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);
      await instance.connect(user).setEgressToken(DOT);

      await instance.connect(deployer).completeEscrow();
      const finalBalance = await DOTinstance.balanceOf(userAddress);

      expect(finalBalance).to.be.above(initalBalance);
    });

    it("should not pay out the escrow in set AUSD when egress token is set", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);
      const initalBalance = await AUSDinstance.balanceOf(userAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);
      await instance.connect(user).setEgressToken(DOT);

      await instance.connect(deployer).completeEscrow();
      const finalBalance = await AUSDinstance.balanceOf(userAddress);

      expect(finalBalance).to.equal(initalBalance);
    });

    it("should emit EscrowUpdate when escrow is completed", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));
      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);

      await expect(instance.connect(deployer).completeEscrow()).to
        .emit(instance, "EscrowUpdate")
        .withArgs(deployerAddress, userAddress, expectedValue, true);
    });

    it("should automatically complete the escrow when given number of blocks has passed", async function () {
      const startingBalance = await ACAinstance.balanceOf(deployerAddress);

      await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

      await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
      const currentEscrow = await instance.numberOfEscrows();
      const initalState = await instance.escrows(currentEscrow.sub(1));
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      const finalState = await instance.escrows(currentEscrow.sub(1));

      expect(initalState.completed).to.be.false;
      expect(finalState.completed).to.be.true;
    });
```

This concludes our test.

<details>

<summary>Your test/AdvancedEscrow.js should look like this:</summary>

```javascript
const { ACA, AUSD, DEX, DOT } = require("@acala-network/contracts/utils/MandalaAddress");
const { expect } = require("chai");
const { Contract, ContractFactory, BigNumber } = require("ethers");

const AdvancedEscrowContract = require("../artifacts/contracts/AdvancedEscrow.sol/AdvancedEscrow.json");
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");
const DEXContract = require("@acala-network/contracts/build/contracts/DEX.json")
const { ApiPromise, WsProvider } = require("@polkadot/api");

require('console.mute');

const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";
const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
const provider = new WsProvider(ENDPOINT_URL);

describe("AdvancedEscrow contract", function () {
    let AdvancedEscrow;
    let instance;
    let ACAinstance;
    let AUSDinstance;
    let DOTinstance;
    let DEXinstance;
    let deployer;
    let user;
    let deployerAddress;
    let userAddress;
    let api;

    beforeEach(async function () {
        [deployer, user] = await ethers.getSigners();
        deployerAddress = await deployer.getAddress();
        userAddress = await user.getAddress();
        AdvancedEscrow = new ContractFactory(AdvancedEscrowContract.abi, AdvancedEscrowContract.bytecode, deployer);
        instance = await AdvancedEscrow.deploy();
        await instance.deployed();
        ACAinstance = new Contract(ACA, TokenContract.abi, deployer);
        AUSDinstance = new Contract(AUSD, TokenContract.abi, deployer);
        DOTinstance = new Contract(DOT, TokenContract.abi, deployer);
        DEXinstance = new Contract(DEX, DEXContract.abi, deployer);
        console.mute();
        api = await ApiPromise.create({ provider });
        console.resume();
    });

    describe("Deployment", function () {
        it("should set the initial number of escrows to 0", async function () {
            expect(await instance.numberOfEscrows()).to.equal(0);
        });
    });

    describe("Operation", function () {
        it("should revert when beneficiary is 0x0", async function () {
            await expect(instance.initiateEscrow(NULL_ADDRESS, ACA, 10_000, 10)).to
                .be.revertedWith("Escrow: beneficiary_ is 0x0");
        });

        it("should revert when ingress token is 0x0", async function () {
            await expect(instance.initiateEscrow(userAddress, NULL_ADDRESS, 10_000, 10)).to
                .be.revertedWith("Escrow: ingressToken_ is 0x0");
        });

        it("should revert when period is 0", async function () {
            await expect(instance.initiateEscrow(userAddress, ACA, 10_000, 0)).to
                .be.revertedWith("Escrow: period is 0");
        });

        it("should revert when balance of the contract is lower than ingressValue", async function () {
            const balance = await ACAinstance.balanceOf(instance.address);

            expect(balance).to.be.below(BigNumber.from("10000"));

            await expect(instance.initiateEscrow(userAddress, ACA, 10_000, 10)).to
                .be.revertedWith("Escrow: contract balance is less than ingress value");
        });

        it("should initate escrow and emit EscrowUpdate when initating escrow", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));

            await expect(instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1)).to
                .emit(instance, "EscrowUpdate")
                .withArgs(deployerAddress, userAddress, expectedValue, false);
        });

        it("should set the values of current escrow when initiating the escrow", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

            const blockNumber = await ethers.provider.getBlockNumber();
            const currentId = await instance.numberOfEscrows();
            const escrow = await instance.escrows(currentId.sub(1));

            expect(escrow.initiator).to.equal(deployerAddress);
            expect(escrow.beneficiary).to.equal(userAddress);
            expect(escrow.ingressToken).to.equal(ACA);
            expect(escrow.egressToken).to.equal(NULL_ADDRESS);
            expect(escrow.AusdValue).to.equal(expectedValue);
            expect(escrow.deadline).to.equal(blockNumber + 1);
            expect(escrow.completed).to.be.false;
        });

        it("should revert when initiating a new escrow when there is a preexiting active escrow", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
            
            await expect(instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1)).to
                .be.revertedWith("Escrow: current escrow not yet completed");
        });

        it("should revert when trying to set the egress token after the escrow has already been completed", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
            await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
            await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

            await expect(instance.connect(user).setEgressToken(DOT)).to
                .be.revertedWith("Escrow: already completed");
        });

        it("should revert when trying to set the egress token while not being the beneficiary", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

            await expect(instance.connect(deployer).setEgressToken(DOT)).to
                .be.revertedWith("Escrow: sender is not beneficiary");
        });

        it("should update the egress token", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);
            const blockNumber = await ethers.provider.getBlockNumber();

            await instance.connect(user).setEgressToken(DOT);

            const currentId = await instance.numberOfEscrows();
            const escrow = await instance.escrows(currentId.sub(1));

            expect(escrow.initiator).to.equal(deployerAddress);
            expect(escrow.beneficiary).to.equal(userAddress);
            expect(escrow.ingressToken).to.equal(ACA);
            expect(escrow.egressToken).to.equal(DOT);
            expect(escrow.AusdValue).to.equal(expectedValue);
            expect(escrow.deadline).to.equal(blockNumber + 10);
            expect(escrow.completed).to.be.false;
        });

        it("should revert when trying to complete an already completed escrow", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
            await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
            await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

            await expect(instance.connect(deployer).completeEscrow()).to
                .be.revertedWith("Escrow: escrow already completed");
        });

        it("should revert when trying to complete an escrow when not being the initiator", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

            await expect(instance.connect(user).completeEscrow()).to
                .be.revertedWith("Escrow: caller is not initiator or this contract");
        });

        it("should pay out the escrow in AUSD if no egress token is set", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);
            const initalBalance = await AUSDinstance.balanceOf(userAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);

            await instance.connect(deployer).completeEscrow();
            const finalBalance = await AUSDinstance.balanceOf(userAddress);

            expect(finalBalance).to.be.above(initalBalance);
        });

        it("should pay out the escrow in set token when egress token is set", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);
            const initalBalance = await DOTinstance.balanceOf(userAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);
            await instance.connect(user).setEgressToken(DOT);

            await instance.connect(deployer).completeEscrow();
            const finalBalance = await DOTinstance.balanceOf(userAddress);

            expect(finalBalance).to.be.above(initalBalance);
        });

        it("should not pay out the escrow in set AUSD when egress token is set", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);
            const initalBalance = await AUSDinstance.balanceOf(userAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);
            await instance.connect(user).setEgressToken(DOT);

            await instance.connect(deployer).completeEscrow();
            const finalBalance = await AUSDinstance.balanceOf(userAddress);

            expect(finalBalance).to.equal(initalBalance);
        });

        it("should emit EscrowUpdate when escrow is completed", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], startingBalance.div(1_000_000));
            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 10);

            await expect(instance.connect(deployer).completeEscrow()).to
                .emit(instance, "EscrowUpdate")
                .withArgs(deployerAddress, userAddress, expectedValue, true);
        });

        it("should automatically complete the escrow when given number of blocks has passed", async function () {
            const startingBalance = await ACAinstance.balanceOf(deployerAddress);

            await ACAinstance.connect(deployer).transfer(instance.address, startingBalance.div(100_000));

            await instance.connect(deployer).initiateEscrow(userAddress, ACA, startingBalance.div(1_000_000), 1);
            const currentEscrow = await instance.numberOfEscrows();
            const initalState = await instance.escrows(currentEscrow.sub(1));
            await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
            await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
            const finalState = await instance.escrows(currentEscrow.sub(1));

            expect(initalState.completed).to.be.false;
            expect(finalState.completed).to.be.true;
        });
    });
});
```

</details>

As our test is ready to be run, we have to add the scripts to be able to run the test. We will be adding two scripts. One to run the tests on the local development network and on the public test network:

```json
    "test-mandala": "hardhat test test/AdvancedEscrow.js --network mandala",
    "test-mandala:pubDev": "hardhat test test/AdvancedEscrow.js --network mandalaPubDev"
```

Running the tests with `yarn test-mandala` should give you the following output:

```shell
yarn test-mandala

yarn run v1.22.19
$ hardhat test test/AdvancedEscrow.js --network mandala

  AdvancedEscrow contract
    Deployment
      ✔ should set the initial number of escrows to 0
    Operation
      ✔ should revert when beneficiary is 0x0
      ✔ should revert when ingress token is 0x0
      ✔ should revert when period is 0
      ✔ should revert when balance of the contract is lower than ingressValue (49ms)
      ✔ should initate escrow and emit EscrowUpdate when initating escrow (6650ms)
      ✔ should set the values of current escrow when initiating the escrow (5670ms)
      ✔ should revert when initiating a new escrow when there is a preexiting active escrow (6686ms)
      ✔ should revert when trying to set the egress token after the escrow has already been completed (6741ms)
      ✔ should revert when trying to set the egress token while not being the beneficiary (6678ms)
      ✔ should update the egress token (8910ms)
      ✔ should revert when trying to complete an already completed escrow (6752ms)
      ✔ should revert when trying to complete an escrow when not being the initiator (6676ms)
      ✔ should pay out the escrow in AUSD if no egress token is set (8906ms)
      ✔ should pay out the escrow in set token when egress token is set (12301ms)
      ✔ should not pay out the escrow in set AUSD when egress token is set (12176ms)
      ✔ should emit EscrowUpdate when escrow is completed (9930ms)
      ✔ should automatically complete the escrow when given number of blocks has passed (5675ms)


  18 passing (3m)

✨  Done in 170.22s.
```

## User journey

Finally we are able to simulate the user journey through the `AdvancedEscrow`. We will create another script in the `scripts` directory called `userJourney.js` and add three scenarios to it:

Beneficiary accepts the funds in aUSD and the escrow is released by `Schedule` predeployed smart contract. Beneficiary accepts the funds in aUSD and the escrow is released by the initiator of the escrow, before it is released by the `Schedule`. Beneficiary decides to get paid in DOT and the escrow is released by the `Schedule`.

To create the `userJourney.` file, we can use the following command:

```shell
touch scripts/userJourney.js
```

Let’s add the imports and constants to the file:

```javascript
const { txParams } = require("../utils/transactionHelper");
const { ACA, AUSD, DOT } = require("@acala-network/contracts/utils/MandalaAddress");
const { Contract } = require("ethers");
const { formatUnits } = require("ethers/lib/utils");
 
const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");
 
const txFeePerGas = '199999946752';
const storageByteDeposit = '100000000000000';
 
async function main() {
 
}
 
main()
 .then(() => process.exit(0))
 .catch((error) => {
   console.error(error);
   process.exit(1);
 });
```

We have to import the `txParams` and invoke it in order to be able to define the deployment transaction gas parameters within the `main` function. The script will use `ACA`, `AUSD` and `DOT` predeployed token smart contracts, so we need to import their addresses from the `Address` utility and `Token` precompile from `@acala-network/contracts` in order to be able to instantiate them. For the same reason as we are importing the `Token` precompile, we are also importing the `Contract` from `ethers` as it is required to instantiate the already deployed smart contract. `formatUnits` utility is imported, so that we will be able to print the formatted balances to the console.

Much like in the `deploy.js` we still need to prepare the deployment transaction gas parameters at the beginning of the `main` function:

```javascript
 const ethParams = await txParams();
```

It’s time to add a setup that includes getting the required signers, deploying the `AdvancedEscrow` smart contract and instantiating ACA ERC20 predeployed contract. We will also output the formatted balance of the ACA token to the console:

```javascript
 const [initiator, beneficiary] = await ethers.getSigners();
 
 const initiatorAddress = await initiator.getAddress();
 const beneficiaryAddress = await beneficiary.getAddress();
 
 console.log("Address of the initiator is", initiatorAddress);
 console.log("Address of the beneficiary is", beneficiaryAddress);
 
 console.log("Deploying AdvancedEscrow smart contract")
 const AdvancedEscrow = await ethers.getContractFactory("AdvancedEscrow");
 const instance = await AdvancedEscrow.deploy({
   gasPrice: ethParams.txGasPrice,
   gasLimit: ethParams.txGasLimit,
 });
 
 console.log("AdvancedEscrow is deployed at address:", instance.address);
 
 console.log("Instantiating ACA predeployed smart contract");
 const primaryTokenInstance = new Contract(ACA, TokenContract.abi, initiator);
 
 const intialPrimaryTokenBalance = await primaryTokenInstance.balanceOf(initiatorAddress);
 const primaryTokenName = await primaryTokenInstance.name();
 const primaryTokenSymbol = await primaryTokenInstance.symbol();
 const primaryTokenDecimals = await primaryTokenInstance.decimals();
 console.log("Initial initiator %s token balance: %s %s", primaryTokenName, formatUnits(intialPrimaryTokenBalance.toString(), primaryTokenDecimals), primaryTokenSymbol);
```

{% hint style="info" %}
**NOTE: We are assigning signer’s addresses to the variables as we will use them a few times and doing so alleviates some of the repetition.**
{% endhint %}

To make the output to the console easier to read, we will add a few empty lines to the script. In the first scenario we will transfer the ACA token to the `AdvancedEscrow` smart contract and initiate the escrow. Then we will get the block number at which the escrow was initiated and output the block number of block in which the `Schedule` should automatically release the funds:

```javascript
 console.log("");
 console.log("");
 
 console.log("Scenario #1: Escrow funds are released by Schedule");
 
 console.log("");
 console.log("");
 
 console.log("Transferring primary token to Escrow instance");
 
 await primaryTokenInstance.connect(initiator).transfer(instance.address, intialPrimaryTokenBalance.div(10_000));
 
 console.log("Initiating escrow");
 
 await instance.connect(initiator).initiateEscrow(beneficiaryAddress, ACA, intialPrimaryTokenBalance.div(100_000), 10);
 
 const escrowBlockNumber = await ethers.provider.getBlockNumber();
 
 console.log("Escrow initiation successful in block %s. Expected automatic completion in block %s", escrowBlockNumber, escrowBlockNumber + 10);
```

{% hint style="warning" %}
**WARNING: As you might have noticed, we initiated the escrow using a tenth of the funds that we transferred to the smart contract. This is because the smart contract needs to have some free balance in order to be able to pay for the scheduled call.**

**If you have transferred less than 2 ACA into the smart contract, 1/10 of this value might not be enough for the \`Schedule\` to complete the escrow. In such case it might be prudent to refactor the script in a way that the smart contract keeps at least 1.5 ACA out of escrow.**
{% endhint %}

Since we made the `escrows` public, we can use the automatically generated getter, to get the information about the escrow we have just created and output it to the console:

```javascript
 const escrow = await instance.escrows(0);
 
 console.log("Escrow initiator:", escrow.initiator);
 console.log("Escrow beneficiary:", escrow.beneficiary);
 console.log("Escrow ingress token:", escrow.ingressToken);
 console.log("Escrow egress token:", escrow.egressToken);
 console.log("Escrow AUSD value:", escrow.AusdValue.toString());
 console.log("Escrow deadline:", escrow.deadline.toString());
 console.log("Escrow completed:", escrow.completed);
```

To make sure the escrow funds release actually increases the beneficiary’s funds, we need to instantiate the AUSD smart contract and get the initial balance of the beneficiary:

```javascript
 console.log("Instantiating AUSD instance");
 const AusdInstance = new Contract(AUSD, TokenContract.abi, initiator);
 
 const initalBeneficiatyAusdBalance = await AusdInstance.balanceOf(beneficiaryAddress);
 
 console.log("Initial aUSD balance of beneficiary: %s AUSD", formatUnits(initalBeneficiatyAusdBalance.toString(), 12));
```

{% hint style="warning" %}
**WARNING: The predeployed ERC20 smart contracts always use 12 decimal spaces, which means, we have to pass the decimals argument to the `formatUnits` and we can not use the default value of 18.**
{% endhint %}

As we have to wait for the `Schedule` to release the escrowed funds, we need to add a while loop to check whether the block in which the `Schedule` will release the funds has already been added to the blockchain. We don’t want to overwhelm the console output, so we have to add a helper function, which we define above the `main` function definition, that will pause the execution of the loop for `x` number of milliseconds:

```javascript
const sleep = async time => new Promise((resolve) => setTimeout(resolve, time));
```

We can finally add the loop that waits for the deadline block to be added to the chain:

```javascript
 let currentBlockNumber = await ethers.provider.getBlockNumber();
 
 while(currentBlockNumber <= escrowBlockNumber + 10){
   console.log("Still waiting. Current block number is %s. Target block number is %s.", currentBlockNumber, escrowBlockNumber + 10);
   currentBlockNumber = await ethers.provider.getBlockNumber();
   await sleep(2500);
 }
```

All that is left to do in the first scenario is to get the final balance of the beneficiary and output it to the console along with the amount of funds, for which their balance has increased:

```javascript
 const finalBeneficiaryAusdBalance = await AusdInstance.balanceOf(beneficiaryAddress);
 
 console.log("Final aUSD balance of beneficiary: %s AUSD", formatUnits(finalBeneficiaryAusdBalance.toString(), 12));
 console.log("Beneficiary aUSD balance has increased for %s AUSD", formatUnits(finalBeneficiaryAusdBalance.sub(initalBeneficiatyAusdBalance).toString(), 12));
```

In the second scenario, we will have the initiator releasing the funds before the `Schedule` has a chance to do so. We will add some more blank lines, so that the console output wil be clearer, initiate a new escrow and output it’s details:

```javascript
 console.log("");
 console.log("");
  console.log("Scenario #2: Escrow initiator releases the funds before the deadline");
 
 console.log("");
 console.log("");
 
 console.log("Initiating escrow");
 
 await instance.connect(initiator).initiateEscrow(beneficiaryAddress, ACA, intialPrimaryTokenBalance.div(100_000), 10);
 
 const escrowBlockNumber2 = await ethers.provider.getBlockNumber();
 
 console.log("Escrow initiation successful in block %s. Expected automatic completion in block %s", escrowBlockNumber2, escrowBlockNumber2 + 10);
 
 const escrow2 = await instance.escrows(1);
 
 console.log("Escrow initiator:", escrow2.initiator);
 console.log("Escrow beneficiary:", escrow2.beneficiary);
 console.log("Escrow ingress token:", escrow2.ingressToken);
 console.log("Escrow egress token:", escrow2.egressToken);
 console.log("Escrow AUSD value:", escrow2.AusdValue.toString());
 console.log("Escrow deadline:", escrow2.deadline.toString());
 console.log("Escrow completed:", escrow2.completed);
```

All that is left to do in this example is to get the balance of the beneficiary before and after the release of funds from the escrow, manually releasing the funds and logging the results to the console.

```javascript
 const initalBeneficiatyAusdBalance2 = await AusdInstance.balanceOf(beneficiaryAddress);
 
 console.log("Initial aUSD balance of beneficiary: %s AUSD", formatUnits(initalBeneficiatyAusdBalance2.toString(), 12));
 
 console.log("Manually releasing the funds");
 
 await instance.connect(initiator).completeEscrow();
 
 let currentBlockNumber2 = await ethers.provider.getBlockNumber();
 
 const finalBeneficiaryAusdBalance2 = await AusdInstance.balanceOf(beneficiaryAddress);
 
 console.log("Escrow funds released at block %s, while the deadline was %s", currentBlockNumber2, escrow2.deadline);
 console.log("Final aUSD balance of beneficiary: %s AUSD", formatUnits(finalBeneficiaryAusdBalance2.toString(), 12));
 console.log("Beneficiary aUSD balance has increased for %s AUSD", formatUnits(finalBeneficiaryAusdBalance2.sub(initalBeneficiatyAusdBalance2).toString(), 12));
```

{% hint style="info" %}
**NOTE: We didn’t have to instantiate AUSD smart contract here, because we already instantiated it in the first scenario.**
{% endhint %}

In the last scenario we will let the `Schedule` release the funds, but the beneficiary will set the egress token to DOT. The beginning of this example is similar to the ones before:

```javascript
 console.log("");
 console.log("");
 
 console.log("Scenario #3: Beneficiary decided to be paid out in DOT");
 
 console.log("");
 console.log("");
 
 console.log("Initiating escrow");
 
 await instance.connect(initiator).initiateEscrow(beneficiaryAddress, ACA, intialPrimaryTokenBalance.div(100_000), 10);
 
 const escrowBlockNumber3 = await ethers.provider.getBlockNumber();
 
 console.log("Escrow initiation successful in block %s. Expected automatic completion in block %s", escrowBlockNumber3, escrowBlockNumber3 + 10);
 
 const escrow3 = await instance.escrows(2);
 
 console.log("Escrow initiator:", escrow3.initiator);
 console.log("Escrow beneficiary:", escrow3.beneficiary);
 console.log("Escrow ingress token:", escrow3.ingressToken);
 console.log("Escrow egress token:", escrow3.egressToken);
 console.log("Escrow AUSD value:", escrow3.AusdValue.toString());
 console.log("Escrow deadline:", escrow3.deadline.toString());
 console.log("Escrow completed:", escrow3.completed);
```

As the escrow is set up, beneficiary can now configure the egress token of the escrow:

```javascript
 console.log("Beneficiary setting the desired escrow egress token");
 
 await instance.connect(beneficiary).setEgressToken(DOT);
```

If we want to output the beneficiary’s DOT balance and the difference in balance after the funds are released from escrow, we need to instantiate the DOT predeployed smart contract. Now we can also output the initial DOT balance of the beneficiary:

```javascript
 console.log("Instantiating DOT instance");
 const DotInstance = new Contract(DOT, TokenContract.abi, initiator);
 
 const initalBeneficiatyDotBalance = await DotInstance.balanceOf(beneficiaryAddress);
 
 console.log("Initial DOT balance of beneficiary: %s DOT", formatUnits(initalBeneficiatyDotBalance.toString(), 12));
```

All that is left to do is to wait for the `Schedule` to release the funds and log the changes and results to the console:

```javascript
 console.log("Waiting for automatic release of funds");
 
 let currentBlockNumber3 = await ethers.provider.getBlockNumber();
 
 while(currentBlockNumber3 <= escrowBlockNumber3 + 10){
   console.log("Still waiting. Current block number is %s. Target block number is %s.", currentBlockNumber3, escrowBlockNumber3 + 10);
   currentBlockNumber3 = await ethers.provider.getBlockNumber();
   await sleep(2500);
 }
 
 const finalBeneficiaryDotBalance = await DotInstance.balanceOf(beneficiaryAddress);
 
 console.log("Final DOT balance of beneficiary: %s DOT", formatUnits(finalBeneficiaryDotBalance.toString(), 12));
 console.log("Beneficiary DOT balance has increased for %s DOT", formatUnits(finalBeneficiaryDotBalance.sub(initalBeneficiatyDotBalance).toString(), 12));
```

With that, our user journey script is completed.

<details>

<summary>Your <code>scripts/userJourney.js</code> should look like this:</summary>

```javascript
const { txParams } = require("../utils/transactionHelper");
const { ACA, AUSD, DOT } = require("@acala-network/contracts/utils/MandalaAddress");
const { Contract } = require("ethers");
const { formatUnits } = require("ethers/lib/utils");

const TokenContract = require("@acala-network/contracts/build/contracts/Token.json");

const sleep = async time => new Promise((resolve) => setTimeout(resolve, time));

async function main() {
    const ethParams = await txParams();
    
    console.log("Getting signers");
    const [initiator, beneficiary] = await ethers.getSigners();
    
    const initiatorAddress = await initiator.getAddress();
    const beneficiaryAddress = await beneficiary.getAddress();
    
    console.log("Address of the initiator is", initiatorAddress);
    console.log("Address of the beneficiary is", beneficiaryAddress);
    
    console.log("Deploying AdvancedEscrow smart contract")
    const AdvancedEscrow = await ethers.getContractFactory("AdvancedEscrow");
    const instance = await AdvancedEscrow.deploy({
    gasPrice: ethParams.txGasPrice,
    gasLimit: ethParams.txGasLimit,
    });
    
    console.log("AdvancedEscrow is deployed at address:", instance.address);
    
    console.log("Instantiating ACA predeployed smart contract");
    const primaryTokenInstance = new Contract(ACA, TokenContract.abi, initiator);
    
    const intialPrimaryTokenBalance = await primaryTokenInstance.balanceOf(initiatorAddress);
    const primaryTokenName = await primaryTokenInstance.name();
    const primaryTokenSymbol = await primaryTokenInstance.symbol();
    const primaryTokenDecimals = await primaryTokenInstance.decimals();
    console.log("Initial initiator %s token balance: %s %s", primaryTokenName, formatUnits(intialPrimaryTokenBalance.toString(), primaryTokenDecimals), primaryTokenSymbol);
    
    console.log("");
    console.log("");
    
    console.log("Scenario #1: Escrow funds are released by Schedule");
    
    console.log("");
    console.log("");
    
    console.log("Transferring primary token to Escrow instance");
    
    await primaryTokenInstance.connect(initiator).transfer(instance.address, intialPrimaryTokenBalance.div(10_000));
    
    console.log("Initiating escrow");
    
    await instance.connect(initiator).initiateEscrow(beneficiaryAddress, ACA, intialPrimaryTokenBalance.div(100_000), 10);
    
    const escrowBlockNumber = await ethers.provider.getBlockNumber();
    
    console.log("Escrow initiation successful in block %s. Expected automatic completion in block %s", escrowBlockNumber, escrowBlockNumber + 10);
    
    const escrow = await instance.escrows(0);
    
    console.log("Escrow initiator:", escrow.initiator);
    console.log("Escrow beneficiary:", escrow.beneficiary);
    console.log("Escrow ingress token:", escrow.ingressToken);
    console.log("Escrow egress token:", escrow.egressToken);
    console.log("Escrow AUSD value:", escrow.AusdValue.toString());
    console.log("Escrow deadline:", escrow.deadline.toString());
    console.log("Escrow completed:", escrow.completed);
    
    console.log("Instantiating AUSD instance");
    const AusdInstance = new Contract(AUSD, TokenContract.abi, initiator);
    
    const initalBeneficiatyAusdBalance = await AusdInstance.balanceOf(beneficiaryAddress);
    
    console.log("Initial aUSD balance of beneficiary: %s AUSD", formatUnits(initalBeneficiatyAusdBalance.toString(), 12));
    
    console.log("Waiting for automatic release of funds");
    
    let currentBlockNumber = await ethers.provider.getBlockNumber();

    while(currentBlockNumber <= escrowBlockNumber + 10){
        console.log("Still waiting. Current block number is %s. Target block number is %s.", currentBlockNumber, escrowBlockNumber + 10);
        currentBlockNumber = await ethers.provider.getBlockNumber();
        await sleep(2500);
    }
    
    const finalBeneficiaryAusdBalance = await AusdInstance.balanceOf(beneficiaryAddress);
    
    console.log("Final aUSD balance of beneficiary: %s AUSD", formatUnits(finalBeneficiaryAusdBalance.toString(), 12));
    console.log("Beneficiary aUSD balance has increased for %s AUSD", formatUnits(finalBeneficiaryAusdBalance.sub(initalBeneficiatyAusdBalance).toString(), 12));
    
    console.log("");
    console.log("");
    console.log("Scenario #2: Escrow initiator releases the funds before the deadline");
    
    console.log("");
    console.log("");
    
    console.log("Initiating escrow");
    
    await instance.connect(initiator).initiateEscrow(beneficiaryAddress, ACA, intialPrimaryTokenBalance.div(100_000), 10);
    
    const escrowBlockNumber2 = await ethers.provider.getBlockNumber();
    
    console.log("Escrow initiation successful in block %s. Expected automatic completion in block %s", escrowBlockNumber2, escrowBlockNumber2 + 10);
    
    const escrow2 = await instance.escrows(1);
    
    console.log("Escrow initiator:", escrow2.initiator);
    console.log("Escrow beneficiary:", escrow2.beneficiary);
    console.log("Escrow ingress token:", escrow2.ingressToken);
    console.log("Escrow egress token:", escrow2.egressToken);
    console.log("Escrow AUSD value:", escrow2.AusdValue.toString());
    console.log("Escrow deadline:", escrow2.deadline.toString());
    console.log("Escrow completed:", escrow2.completed);
    
    const initalBeneficiatyAusdBalance2 = await AusdInstance.balanceOf(beneficiaryAddress);
    
    console.log("Initial aUSD balance of beneficiary: %s AUSD", formatUnits(initalBeneficiatyAusdBalance2.toString(), 12));
    
    console.log("Manually releasing the funds");
    
    await instance.connect(initiator).completeEscrow();
    
    let currentBlockNumber2 = await ethers.provider.getBlockNumber();
    
    const finalBeneficiaryAusdBalance2 = await AusdInstance.balanceOf(beneficiaryAddress);
    
    console.log("Escrow funds released at block %s, while the deadline was %s", currentBlockNumber2, escrow2.deadline);
    console.log("Final aUSD balance of beneficiary: %s AUSD", formatUnits(finalBeneficiaryAusdBalance2.toString(), 12));
    console.log("Beneficiary aUSD balance has increased for %s AUSD", formatUnits(finalBeneficiaryAusdBalance2.sub(initalBeneficiatyAusdBalance2).toString(), 12));
    
    console.log("");
    console.log("");
    
    console.log("Scenario #3: Beneficiary decided to be paid out in DOT");
    
    console.log("");
    console.log("");
    
    console.log("Initiating escrow");
    
    await instance.connect(initiator).initiateEscrow(beneficiaryAddress, ACA, intialPrimaryTokenBalance.div(100_000), 10);
    
    const escrowBlockNumber3 = await ethers.provider.getBlockNumber();
    
    console.log("Escrow initiation successful in block %s. Expected automatic completion in block %s", escrowBlockNumber3, escrowBlockNumber3 + 10);
    
    const escrow3 = await instance.escrows(2);

    console.log("Escrow initiator:", escrow3.initiator);
    console.log("Escrow beneficiary:", escrow3.beneficiary);
    console.log("Escrow ingress token:", escrow3.ingressToken);
    console.log("Escrow egress token:", escrow3.egressToken);
    console.log("Escrow AUSD value:", escrow3.AusdValue.toString());
    console.log("Escrow deadline:", escrow3.deadline.toString());
    console.log("Escrow completed:", escrow3.completed);
    
    console.log("Beneficiary setting the desired escrow egress token");
    
    await instance.connect(beneficiary).setEgressToken(DOT);
    
    console.log("Instantiating DOT instance");
    const DotInstance = new Contract(DOT, TokenContract.abi, initiator);
    
    const initalBeneficiatyDotBalance = await DotInstance.balanceOf(beneficiaryAddress);
    
    console.log("Initial DOT balance of beneficiary: %s DOT", formatUnits(initalBeneficiatyDotBalance.toString(), 12));
    
    console.log("Waiting for automatic release of funds");
    
    let currentBlockNumber3 = await ethers.provider.getBlockNumber();
    
    while(currentBlockNumber3 <= escrowBlockNumber3 + 10){
        console.log("Still waiting. Current block number is %s. Target block number is %s.", currentBlockNumber3, escrowBlockNumber3 + 10);
        currentBlockNumber3 = await ethers.provider.getBlockNumber();
        await sleep(2500);
    }
    
    const finalBeneficiaryDotBalance = await DotInstance.balanceOf(beneficiaryAddress);
    
    console.log("Final DOT balance of beneficiary: %s DOT", formatUnits(finalBeneficiaryDotBalance.toString(), 12));
    console.log("Beneficiary DOT balance has increased for %s DOT", formatUnits(finalBeneficiaryDotBalance.sub(initalBeneficiatyDotBalance).toString(), 12));
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
    console.error(error);
    process.exit(1);
    });
```

</details>

To be able to run the user journey script, we have to add two additional commands to the `"scripts”` section of `package.json`:

```json
   "user-journey-mandala": "hardhat run scripts/userJourney.js --network mandala",
   "user-journey-mandala:pubDev": "hardhat run scripts/userJourney.js --network mandalaPubDev"
```

These two commands allow us to run the user journey script in the local development network and in the public development network.

To be able to run the user journey script, we will be adding a `loop.js` helper, which should only be run when needed.

{% hint style="info" %}
**NOTE: If the `--instant-sealing` flag is used in the local development network, the block generation has to be forced and the tests might fail because of the package manager waiting for new blocks. To avoid script failure, a helper `loop.js` method is added to the `utils/` directory.**
{% endhint %}

Usage of `--instant-sealing` flag in a development network is beneficial, as it decreases the time needed to test out the behaviour of the project, but it also requires the `loop.js` helper, which forces the block creation. To add `loop.js` helper, run the following command in the root of your project:

```shell
touch utils/loop.js
```

The `loop.js` helper will continuously force the block generation using the `@polkadot/api` dependency that will interact directly with the local development network. In order ro be able to use it, we have to add it to our project with:

```shell
yarn add --dev @polkadot/api
```

At the beginning of the helper we import the `ApiPromise` and `WsProvider` from `@polkadot/api` and define a `sleep()` function, that we will use to ensure that block generation is forced every 2 seconds:

```javascript
const { ApiPromise, WsProvider } = require('@polkadot/api');

const sleep = async time => new Promise((resolve) => setTimeout(resolve, time));
```

Now we are ready to define the `loop()` function and call it. Within the definition we also ensure that interval for function execution is set for 1 second:

```javascript
const loop = async (interval = 2000) => {

};

loop();
```

At the beginning of the `loop` function definition, we create a `ENDPOINT_URL` variable to which we assign the URL of the web socket endpoint for the provider to use. First we check if there is an environment variable with it, if there isn't we use the default value. `provider` is connected directly to the local development network in stead of the RPC adapter and `api` is assigned `ApiPromise` to the `provider` at the top of the `loop()` function definition. Below we log the start of the loop to the console and define a `count` variable, which will be used to keep track of how many times the function has forced a block generation:

```javascript
  const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
  const provider = new WsProvider(ENDPOINT_URL);

  const api = await ApiPromise.create({ provider });
  
  console.log('Started forced block generation loop!')

  let count = 0;
```

Now that the setup for the continuous forced block generation is set up, we can add a `while` loop to constantly force the block generation. Within it we use the `interval` variable to ensure that the next step of the loop is executed 1 second after the previous one has ended and we use the `api` to generate a block. Before finishing the loop, we output the current number of times the block generation was forced us**i**ng this script:

```javascript
  while (true) {
    await sleep(interval);
    await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
    console.log(`Current number of force generated blocks: ${++count}`);
  }
```

{% hint style="info" %}
**NOTE: the first argument in `createBlock` is used to create an empty block if there are no transaction in the transaction pool and the second one is used to create a transaction only if there is at least one transaction within the transaction pool. As we only care that the next block is generated, we set both to `true`.**
{% endhint %}

<details>

<summary>Your utils/loop.js should look like this:</summary>

```javascript
const { ApiPromise, WsProvider } = require('@polkadot/api');

const sleep = async time => new Promise((resolve) => setTimeout(resolve, time));

const loop = async (interval = 2000) => {
  const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
  const provider = new WsProvider(ENDPOINT_URL);

  const api = await ApiPromise.create({ provider });
  
  console.log('Started forced block generation loop!')

  let count = 0;

  while (true) {
    await sleep(interval);
    await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
    console.log(`Current number of force generated blocks: ${++count}`);
  }
};

loop();
```

</details>

The last thing we need to do with the `loop.js` helper, is to add its execution to `scripts` section of our `package.json`. Since we will only be using it in a local development network that uses the `--instant-sealing` flag, we only need to add one execution script:

```json5
    "loop": "hardhat run utils/loop.js --network mandala"
```

This has to be run in its own terminal only when running the user journey script in the local development local development network that uses `--instant-sealing` flag with:

```shell
yarn loop
```

Running the user journey script in the local development environment should give you an output similar to this one:

```shell
yarn user-journey-mandala

yarn run v1.22.17
$ hardhat run scripts/userJourney.js --network mandala

Getting signers
Address of the initiator is 0x75E480dB528101a381Ce68544611C169Ad7EB342
Address of the beneficiary is 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6
Deploying AdvancedEscrow smart contract
AdvancedEscrow is deployed at address: 0xA8505C02cfd9d84389c11702C9994db27E4E2E1B
Instantiating ACA predeployed smart contract
Initial initiator Acala token balance: 9998842.411516250838 ACA


Scenario #1: Escrow funds are released by Schedule


Transferring primary token to Escrow instance
Initiating escrow
Escrow initiation successful in block 136. Expected automatic completion in block 146
Escrow initiator: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Escrow beneficiary: 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6
Escrow ingress token: 0x0000000000000000000100000000000000000000
Escrow egress token: 0x0000000000000000000000000000000000000000
Escrow AUSD value: 199637171695833
Escrow deadline: 146
Escrow completed: false
Instantiating AUSD instance
Initial aUSD balance of beneficiary: 10000399.559877590747 AUSD
Waiting for automatic release of funds
Still waiting. Current block number is 137. Target block number is 146.
Still waiting. Current block number is 137. Target block number is 146.
Still waiting. Current block number is 137. Target block number is 146.
Still waiting. Current block number is 138. Target block number is 146.
Still waiting. Current block number is 138. Target block number is 146.
Still waiting. Current block number is 139. Target block number is 146.
Still waiting. Current block number is 140. Target block number is 146.
Still waiting. Current block number is 141. Target block number is 146.
Still waiting. Current block number is 142. Target block number is 146.
Still waiting. Current block number is 142. Target block number is 146.
Still waiting. Current block number is 143. Target block number is 146.
Still waiting. Current block number is 144. Target block number is 146.
Still waiting. Current block number is 145. Target block number is 146.
Still waiting. Current block number is 146. Target block number is 146.
Still waiting. Current block number is 146. Target block number is 146.
Final aUSD balance of beneficiary: 10000599.19704928658 AUSD
Beneficiary aUSD balance has increased for 199.637171695833 AUSD


Scenario #2: Escrow initiator releases the funds before the deadline


Initiating escrow
Escrow initiation successful in block 149. Expected automatic completion in block 159
Escrow initiator: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Escrow beneficiary: 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6
Escrow ingress token: 0x0000000000000000000100000000000000000000
Escrow egress token: 0x0000000000000000000000000000000000000000
Escrow AUSD value: 199597288782589
Escrow deadline: 159
Escrow completed: false
Initial aUSD balance of beneficiary: 10000599.19704928658 AUSD
Manually releasing the funds
Escrow funds released at block 151, while the deadline was 159
Final aUSD balance of beneficiary: 10000798.794338069169 AUSD
Beneficiary aUSD balance has increased for 199.597288782589 AUSD


Scenario #3: Beneficiary decided to be paid out in DOT


Initiating escrow
Escrow initiation successful in block 152. Expected automatic completion in block 162
Escrow initiator: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Escrow beneficiary: 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6
Escrow ingress token: 0x0000000000000000000100000000000000000000
Escrow egress token: 0x0000000000000000000000000000000000000000
Escrow AUSD value: 199557417821676
Escrow deadline: 162
Escrow completed: false
Beneficiary setting the desired escrow egress token
Instantiating DOT instance
Initial DOT balance of beneficiary: 10000003.989610526389 DOT
Waiting for automatic release of funds
Still waiting. Current block number is 154. Target block number is 162.
Still waiting. Current block number is 154. Target block number is 162.
Still waiting. Current block number is 155. Target block number is 162.
Still waiting. Current block number is 155. Target block number is 162.
Still waiting. Current block number is 156. Target block number is 162.
Still waiting. Current block number is 157. Target block number is 162.
Still waiting. Current block number is 158. Target block number is 162.
Still waiting. Current block number is 158. Target block number is 162.
Still waiting. Current block number is 159. Target block number is 162.
Still waiting. Current block number is 160. Target block number is 162.
Still waiting. Current block number is 161. Target block number is 162.
Still waiting. Current block number is 162. Target block number is 162.
Still waiting. Current block number is 162. Target block number is 162.
Final DOT balance of beneficiary: 10000007.974382139931 DOT
Beneficiary DOT balance has increased for 3.984771613542 DOT
✨  Done in 83.36s.
```

## Conclusion

We have successfully built an `AdvancedEscrow` smart contract that allows users to deposit funds in one token and is paid out in another. It also supports automatic release of funds after a desired number of blocks. We added scripts and commands to run those scripts. To compile the smart contract, use `yarn build`. In order to deploy the smart contract to a local development network use `yarn deploy-mandala` and to deploy it to a public test network use `yarn deploy-mandala:pubDev`. We also created a script that simulates the user's journey through the use of the `AdvancedEscrow`. To run the script in the local development network use `yarn user-journey-mandala` and to run it on the public test network use `yarn user-journey-mandala:pubDev`.

This concludes our `AdvancedEscrow` tutorial. We hope you enjoyed this dive into Acala EVM+ and have gotten a satisfying glimpse of what the **+** stands for.

All of the Acalanauts wish you a pleasant journey into the future of web3!
