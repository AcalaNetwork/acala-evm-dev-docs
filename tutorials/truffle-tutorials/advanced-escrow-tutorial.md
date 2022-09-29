---
description: >-
  Tutorial on how to build an Escrow smart contract using Acala EVM+'s built in
  predeployed tokens, DEX and Schedule (an on-chain automation tool).
---

# Advanced Escrow tutorial

## Table of contents

* [Intro](advanced-escrow-tutorial.md#intro)
* [Setting up](advanced-escrow-tutorial.md#setting-up)
* [Smart contract](advanced-escrow-tutorial.md#smart-contract)
* [Deploy script](advanced-escrow-tutorial.md#deploy-script)
* [Test](advanced-escrow-tutorial.md#test)
* [User journey](advanced-escrow-tutorial.md#user-journey)
* [Conclusion](advanced-escrow-tutorial.md#conclusion)

## Intro

This tutorial dives into Acala EVM+ smart contract development using Truffle development framework. We will start with the setup, build the smart contract and write deployment, test and user journey scripts. The smart contract will allow users to initiate escrows in one currency, and for beneficiaries to specify if they desire to be paid in another currency. Another feature we will familiarise ourselves with is the on-chain automation using a predeployed smart contract called `Schedule`. Using it will allow us to set the automatic completion of escrow after a certain number of blocks are included in the blockchain.

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/truffle-tutorials/tree/master/advanced-escrow](https://github.com/AcalaNetwork/truffle-tutorials/tree/master/advanced-escrow)
{% endhint %}

Let's jump right in!

## Setting up

Assuming you have [Truffle](https://www.trufflesuite.com/docs/truffle/getting-started/installation) and yarn installed, we can jump right into creating a new Truffle project.

<details>

<summary>You can install Truffle using the following command:</summary>

```shell
yarn add -g truffle
```

</details>

1. Open a terminal window in a directory where you want your `AdvancedEscrow` example to reside, create a directory for it and then initialize a yarn project within it, as well as add Truffle as a dependency, with the following commands:

```bash
mkdir AdvancedEscrow
cd AdvancedEscrow
yarn init --yes
yarn add truffle
truffle init
```

In addition to initiating a Truffle project, Truffle has already created `contracts`, `migrations` and `test` directories that we require for this tutorial.

### Configure Truffle

As we will be using Truffle to compile, test and deploy the smart contract, we need to configure it. Uncomment the following line in the `truffle-config.js`:

**For me this is line 21, so even if there is any discrepancy between Truffle versions, this should narrow down the search.**

```js
const HDWalletProvider = require('@truffle/hdwallet-provider');
```

As you may have noticed, we are importing `@truffle/hdwallet-provider`, so we need to add it to the project. Let's add it as a development dependency:

```bash
yarn add --dev @truffle/hdwallet-provider
```

We will be enabling a Mandala local development network by adding it to the configuration. Let's add the its configuration. As the public test network has the same configuration as the local development network, let's add a helper function to the config. Make sure to add this helper method above the `module.exports` (you can place it below as well, just not within). We will call it `mandalaConfig` and expect one argument to be passed to it. The argument passed to it is called the `endpointUrl` and it specifies the RPC endpoint, to which the Truffle connects to. Copy the method into your `truffle-config.js`:

```javascript
const mandalaConfig = (endpointUrl) => ({
  provider: () =>
    new HDWalletProvider(mnemonicPhrase, endpointUrl),
  network_id: 595,
  gasPrice: 0x2f82f103ea, // storage_limit = 64001, validUntil = 360001, gasLimit = 10000000
  gas: 0x329b140,
  timeoutBlocks: 25,
  confirmations: 0
});
```

Let's break down this configuration:

* `provider` uses the `HDWalletProvided` that we imported. We pass the mnemonic prase, from which the accounts are derived, as well as the URL for the RPC endpoint of the desired network. You might have noticed that we haven't specified `mnemonicPhrase` anywhere just yet. Let's do it. Above the `mandalaConfig` add the following mnemonic:

```javascript
const mnemonicPhrase = 'fox sight canyon orphan hotel grow hedgehog build bless august weather swarm';
```

{% hint style="info" %}
**NOTE: This mnemonic phrase is used in all of the examples and represents the default development accounts of Acala EVM+. These accounts are not safe to use and you should use your own, following the secret management guidelines of** [**`HDWalletProvider`**](https://github.com/trufflesuite/truffle-hdwallet-provider)**.**
{% endhint %}

Now that we analyzed the `provider`, let's move on to the other arguments:

* `network_id` is the default network ID of the development Mandala network.
* `gasPrice` is the default gas price for the local development Mandala network. Commented out section represents additional parameters of Acala EVM+.
* `gas` is the current gas limit for transactions.
* `timeoutBlocks` and `confirmations` are set to our discretion and we opted for the values above in this tutorial.

{% hint style="warning" %}
The gas limit and gas price of the example might get out of sync as new blocks are mined on Mandala test network. If you encounter OutOfStorage error, or any other for that matter, while trying to deploy your smart contract using the Remix IDE, we suggest verifying these values following [these](../../network/gas-parameters.md#for-developers) instructions.
{% endhint %}

To be able to use the local development network and public test network within the project, we have to add them to the `networks` section of the config using the `mandalaConfig` helper. We do this by pasting the following two lines of code into it:

```javascript
    mandala: mandalaConfig("http://127.0.0.1:8545"),
    mandalaPublicDev: mandalaConfig("https://acala-mandala-adapter.api.onfinality.io/public"),
```

Now that Mandala local development network is added to our project, let's take care of the remaining configuration. Mocha timeout should be active, to make sure that we don't get stuck in a loop if something goes wrong during tests. For this line 91 (this is after the modifications) in `truffle-config.js` should be uncommented:

```javascript
    timeout: 100000
```

Lastly, let's set the compiler version to `0.8.9` as this is the Solidity version we will be using in our example smart contract. To do this, line 97 needs to be uncommented and modified to:

```javascript
      version: "0.8.9",    // Fetch exact version from solc-bin (default: truffle's version)
```

## Smart contract

The `AdvancedEscrow` smart contract, which we will add in the following section, will still leave some areas that could be improved. `Advanced` is referring to the use of the predeployed smart contracts in the Acala EVM+ rather than its operation.

When two parties enter into an escrow agreement, using the `AdvancedEscrow` smart contract, the party paying for the service first transfers the tokens from one of the predeployed ERC20 smart contracts into the escrow smart contract. The party then initiates the escrow within the smart contract. Initiation of escrow requires both the contract address of the token being escrowed, and the wallet address of the beneficiary of escrow.

Upon initiation of the escrow, the smart contract exchanges the tokens coming into escrow for AUSD. Then it sets the deadline after which AUSD is released to the beneficiary. The beneficiary also has the ability to specify which tokens they want to receive from escrow and the smart contract exchanges the AUSD it is holding in escrow for the desired tokens upon completion of escrow.

We also allow for the escrow to be completed before the deadline, with the ability for the initiating party to release the funds to the beneficiary manually.

In order to add our smart contract, we will use Truffle built-in utility `create`:

```solidity
truffle create contract AdvancedEscrow
```

This command created a `HelloWorld.sol` file with a skeleton smart contract within `contracts` directory. First line of this smart contract, after the license definition, should specify the exact version of Solidity we will be using, which is `0.8.9`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

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
* `beneficiary`; The account that is to receive the escrowed funds
* `ingressToken`: Address of the token that was used to fund the escrow
* `egressToken`: Address of the token that will be used to pay out of the escrow
* `AusdValue`: Value of the escrow in AUSD
* `deadline`: Block number of the block after which, the escrow will be paid out
* `completed`: As an escrow can only be active or fulfilled, this can be represented as by a boolean value.

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

The event contains information about the current state of the latest escrow:

* `initiator`: Address of the account that initiated the escrow
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

<summary>Your contracts/AdvancedEscrow.sol should look like this:</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity =0.8.9;

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

Now that we have the smart contract ready, we have to compile it. For this, we will add the `build` script to the `package.json`. To do this, we have to add `scripts` section to it. We will be using Truffle's compile functionality, so the `scripts` section should look like this:

```json
  "scripts": {
    "build": "truffle compile"
  }
```

When you run the `build` command using `yarn build`, the `build` directory is created and it contains the compiled smart contract.

## Deploy script

Now that we have our smart contract ready, we can deploy it, so we can use it. We can again use the Truffle built-in utility `create` to create a migration file:

```shell
truffle create migration AdvancedEscrow
```

The utility created a barebones migration file in the `migrations` folder. First thing we need to do is import our smart contract into it. We do this with the following line of code at the top of the file:

```javascript
const AdvancedEscrow = artifacts.require('AdvancedEscrow');
```

To make sure that our migration will successfully deploy our smart contract, we have to make sure that our `deployer` is ready. To do that, we need to modfy the deployment function to be asynchronous. Replace the 3rd line of the migration with:

```javascript
module.exports = async function (deployer) {
```

Now that we have the smart contract imported within the migration, we can deploy the smart contract. We do this by invoking `deployer`, which is defined in the definition of the function. Additionally we will output the address of the deployed smart contract:

```javascript
  console.log('Deploy AdvancedEscrow');

  await deployer.deploy(AdvancedEscrow);

  console.log(`Advanced escrow deployed at: ${AdvancedEscrow.address}`);
```

This completes our migration and allows us to deploy the example smart contract as well as run tests for it.

To run the migration and deploy the smart contract, we have to add the scripts to deploy the smart contract to the local development network as well as public Mandala test network. To do this, we have to add them to the `scripts` section of the `package.json`:

```json
    "deploy-mandala": "truffle migrate --network mandala",
    "deploy-mandala:pubDev": "truffle migrate --network mandalaPublicDev"
```

Deploying the `AdvancedEscrow` smart contract using `yarn deploy-mandala` command should return the following output:

```shell
yarn deploy-mandala


yarn run v1.22.18
$ truffle migrate --network mandala

Compiling your contracts...
===========================
> Compiling ./../DEX/contracts/PrecompiledDEX.sol
> Artifacts written to /Users/jan/Acala/truffle-tutorials/AdvancedEscrow/build/contracts
> Compiled successfully using:
   - solc: 0.8.9+commit.e5eed63a.Emscripten.clang


Starting migrations...
======================
> Network name:    'mandala'
> Network id:      595
> Block gas limit: 15000000 (0xe4e1c0)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0x315d14d6fd5e6640b981537044b653774002bcfbbe9ff8cc8f6c54502328c8bc
   > Blocks: 0            Seconds: 0
   > contract address:    0x46CD18A2CE038D21b78dC3EF470CCf9Dc586AEa4
   > block number:        3871
   > block timestamp:     1652125822
   > account:             0x75E480dB528101a381Ce68544611C169Ad7EB342
   > balance:             9967868.909147408268
   > gas used:            250142 (0x3d11e)
   > gas price:           10770.794810139 gwei
   > value sent:          0 ETH
   > total cost:          2.694228155397789738 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     2.694228155397789738 ETH


1651609607_advanced_escrow.js
=============================
Deploy AdvancedEscrow

   Deploying 'AdvancedEscrow'
   --------------------------
   > transaction hash:    0x21a8bf1d1ec696d9ca0e2cf2a02919ea73483533b630392db596cfa59d18e7d0
   > Blocks: 0            Seconds: 0
   > contract address:    0xF49B534C00Fbeb4E7B055BCbAcDAC161BC4090F5
   > block number:        3873
   > block timestamp:     1652125834
   > account:             0x75E480dB528101a381Ce68544611C169Ad7EB342
   > balance:             9967865.664529159889
   > gas used:            2262155 (0x22848b)
   > gas price:           2010.260490745 gwei
   > value sent:          0 ETH
   > total cost:          4.547520820441255475 ETH

Advanced escrow deployed at: 0xF49B534C00Fbeb4E7B055BCbAcDAC161BC4090F5
   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     4.547520820441255475 ETH

Summary
=======
> Total deployments:   2
> Final cost:          7.241748975839045213 ETH


✨  Done in 5.64s.
```

## Test

To add a test for the smart contract, we can again use the Truffle built-in `create` utility:

```shell
truffle create test AdvancedEscrow
```

At the beginning of the file, we will be importing all of the constants and methods that we will require to successfully run the tests. The predeploy smart contract addresses are imported from `@acala-network/contracts` `ADDRESS` utility. We require the compiled artifacts of the smart contracts that we will be using within the test. This is why we assign them to the `AdvancedEscrow`, `PrecompiledToken` and `PrecompiledDEX` constants. In order to be able to test the block based deadlines, we need the ability to force the block generation within tests. For this reason, we import the `ApiPromise` and `WsProvider` from `@polkadot/api`, which we need to add to the project. As initiating the `ApiPromise` generates a lot of output, our test output would get very messy if we didn't silence it. To do this we use the `console.mute` dependency, that we have to add to the project, along with `@polkadot/api`, by using:

```shell
yarn add --dev console.mute @polkadot/api
```

In order to be able to validate the expected reverts and event emissions, we will use `truffleAssert` method from `truffle-assertions` dependency, which we import using:

```shell
yarn add --dev truffle-assertions
```

To be able to easily validate things dependent on `0x0` address, we assign it to the `NULL_ADDRESS` constant. Lastly we configure the `ENDPOINT_URL` constant to be used by the `provider`. And instantiate the `WsProvider` to the `provider` constant. The test file with import statements and an empty test should look like this:

```javascript
const AdvancedEscrow = artifacts.require('AdvancedEscrow');
const PrecompiledDEX = artifacts.require('IDEX');
const PrecompiledToken = artifacts.require('Token');

const { ApiPromise, WsProvider } = require('@polkadot/api');
const truffleAssert = require('truffle-assertions');
require('console.mute');

const { ACA, AUSD, DOT, DEX } = require('@acala-network/contracts/utils/MandalaAddress');
const NULL_ADDRESS = '0x0000000000000000000000000000000000000000';
const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
const provider = new WsProvider(ENDPOINT_URL);

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract('AdvancedEscrow', function (accounts) {
  
});
```

To setup for each of the test examples we define the `instance` variable which will hold the instance of the smart contract that we will be testing against and the `ACAinstance`, `AUSDinstance`, `DOTinstance` and `DEXinstance` hold the instances of the predeployed smart contracts. The `deployer` and `user` hold the accounts we will be using within the tests. Finally the `api` variable holds the `ApiPromise`, which we will use to force the generation of blocks. As creation of `ApiPromise` generates a lot of console output, especially when being run before each of the test examples, we have to mute the console output before we create it and resume it after, to keep the expected behaviour of the console. All of the values are assigned in the `beforeEach` action:

```javascript
  let instance;
  let ACAinstance;
  let AUSDinstance;
  let DOTinstance;
  let DEXinstance;
  let deployer;
  let user;
  let api;

  beforeEach('setup development environment', async function () {
    [deployer, user] = accounts;
    instance = await AdvancedEscrow.new();
    ACAinstance = await PrecompiledToken.at(ACA);
    AUSDinstance = await PrecompiledToken.at(AUSD);
    DOTinstance = await PrecompiledToken.at(DOT);
    DEXinstance = await PrecompiledDEX.at(DEX);
    console.mute();
    api = await ApiPromise.create({ provider });
    console.resume();
  });
```

{% hint style="info" %}
**NOTE: You can see how we used the `ACA`, `AUSD`, `DOT` and `DEX` from the `ADDRESS` utility in order to set the addresses of our predeployed smart contract.**
{% endhint %}

Our test cases will be split into two groups. One will be called `Deployment` and it will verify that the deployed smart contract has expected values set before it is being used. The second one will be called `Operation` and it will validate the expected behaviour of our smart contract. The empty sections should look like this:

```javascript
  describe('Deployment', function () {
    
  });

  describe('Operation', function () {
    
  });
```

We will only have one example within the `Deployment` section and it will verify that the number of escrows in a newly deployed smart contract is set to `0`:

```javascript
    it('should set the initial number of escrows to 0', async function () {
      const numberOfEscrows = await instance.numberOfEscrows();

      expect(numberOfEscrows.isZero()).to.be.true;
    });
```

The `Operation` section will hold more test examples. We will be checking for the following cases:

1. Initiating an escrow with a beneficiary of `0x0` should revert.
2. Initiating an escrow with a token address of `0x0` should revert.
3. Initiating an escrow with a duration of `0` blocks should revert.
4. Initiating an escrow with the value of escrow being higher than the balance of the smart contract should revert.
5. Upon successfully initiating an escrow, `EscrowUpdate` should be emitted.
6. Upon successfully init**i**ating an escrow, the values defining it should correspond to those passed upon initiation.
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
    it('should revert when beneficiary is 0x0', async function () {
      await truffleAssert.reverts(
        instance.initiateEscrow(NULL_ADDRESS, ACA, 10_000, 10, { from: deployer }),
        'Escrow: beneficiary_ is 0x0'
      );
    });

    it('should revert when ingress token is 0x0', async function () {
      await truffleAssert.reverts(
        instance.initiateEscrow(user, NULL_ADDRESS, 10_000, 10, { from: deployer }),
        'Escrow: ingressToken_ is 0x0'
      );
    });

    it('should revert when period is 0', async function () {
      await truffleAssert.reverts(
        instance.initiateEscrow(user, ACA, 10_000, 0, { from: deployer }),
        'Escrow: period is 0'
      );
    });

    it('should revert when balance of the contract is lower than ingressValue', async function () {
      const balance = await ACAinstance.balanceOf(instance.address);

      expect(balance.lt(web3.utils.toBN('10000'))).to.be.true;

      await truffleAssert.reverts(
        instance.initiateEscrow(user, ACA, 10_000, 10, { from: deployer }),
        'Escrow: contract balance is less than ingress value'
      );
    });

    it('should initiate escrow and emit EscrowUpdate when initializing escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      truffleAssert.eventEmitted(
        await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer }),
        'EscrowUpdate',
        {
          initiator: deployer,
          beneficiary: user,
          AusdValue: expectedValue,
          fulfilled: false
        }
      );
    });

    it('should set the values of current escrow when initiating the escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });

      const blockNumber = await web3.eth.getBlock('latest');
      const currentId = await instance.numberOfEscrows();
      const escrow = await instance.escrows(currentId - 1);

      expect(escrow.initiator).to.equal(deployer);
      expect(escrow.beneficiary).to.equal(user);
      expect(escrow.ingressToken).to.equal(ACA);
      expect(escrow.egressToken).to.equal(NULL_ADDRESS);
      expect(escrow.AusdValue.eq(expectedValue)).to.be.true;
      expect(escrow.deadline == (blockNumber.number + 1)).to.be.true;
      expect(escrow.completed).to.be.false;
    });

    it('should revert when initiating a new escrow when there is a preexisting active escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      await truffleAssert.reverts(
        instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer }),
        'Escrow: current escrow not yet completed'
      );
    });

    it('should revert when trying to set the egress token after the escrow has already been completed', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

      await truffleAssert.reverts(
        instance.setEgressToken(DOT, { from: user }),
        'Escrow: already completed'
      );
    });

    it('should revert when trying to set the egress token while not being the beneficiary', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

      await truffleAssert.reverts(
        instance.setEgressToken(DOT, { from: deployer }),
        'Escrow: already completed'
      );
    });

    it('should revert when trying to set the egress token while not being the beneficiary', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });

      await truffleAssert.reverts(
        instance.setEgressToken(DOT, { from: deployer }),
        'Escrow: sender is not beneficiary'
      );
    });

    it('should update the egress token', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      const blockNumber = await web3.eth.getBlock('latest');

      await instance.setEgressToken(DOT, { from: user });

      const currentId = await instance.numberOfEscrows();
      const escrow = await instance.escrows(currentId - 1);

      expect(escrow.initiator).to.equal(deployer);
      expect(escrow.beneficiary).to.equal(user);
      expect(escrow.ingressToken).to.equal(ACA);
      expect(escrow.egressToken).to.equal(DOT);
      expect(escrow.AusdValue.eq(expectedValue)).to.be.true;
      expect(escrow.deadline == (blockNumber.number + 10)).to.be.true;
      expect(escrow.completed).to.be.false;
    });

    it('should revert when trying to complete an already completed escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
  
      await truffleAssert.reverts(
        instance.completeEscrow({ from: deployer }),
        'Escrow: escrow already completed'
      );
    });
  
    it('should revert when trying to complete an escrow when not being the initiator', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
  
      await truffleAssert.reverts(
        instance.completeEscrow({ from: user }),
        'Escrow: caller is not initiator or this contract'
      );
    });

    it('should pay out the escrow in AUSD if no egress token is set', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
      const initalBalance = await AUSDinstance.balanceOf(user);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      await instance.completeEscrow({ from: deployer });
      const finalBalance = await AUSDinstance.balanceOf(user);

      expect(finalBalance.gt(initalBalance)).to.be.true;
    });

    it('should pay out the escrow in set token when egress token is set', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
      const initalBalance = await DOTinstance.balanceOf(user);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });
      await instance.setEgressToken(DOT, { from: user });

      await instance.completeEscrow({ from: deployer });
      const finalBalance = await DOTinstance.balanceOf(user);

      expect(finalBalance.gt(initalBalance)).to.be.true;
    });

    it('should not pay out the escrow in set AUSD when egress token is set', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
      const initalBalance = await AUSDinstance.balanceOf(user);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });
      await instance.setEgressToken(DOT, { from: user });

      await instance.completeEscrow({ from: deployer });
      const finalBalance = await AUSDinstance.balanceOf(user);

      expect(finalBalance.eq(initalBalance)).to.be.true;
    });

    it('should emit EscrowUpdate when escrow is completed', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      truffleAssert.eventEmitted(
        await instance.completeEscrow({ from: deployer }),
        'EscrowUpdate',
        {
          initiator: deployer,
          beneficiary: user,
          AusdValue: expectedValue,
          fulfilled: true
        }
      );
    });

    it('should automatically complete the escrow when given number of blocks has passed', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });

      const currentEscrow = await instance.numberOfEscrows();
      const initalState = await instance.escrows(currentEscrow - 1);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      const finalState = await instance.escrows(currentEscrow - 1);

      expect(initalState.completed).to.be.false;
      expect(finalState.completed).to.be.true;
    });
```

This concludes our test.

<details>

<summary>Your test/AdvancedEscrow.js should look like this:</summary>

```javascript
const AdvancedEscrow = artifacts.require('AdvancedEscrow');
const PrecompiledDEX = artifacts.require('IEX');
const PrecompiledToken = artifacts.require('Token');

const { ApiPromise, WsProvider } = require('@polkadot/api');
const truffleAssert = require('truffle-assertions');
require('console.mute');

const { ACA, AUSD, DOT, DEX } = require('@acala-network/contracts/utils/MandalaAddress');
const NULL_ADDRESS = '0x0000000000000000000000000000000000000000';
const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
const provider = new WsProvider(ENDPOINT_URL);

/*
* uncomment accounts to access the test accounts made available by the
* Ethereum client
* See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
*/
contract('AdvancedEscrow', function (accounts) {
  let instance;
  let ACAinstance;
  let AUSDinstance;
  let DOTinstance;
  let DEXinstance;
  let deployer;
  let user;
  let api;

  beforeEach('setup development environment', async function () {
    [deployer, user] = accounts;
    instance = await AdvancedEscrow.new();
    ACAinstance = await PrecompiledToken.at(ACA);
    AUSDinstance = await PrecompiledToken.at(AUSD);
    DOTinstance = await PrecompiledToken.at(DOT);
    DEXinstance = await PrecompiledDEX.at(DEX);
    console.mute();
    api = await ApiPromise.create({ provider });
    console.resume();
  });

  describe('Deployment', function () {
    it('should set the initial number of escrows to 0', async function () {
      const numberOfEscrows = await instance.numberOfEscrows();

      expect(numberOfEscrows.isZero()).to.be.true;
    });
  });

  describe('Operation', function () {
    it('should revert when beneficiary is 0x0', async function () {
      await truffleAssert.reverts(
        instance.initiateEscrow(NULL_ADDRESS, ACA, 10_000, 10, { from: deployer }),
        'Escrow: beneficiary_ is 0x0'
      );
    });

    it('should revert when ingress token is 0x0', async function () {
      await truffleAssert.reverts(
        instance.initiateEscrow(user, NULL_ADDRESS, 10_000, 10, { from: deployer }),
        'Escrow: ingressToken_ is 0x0'
      );
    });

    it('should revert when period is 0', async function () {
      await truffleAssert.reverts(
        instance.initiateEscrow(user, ACA, 10_000, 0, { from: deployer }),
        'Escrow: period is 0'
      );
    });

    it('should revert when balance of the contract is lower than ingressValue', async function () {
      const balance = await ACAinstance.balanceOf(instance.address);

      expect(balance.lt(web3.utils.toBN('10000'))).to.be.true;

      await truffleAssert.reverts(
        instance.initiateEscrow(user, ACA, 10_000, 10, { from: deployer }),
        'Escrow: contract balance is less than ingress value'
      );
    });

    it('should initiate escrow and emit EscrowUpdate when initializing escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      truffleAssert.eventEmitted(
        await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer }),
        'EscrowUpdate',
        {
          initiator: deployer,
          beneficiary: user,
          AusdValue: expectedValue,
          fulfilled: false
        }
      );
    });

    it('should set the values of current escrow when initiating the escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });

      const blockNumber = await web3.eth.getBlock('latest');
      const currentId = await instance.numberOfEscrows();
      const escrow = await instance.escrows(currentId - 1);

      expect(escrow.initiator).to.equal(deployer);
      expect(escrow.beneficiary).to.equal(user);
      expect(escrow.ingressToken).to.equal(ACA);
      expect(escrow.egressToken).to.equal(NULL_ADDRESS);
      expect(escrow.AusdValue.eq(expectedValue)).to.be.true;
      expect(escrow.deadline == (blockNumber.number + 1)).to.be.true;
      expect(escrow.completed).to.be.false;
    });

    it('should revert when initiating a new escrow when there is a preexisting active escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      await truffleAssert.reverts(
        instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer }),
        'Escrow: current escrow not yet completed'
      );
    });

    it('should revert when trying to set the egress token after the escrow has already been completed', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

      await truffleAssert.reverts(
        instance.setEgressToken(DOT, { from: user }),
        'Escrow: already completed'
      );
    });

    it('should revert when trying to set the egress token while not being the beneficiary', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);

      await truffleAssert.reverts(
        instance.setEgressToken(DOT, { from: deployer }),
        'Escrow: already completed'
      );
    });

    it('should revert when trying to set the egress token while not being the beneficiary', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });

      await truffleAssert.reverts(
        instance.setEgressToken(DOT, { from: deployer }),
        'Escrow: sender is not beneficiary'
      );
    });

    it('should update the egress token', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      const blockNumber = await web3.eth.getBlock('latest');

      await instance.setEgressToken(DOT, { from: user });

      const currentId = await instance.numberOfEscrows();
      const escrow = await instance.escrows(currentId - 1);

      expect(escrow.initiator).to.equal(deployer);
      expect(escrow.beneficiary).to.equal(user);
      expect(escrow.ingressToken).to.equal(ACA);
      expect(escrow.egressToken).to.equal(DOT);
      expect(escrow.AusdValue.eq(expectedValue)).to.be.true;
      expect(escrow.deadline == (blockNumber.number + 10)).to.be.true;
      expect(escrow.completed).to.be.false;
    });

    it('should revert when trying to complete an already completed escrow', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
  
      await truffleAssert.reverts(
        instance.completeEscrow({ from: deployer }),
        'Escrow: escrow already completed'
      );
    });
  
    it('should revert when trying to complete an escrow when not being the initiator', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });
  
      await truffleAssert.reverts(
        instance.completeEscrow({ from: user }),
        'Escrow: caller is not initiator or this contract'
      );
    });

    it('should pay out the escrow in AUSD if no egress token is set', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
      const initalBalance = await AUSDinstance.balanceOf(user);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      await instance.completeEscrow({ from: deployer });
      const finalBalance = await AUSDinstance.balanceOf(user);

      expect(finalBalance.gt(initalBalance)).to.be.true;
    });

    it('should pay out the escrow in set token when egress token is set', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
      const initalBalance = await DOTinstance.balanceOf(user);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });
      await instance.setEgressToken(DOT, { from: user });

      await instance.completeEscrow({ from: deployer });
      const finalBalance = await DOTinstance.balanceOf(user);

      expect(finalBalance.gt(initalBalance)).to.be.true;
    });

    it('should not pay out the escrow in set AUSD when egress token is set', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);
      const initalBalance = await AUSDinstance.balanceOf(user);
  
      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
  
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });
      await instance.setEgressToken(DOT, { from: user });

      await instance.completeEscrow({ from: deployer });
      const finalBalance = await AUSDinstance.balanceOf(user);

      expect(finalBalance.eq(initalBalance)).to.be.true;
    });

    it('should emit EscrowUpdate when escrow is completed', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });

      const expectedValue = await DEXinstance.getSwapTargetAmount([ACA, AUSD], Math.floor(startingBalance/1000000));

      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 10, { from: deployer });

      truffleAssert.eventEmitted(
        await instance.completeEscrow({ from: deployer }),
        'EscrowUpdate',
        {
          initiator: deployer,
          beneficiary: user,
          AusdValue: expectedValue,
          fulfilled: true
        }
      );
    });

    it('should automatically complete the escrow when given number of blocks has passed', async function () {
      const startingBalance = await ACAinstance.balanceOf(deployer);

      await ACAinstance.transfer(instance.address, Math.floor(startingBalance/100000), { from: deployer });
      await instance.initiateEscrow(user, ACA, Math.floor(startingBalance/1000000), 1, { from: deployer });

      const currentEscrow = await instance.numberOfEscrows();
      const initalState = await instance.escrows(currentEscrow - 1);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
      const finalState = await instance.escrows(currentEscrow - 1);

      expect(initalState.completed).to.be.false;
      expect(finalState.completed).to.be.true;
    });
  });
});
```

</details>

As our test is ready to be run, we have to add the scripts to the `scripts` section of the `package.json` to be able to run the test. We will be adding two scripts. One to run the tests on the local development network and on the public test network:

```json
    "test-mandala": "truffle test --network mandala",
    "test-mandala:pubDev": "truffle test --network mandalaPublicDev"
```

When you run the test with (for example) `yarn test-mandala`, your tests should pass with the following output:

```shell
yarn test-mandala


yarn run v1.22.18
$ truffle test --network mandala
Using network 'mandala'.


Compiling your contracts...
===========================
> Compiling ./../DEX/contracts/PrecompiledDEX.sol
> Artifacts written to /var/folders/_c/x274s0_x6qj1xtkv60pllc800000gp/T/test--89218-nAYUrlX3849G
> Compiled successfully using:
   - solc: 0.8.9+commit.e5eed63a.Emscripten.clang
Deploy AdvancedEscrow
Advanced escrow deployed at: 0x952F4783C042f51B8FD6325eFA88563e21210C9b


  Contract: AdvancedEscrow
    Deployment
      ✓ should set the initial number of escrows to 0
    Operation
      ✓ should revert when beneficiary is 0x0 (107ms)
      ✓ should revert when ingress token is 0x0 (75ms)
      ✓ should revert when period is 0 (68ms)
      ✓ should revert when balance of the contract is lower than ingressValue (128ms)
      ✓ should initiate escrow and emit EscrowUpdate when initializing escrow (525ms)
      ✓ should set the values of current escrow when initiating the escrow (724ms)
      ✓ should revert when initiating a new escrow when there is a preexisting active escrow (491ms)
      ✓ should revert when trying to set the egress token after the escrow has already been completed (717ms)
      ✓ should revert when trying to set the egress token while not being the beneficiary (692ms)
      ✓ should revert when trying to set the egress token while not being the beneficiary (919ms)
      ✓ should update the egress token (1090ms)
      ✓ should revert when trying to complete an already completed escrow (937ms)
      ✓ should revert when trying to complete an escrow when not being the initiator (907ms)
      ✓ should pay out the escrow in AUSD if no egress token is set (1433ms)
      ✓ should pay out the escrow in set token when egress token is set (1754ms)
      ✓ should not pay out the escrow in set AUSD when egress token is set (1855ms)
      ✓ should emit EscrowUpdate when escrow is completed (1458ms)
      ✓ should automatically complete the escrow when given number of blocks has passed (1513ms)


  19 passing (33s)

✨  Done in 37.42s.
```

## User journey

Finally we are able to simulate the user journey through the `AdvancedEscrow`. We will create another script in the `scripts` directory called `userJourney.js` and add three scenarios to it:

Beneficiary accepts the funds in aUSD and the escrow is released by `Schedule` predeployed smart contract. Beneficiary accepts the funds in aUSD and the escrow is released by the initiator of the escrow, before it is released by the `Schedule`. Beneficiary decides to get paid in DOT and the escrow is released by the `Schedule`.

The simulation script will be called `userJourney.js` and will reside in the `scripts/` folder:

```shell
touch scripts/userJourney.js
```

The empty user journey script starts with the imports of  `AdvancedEscrow` and the precompiled `TokenContract` from `@acala-network/contracts`. Next we import the addresses of the `ACA`, `AUSD` and `DOT` predeployed token smart contracts, the `formatUnits` utility from `ethers` and `ApiPromise` and `WsProvider` from `@polkadot/api`. Finally we have to prepare the `ENDPOINT_URL` of the network node as well as a `provider` to be used to force the block generation in the script. The imports, constants and the empty script should look like this:

```javascript
const AdvancedEscrow = artifacts.require('AdvancedEscrow');
const TokenContract = artifacts.require('@acala-network/contracts/build/contracts/Token');

const { ACA, AUSD, DOT } = require('@acala-network/contracts/utils/MandalaAddress');
const { formatUnits } = require('ethers/lib/utils');
const { ApiPromise, WsProvider } = require('@polkadot/api');

const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
const provider = new WsProvider(ENDPOINT_URL);

module.exports = async function(callback) {
  try {
    
  }
  catch(error) {
    console.log(error)
  }

  callback()
}
```

At the beginning of the script we have to define the `api` that we connect to the provider and use to force the block generation:

```javascript
    const api = await ApiPromise.create({ provider });
```

In order to get a more verbose output, we will pad the log to the console with empty strings. Then we assign address values to the `initiator` and the `beneficiary`, deploy the `AdvancedEscrow` smart contract and instantiate the ACA ERC20 predeployed contract with the help of the `ADDRESS` utility. We will also output the formatted balance of the ACA token to the console:

```javascript
    console.log('');
    console.log('');

    console.log('Starting user journey');

    console.log('');
    console.log('');

    const accounts = await web3.eth.getAccounts();
    const initiator = accounts[0];
    const beneficiary = accounts[1];

    console.log(`Address of the initiator is ${initiator}`);
    console.log(`Address of the beneficiary is ${beneficiary}`);

    console.log('');
    console.log('');

    console.log('Deploying AdvancedEscrow smart contract');
    const instance = await AdvancedEscrow.new();

    console.log(`AdvancedEscrow is deployed at address ${instance.address}`);

    console.log('');
    console.log('');

    console.log('Instantiating ACA predeployed smart contract');
    const primaryTokenInstance = await TokenContract.at(ACA);

    const initialPrimaryTokenBalance = await primaryTokenInstance.balanceOf(initiator);
    const primaryTokenName = await primaryTokenInstance.name();
    const primaryTokenSymbol = await primaryTokenInstance.symbol();
    const primaryTokenDecimals = await primaryTokenInstance.decimals();
    console.log(
      `Initial initator ${primaryTokenName} token balance: ${formatUnits(initialPrimaryTokenBalance.toString(), primaryTokenDecimals.toString())} ${primaryTokenSymbol}`
    );
```

In the first scenario we will transfer the ACA token to the `AdvancedEscrow` smart contract and initiate the escrow. Then we will get the block number at which the escrow was initiated and output the block number of block in which the `Schedule` should automatically release the funds:

```javascript
    console.log('');
    console.log('');

    console.log('Scenario #1: Escrow funds are released by Schedule');

    console.log('');
    console.log('');
  
    console.log('Transferring primary token to Escrow instance');

    await primaryTokenInstance.transfer(instance.address, Math.floor(initialPrimaryTokenBalance/10000), { from: initiator });

    console.log('Initiating escrow');

    await instance.initiateEscrow(beneficiary, ACA, Math.floor(initialPrimaryTokenBalance/1000000), 10, { from: initiator });

    const escrowBlockNumber = await web3.eth.getBlock('latest');

    console.log(`Escrow initiation successful in block ${escrowBlockNumber.number}. Expected automatic completion in block ${escrowBlockNumber.number + 10}`);
```

{% hint style="warning" %}
**WARNING: As you might have noticed, we initiated the escrow using a tenth of the funds that we transferred to the smart contract. This is because the smart contract needs to have some free balance in order to be able to pay for the scheduled call.**
{% endhint %}

Since we made the `escrows` public, we can use the automatically generated getter, to get the information about the escrow we have just created and output it to the console:

```javascript
    const escrow = await instance.escrows(0);

    console.log('Escrow initiator:', escrow.initiator);
    console.log('Escrow beneficiary:', escrow.beneficiary);
    console.log('Escrow ingress token:', escrow.ingressToken);
    console.log('Escrow egress token:', escrow.egressToken);
    console.log('Escrow AUSD value:', escrow.AusdValue.toString());
    console.log('Escrow deadline:', escrow.deadline.toString());
    console.log('Escrow completed:', escrow.completed);
```

To make sure the escrow funds release actually increases the beneficiary’s funds, we need to instantiate the AUSD smart contract and get the initial balance of the beneficiary:

```javascript
    console.log('');
    console.log('');

    console.log('Instantiating AUSD instance');

    const AusdInstance = await TokenContract.at(AUSD);

    const initialBeneficiaryAusdBalance = await AusdInstance.balanceOf(beneficiary);


    console.log(`Initial AUSD balance of beneficiary: ${formatUnits(initialBeneficiaryAusdBalance.toString(), 12)} AUSD`);
```

{% hint style="info" %}
**NOTE: The predeployed ERC20 smart contracts always use 12 decimal spaces, which means, we have to pass the decimals argument to the `formatUnits` and we can not use the default value of 18.**
{% endhint %}

As we have to wait for the `Schedule` to release the escrowed funds, we need to add a while loop to check whether the block in which the `Schedule` will release the funds has already been added to the blockchain. In our user journey script, we will force a generation of a block each time the loop is executed:

```javascript
    console.log('Waiting for automatic release of funds');

    let currentBlockNumber = await web3.eth.getBlock('latest');

    while(currentBlockNumber.number <= escrowBlockNumber.number + 10) {
      console.log(`Still waiting. Current block number is ${currentBlockNumber.number}. Target block number is ${escrowBlockNumber.number + 10}`);
      currentBlockNumber = await web3.eth.getBlock('latest');
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
    }
```

All that is left to do in the first scenario is to get the final balance of the beneficiary and output it to the console along with the amount of funds, for which their balance has increased:

```javascript
    const finalBeneficiaryAusdBalance = await AusdInstance.balanceOf(beneficiary);

    console.log(`Final AUSD balance of beneficiary: ${formatUnits(finalBeneficiaryAusdBalance.toString(), 12)} AUSD`);
    console.log(`Beneficiary AUSD balance has increased for ${formatUnits((finalBeneficiaryAusdBalance-initialBeneficiaryAusdBalance).toString(), 12)} AUSD`);
```

In the second scenario, we will have the initiator releasing the funds before the `Schedule` has a chance to do so. We will add some more blank lines, so that the console output wil**l** be clearer, initiate a new escrow and output it’s details:

```javascript
    console.log('');
    console.log('');
  
    console.log('Scenario #2: Escrow initiator releases the funds before the deadline');

    console.log('');
    console.log('');
  
    console.log('Initiating escrow');

    await instance.initiateEscrow(beneficiary, ACA, Math.floor(initialPrimaryTokenBalance/100000), 10, { from: initiator });

    const escrowBlockNumber2 = await web3.eth.getBlock('latest');

    console.log(`Escrow initiation successfull in block ${escrowBlockNumber2.number}. Expected automatic completion in block ${escrowBlockNumber2.number + 10}.`);

    const escrow2 = await instance.escrows(1);

    console.log('Escrow initiator:', escrow2.initiator);
    console.log('Escrow beneficiary:', escrow2.beneficiary);
    console.log('Escrow ingress token:', escrow2.ingressToken);
    console.log('Escrow egress token:', escrow2.egressToken);
    console.log('Escrow AUSD value:', escrow2.AusdValue.toString());
    console.log('Escrow deadline:', escrow2.deadline.toString());
    console.log('Escrow completed:', escrow2.completed);
```

All that is left to do in this example is to get the balance of the beneficiary before and after the release of funds from the escrow, manually releasing the funds and logging the results to the console.

```javascript
    const initialBeneficiaryAusdBalance2 = await AusdInstance.balanceOf(beneficiary);

    console.log(`Initial AUSD balance of beneficiary: ${formatUnits(initialBeneficiaryAusdBalance2.toString(), 12)} AUSD`);

    console.log('Manually releasing the funds');

    await instance.completeEscrow({ from: initiator });

    let currentBlockNumber2 = await web3.eth.getBlock('latest');

    const finalBeneficiaryAusdBalance2 = await AusdInstance.balanceOf(beneficiary);

    console.log(`Escrow funds released at block ${currentBlockNumber2.number}, while deadline was ${escrow2.deadline}`);
    console.log(`Final AUSD balance of beneficiary: ${formatUnits(finalBeneficiaryAusdBalance2.toString(), 12)} AUSD`);
    console.log(`Beneficiary AUSD balance has increased for ${formatUnits((finalBeneficiaryAusdBalance2-initialBeneficiaryAusdBalance2).toString(), 12)} AUSD`);
```

{% hint style="info" %}
**NOTE: We didn’t have to instantiate AUSD smart contract here, because we already instantiated it in the first scenario.**
{% endhint %}

In the last scenario we will let the `Schedule` release the funds, but the beneficiary will set the egress token to DOT. The beginning of this example is similar to the ones before:

```javascript
    console.log('');
    console.log('');
  
    console.log('Scenario #3: Beneficiary decided to be paid out in DOT');

    console.log('');
    console.log('');
  
    console.log('Initiating escrow');
  
    await instance.initiateEscrow(beneficiary, ACA, Math.floor(initialPrimaryTokenBalance/100000), 10, { from: initiator });
  
    const escrowBlockNumber3 = await web3.eth.getBlock('latest');

    console.log(`Escrow initiation successful in block ${escrowBlockNumber3.number}. Expected automatic completion in block ${escrowBlockNumber3.number + 10}`);

    const escrow3 = await instance.escrows(2);

    console.log('Escrow initiator:', escrow3.initiator);
    console.log('Escrow beneficiary:', escrow3.beneficiary);
    console.log('Escrow ingress token:', escrow3.ingressToken);
    console.log('Escrow egress token:', escrow3.egressToken);
    console.log('Escrow AUSD value:', escrow3.AusdValue.toString());
    console.log('Escrow deadline:', escrow3.deadline.toString());
    console.log('Escrow completed:', escrow3.completed);
```

As the escrow is set up, beneficiary can now configure the egress token of the escrow:

```javascript
    console.log('Beneficiary setting the desired escrow egress token');

    await instance.setEgressToken(DOT, { from: beneficiary });
```

If we want to output the beneficiary’s DOT balance and the difference in balance after the funds are released from escrow, we need to instantiate the DOT predeployed smart contract. Now we can also output the initial DOT balance of the beneficiary:

```javascript
    console.log('Instantiating DOT instance');
    const DotInstance = await TokenContract.at(DOT);

    const initialBeneficiaryDotBalance = await DotInstance.balanceOf(beneficiary);

    console.log(`Initial DOT balance of beneficiary: ${formatUnits(initialBeneficiaryDotBalance.toString(), 12)} DOT`);
```

All that is left to do is to wait for the `Schedule` to release the funds and log the changes and results to the console:

```javascript
    console.log('Waiting for automatic release of funds');

    let currentBlockNumber3 = await web3.eth.getBlock('latest');

    while(currentBlockNumber3.number <= escrowBlockNumber3.number + 10){
      console.log(`Still waiting. Current block number is ${currentBlockNumber3.number}. Target block number is ${escrowBlockNumber3.number + 10}.`);
      currentBlockNumber3 = await web3.eth.getBlock('latest');
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
    }

    const finalBeneficiaryDotBalance = await DotInstance.balanceOf(beneficiary);

    console.log(`Final DOT balance of beneficiary: ${finalBeneficiaryDotBalance.toString(), 12}`);
    console.log(`Beneficiary DOT balance has increased by ${formatUnits((finalBeneficiaryDotBalance-initialBeneficiaryDotBalance).toString(), 12)} DOT`);
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
const AdvancedEscrow = artifacts.require('AdvancedEscrow');
const TokenContract = artifacts.require('@acala-network/contracts/build/contracts/Token');

const { ACA, AUSD, DOT } = require('@acala-network/contracts/utils/MandalaAddress');
const { formatUnits } = require('ethers/lib/utils');
const { ApiPromise, WsProvider } = require('@polkadot/api');

const ENDPOINT_URL = process.env.ENDPOINT_URL || 'ws://127.0.0.1:9944';
const provider = new WsProvider(ENDPOINT_URL);

module.exports = async function (callback) {
  try {
    const api = await ApiPromise.create({ provider });

    console.log('');
    console.log('');

    console.log('Starting user journey');

    console.log('');
    console.log('');

    const accounts = await web3.eth.getAccounts();
    const initiator = accounts[0];
    const beneficiary = accounts[1];

    console.log(`Address of the initiator is ${initiator}`);
    console.log(`Address of the beneficiary is ${beneficiary}`);

    console.log('');
    console.log('');

    console.log('Deploying AdvancedEscrow smart contract');
    const instance = await AdvancedEscrow.new();

    console.log(`AdvancedEscrow is deployed at address ${instance.address}`);

    console.log('');
    console.log('');

    console.log('Instantiating ACA predeployed smart contract');
    const primaryTokenInstance = await TokenContract.at(ACA);

    const initialPrimaryTokenBalance = await primaryTokenInstance.balanceOf(initiator);
    const primaryTokenName = await primaryTokenInstance.name();
    const primaryTokenSymbol = await primaryTokenInstance.symbol();
    const primaryTokenDecimals = await primaryTokenInstance.decimals();
    console.log(
      `Initial initator ${primaryTokenName} token balance: ${formatUnits(initialPrimaryTokenBalance.toString(), primaryTokenDecimals.toString())} ${primaryTokenSymbol}`
    );

    console.log('');
    console.log('');

    console.log('Scenario #1: Escrow funds are released by Schedule');

    console.log('');
    console.log('');
  
    console.log('Transferring primary token to Escrow instance');

    await primaryTokenInstance.transfer(instance.address, Math.floor(initialPrimaryTokenBalance/10000), { from: initiator });

    console.log('Initiating escrow');

    await instance.initiateEscrow(beneficiary, ACA, Math.floor(initialPrimaryTokenBalance/1000000), 10, { from: initiator });

    const escrowBlockNumber = await web3.eth.getBlock('latest');

    console.log(`Escrow initiation successful in block ${escrowBlockNumber.number}. Expected automatic completion in block ${escrowBlockNumber.number + 10}`);

    const escrow = await instance.escrows(0);

    console.log('Escrow initiator:', escrow.initiator);
    console.log('Escrow beneficiary:', escrow.beneficiary);
    console.log('Escrow ingress token:', escrow.ingressToken);
    console.log('Escrow egress token:', escrow.egressToken);
    console.log('Escrow AUSD value:', escrow.AusdValue.toString());
    console.log('Escrow deadline:', escrow.deadline.toString());
    console.log('Escrow completed:', escrow.completed);

    console.log('');
    console.log('');

    console.log('Instantiating AUSD instance');

    const AusdInstance = await TokenContract.at(AUSD);

    const initialBeneficiaryAusdBalance = await AusdInstance.balanceOf(beneficiary);


    console.log(`Initial AUSD balance of beneficiary: ${formatUnits(initialBeneficiaryAusdBalance.toString(), 12)} AUSD`);

    console.log('Waiting for automatic release of funds');

    let currentBlockNumber = await web3.eth.getBlock('latest');

    while(currentBlockNumber.number <= escrowBlockNumber.number + 10) {
      console.log(`Still waiting. Current block number is ${currentBlockNumber.number}. Target block number is ${escrowBlockNumber.number + 10}`);
      currentBlockNumber = await web3.eth.getBlock('latest');
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
    }

    const finalBeneficiaryAusdBalance = await AusdInstance.balanceOf(beneficiary);

    console.log(`Final AUSD balance of beneficiary: ${formatUnits(finalBeneficiaryAusdBalance.toString(), 12)} AUSD`);
    console.log(`Beneficiary AUSD balance has increased for ${formatUnits((finalBeneficiaryAusdBalance-initialBeneficiaryAusdBalance).toString(), 12)} AUSD`);

    console.log('');
    console.log('');
  
    console.log('Scenario #2: Escrow initiator releases the funds before the deadline');

    console.log('');
    console.log('');
  
    console.log('Initiating escrow');

    await instance.initiateEscrow(beneficiary, ACA, Math.floor(initialPrimaryTokenBalance/100000), 10, { from: initiator });

    const escrowBlockNumber2 = await web3.eth.getBlock('latest');

    console.log(`Escrow initiation successfull in block ${escrowBlockNumber2.number}. Expected automatic completion in block ${escrowBlockNumber2.number + 10}.`);

    const escrow2 = await instance.escrows(1);

    console.log('Escrow initiator:', escrow2.initiator);
    console.log('Escrow beneficiary:', escrow2.beneficiary);
    console.log('Escrow ingress token:', escrow2.ingressToken);
    console.log('Escrow egress token:', escrow2.egressToken);
    console.log('Escrow AUSD value:', escrow2.AusdValue.toString());
    console.log('Escrow deadline:', escrow2.deadline.toString());
    console.log('Escrow completed:', escrow2.completed);

    const initialBeneficiaryAusdBalance2 = await AusdInstance.balanceOf(beneficiary);

    console.log(`Initial AUSD balance of beneficiary: ${formatUnits(initialBeneficiaryAusdBalance2.toString(), 12)} AUSD`);

    console.log('Manually releasing the funds');

    await instance.completeEscrow({ from: initiator });

    let currentBlockNumber2 = await web3.eth.getBlock('latest');

    const finalBeneficiaryAusdBalance2 = await AusdInstance.balanceOf(beneficiary);

    console.log(`Escrow funds released at block ${currentBlockNumber2.number}, while deadline was ${escrow2.deadline}`);
    console.log(`Final AUSD balance of beneficiary: ${formatUnits(finalBeneficiaryAusdBalance2.toString(), 12)} AUSD`);
    console.log(`Beneficiary AUSD balance has increased for ${formatUnits((finalBeneficiaryAusdBalance2-initialBeneficiaryAusdBalance2).toString(), 12)} AUSD`);

    console.log('');
    console.log('');
  
    console.log('Scenario #3: Beneficiary decided to be paid out in DOT');

    console.log('');
    console.log('');
  
    console.log('Initiating escrow');
  
    await instance.initiateEscrow(beneficiary, ACA, Math.floor(initialPrimaryTokenBalance/100000), 10, { from: initiator });
  
    const escrowBlockNumber3 = await web3.eth.getBlock('latest');

    console.log(`Escrow initiation successful in block ${escrowBlockNumber3.number}. Expected automatic completion in block ${escrowBlockNumber3.number + 10}`);

    const escrow3 = await instance.escrows(2);

    console.log('Escrow initiator:', escrow3.initiator);
    console.log('Escrow beneficiary:', escrow3.beneficiary);
    console.log('Escrow ingress token:', escrow3.ingressToken);
    console.log('Escrow egress token:', escrow3.egressToken);
    console.log('Escrow AUSD value:', escrow3.AusdValue.toString());
    console.log('Escrow deadline:', escrow3.deadline.toString());
    console.log('Escrow completed:', escrow3.completed);

    console.log('Beneficiary setting the desired escrow egress token');

    await instance.setEgressToken(DOT, { from: beneficiary });

    console.log('Instantiating DOT instance');
    const DotInstance = await TokenContract.at(DOT);

    const initialBeneficiaryDotBalance = await DotInstance.balanceOf(beneficiary);

    console.log(`Initial DOT balance of beneficiary: ${formatUnits(initialBeneficiaryDotBalance.toString(), 12)} DOT`);

    console.log('Waiting for automatic release of funds');

    let currentBlockNumber3 = await web3.eth.getBlock('latest');

    while(currentBlockNumber3.number <= escrowBlockNumber3.number + 10){
      console.log(`Still waiting. Current block number is ${currentBlockNumber3.number}. Target block number is ${escrowBlockNumber3.number + 10}.`);
      currentBlockNumber3 = await web3.eth.getBlock('latest');
      await api.rpc.engine.createBlock(true /* create empty */, true /* finalize it*/);
    }

    const finalBeneficiaryDotBalance = await DotInstance.balanceOf(beneficiary);

    console.log(`Final DOT balance of beneficiary: ${finalBeneficiaryDotBalance.toString(), 12}`);
    console.log(`Beneficiary DOT balance has increased by ${formatUnits((finalBeneficiaryDotBalance-initialBeneficiaryDotBalance).toString(), 12)} DOT`);

    console.log('');
    console.log('');

    console.log('User journey completed!');
  } catch (error) {
    console.log(error);
  }

  callback();
};
```

</details>

To be able to run the user journey script, we have to add two additional commands to the `"scripts”` section of `package.json`:

```json
    "user-journey-mandala": "truffle exec scripts/userJourney.js --network mandala",
    "user-journey-mandala:pubDev": "truffle exec scripts/userJourney.js --network mandalaPublicDev"
```

These two commands allow us to run the user journey script in the local development network and in the public development network.

Running the `yarn user-journey-mandala` script should return the following output:

```shell
yarn user-journey-mandala


yarn run v1.22.18
$ truffle exec scripts/userJourney.js --network mandala
Using network 'mandala'.

2022-05-10 00:33:58        REGISTRY: Unknown signed extensions SetEvmOrigin found, treating them as no-effect
2022-05-10 00:33:58        API/INIT: RPC methods not decorated: evm_call, evm_estimateResources, oracle_getAllValues, oracle_getValue, tokens_queryExistentialDeposit


Starting user journey


Address of the initiator is 0x75E480dB528101a381Ce68544611C169Ad7EB342
Address of the beneficiary is 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6


Deploying AdvancedEscrow smart contract
AdvancedEscrow is deployed at address 0xbF4b38dC347Afe4fCEE0d468CAf62206ACA90C25


Instantiating ACA predeployed smart contract
Initial initator Acala token balance: 9967862.42006935056 ACA


Scenario #1: Escrow funds are released by Schedule


Transferring primary token to Escrow instance
Initiating escrow
Escrow initiation successful in block 3877. Expected automatic completion in block 3887
Escrow initiator: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Escrow beneficiary: 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6
Escrow ingress token: 0x0000000000000000000100000000000000000000
Escrow egress token: 0x0000000000000000000000000000000000000000
Escrow AUSD value: 19812638834622
Escrow deadline: 3887
Escrow completed: false


Instantiating AUSD instance
Initial AUSD balance of beneficiary: 10004219.6987726134 AUSD
Waiting for automatic release of funds
Still waiting. Current block number is 3877. Target block number is 3887
Still waiting. Current block number is 3877. Target block number is 3887
Still waiting. Current block number is 3878. Target block number is 3887
Still waiting. Current block number is 3879. Target block number is 3887
Still waiting. Current block number is 3880. Target block number is 3887
Still waiting. Current block number is 3881. Target block number is 3887
Still waiting. Current block number is 3882. Target block number is 3887
Still waiting. Current block number is 3883. Target block number is 3887
Still waiting. Current block number is 3883. Target block number is 3887
Still waiting. Current block number is 3885. Target block number is 3887
Still waiting. Current block number is 3886. Target block number is 3887
Still waiting. Current block number is 3887. Target block number is 3887
Final AUSD balance of beneficiary: 10004239.511411448022 AUSD
Beneficiary AUSD balance has increased for 19.812638834688 AUSD


Scenario #2: Escrow initiator releases the funds before the deadline


Initiating escrow
Escrow initiation successfull in block 3890. Expected automatic completion in block 3900.
Escrow initiator: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Escrow beneficiary: 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6
Escrow ingress token: 0x0000000000000000000100000000000000000000
Escrow egress token: 0x0000000000000000000000000000000000000000
Escrow AUSD value: 198104742797684
Escrow deadline: 3900
Escrow completed: false
Initial AUSD balance of beneficiary: 10004239.511411448022 AUSD
Manually releasing the funds
Escrow funds released at block 3891, while deadline was 3900
Final AUSD balance of beneficiary: 10004437.616154245706 AUSD
Beneficiary AUSD balance has increased for 198.104742797312 AUSD


Scenario #3: Beneficiary decided to be paid out in DOT


Initiating escrow
Escrow initiation successful in block 3892. Expected automatic completion in block 3902
Escrow initiator: 0x75E480dB528101a381Ce68544611C169Ad7EB342
Escrow beneficiary: 0x0085560b24769dAC4ed057F1B2ae40746AA9aAb6
Escrow ingress token: 0x0000000000000000000100000000000000000000
Escrow egress token: 0x0000000000000000000000000000000000000000
Escrow AUSD value: 198065379354743
Escrow deadline: 3902
Escrow completed: false
Beneficiary setting the desired escrow egress token
Instantiating DOT instance
Initial DOT balance of beneficiary: 10000019.039861390087 DOT
Waiting for automatic release of funds
Still waiting. Current block number is 3893. Target block number is 3902.
Still waiting. Current block number is 3893. Target block number is 3902.
Still waiting. Current block number is 3894. Target block number is 3902.
Still waiting. Current block number is 3895. Target block number is 3902.
Still waiting. Current block number is 3896. Target block number is 3902.
Still waiting. Current block number is 3897. Target block number is 3902.
Still waiting. Current block number is 3898. Target block number is 3902.
Still waiting. Current block number is 3899. Target block number is 3902.
Still waiting. Current block number is 3900. Target block number is 3902.
Still waiting. Current block number is 3901. Target block number is 3902.
Still waiting. Current block number is 3902. Target block number is 3902.
Final DOT balance of beneficiary: 12
Beneficiary DOT balance has increased by 3.949030729728 DOT


User journey completed!
✨  Done in 6.78s.
```

## Conclusion

We have successfully built an `AdvancedEscrow` smart contract that allows users to deposit funds in one token and to be paid out in another. It also supports automatic release of funds after a desired number of blocks. We added scripts and commands to run those scripts. To compile the smart contract, use `yarn build`. In order to deploy the smart contract to a local development network use `yarn deploy-mandala` and to deploy it to a public test network use `yarn deploy-mandala:pubDev`. We also created a script that simulates the user's journey through the use of the `AdvancedEscrow`. To run the script in the local development network use `yarn user-journey-mandala` and to run it on the public test network use `yarn user-journey-mandala:pubDev`. We also created a test script that can be run in the local development network using `yarn test-mandala` or in the public test network using `yarn test-mandala:pubDev`.

This concludes our `AdvancedEscrow` tutorial. We hope you enjoyed this dive into Acala EVM+ and have gotten a satisfying glimpse of what the **+** stands for.

All of the Acalanauts wish you a pleasant journey into the future of web3!
