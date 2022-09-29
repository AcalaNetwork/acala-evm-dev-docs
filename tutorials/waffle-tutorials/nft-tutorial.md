---
description: A tutorial on how to build an ERC-721 compatible smart contract in Acala EVM+.
---

# NFT tutorial

## Table of contents

* [About](nft-tutorial.md#about)
* [Smart contract](nft-tutorial.md#smart-contract)
* [Test](nft-tutorial.md#test)
* [Deploy script](nft-tutorial.md#deploy-script)
* [Summary](nft-tutorial.md#summary)

## About

This is an example that builds upon the [token example](../truffle-tutorials/token-tutorial.md). `Token` was a simple example on building an ERC20 compatible token smart contract. `NFT` is an example of [ERC721](https://eips.ethereum.org/EIPS/eip-721) token implementation in Acala EVM+. We won't be building an administrated or upgradeable token and we will use OpenZeppelin [ERC721 implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol). For the setup and naming, replace the `token` with `NFT`. Let's jump into it!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/waffle-tutorials/tree/master/NFT](https://github.com/AcalaNetwork/waffle-tutorials/tree/master/NFT)
{% endhint %}

## Smart contract

In this tutorial we will be adding a simple smart contract that imports the ERC721 smart contract from `openzeppelin/contracts`, `Counters` utility from `@openzeppelin/conracts` and `ERC721URIStorage` and has a constructor that sets the name of the token and its abbreviation:

Your empty smart contract should look like this:

```solidity
pragma solidity =0.8.9;

contract NFT is ERC721URIStorage {
        
}
```

Import of the `ERC721`, `Counters` utility and `ERC721URIStorage` from `@openzeppelin/contracts` is done between the `pragma` definition and the start of the `contract` block. The `ERC721` contract is OpenZeppelin implementation of the ERC721 standard. `Counters` utility is used to increment `_tokenIds` every time a new token is minted. `ERC721URIStorage` is used to store the NFTs URI that point to the data associated to the tokens. The import statements look like this:

```solidity
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
```

As we now have access to `ERC721URIStorage.sol` from `@openzeppelin/contracts`, we can set the inheritance of our `NFT` contract:

```solidity
contract NFT is ERC721URIStorage {
```

Before we build the constructor, we have to specify the use of the `Counters` utility and define a `_tokenIds` counter:

```solidity
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;
```

The `constructor()` in itself doesn't set any parameters, but we will also include a call to the `ERC721` constructor, where we set the token name and symbol:

```solidity
    constructor() ERC721("Example non-fungible token", "eNFT") {}
```

We will be adding our own minting function. `recipient` will be the address that receives the token and `tokenURI` is the `URI` of the token's resource. We use the `_tokenIds` variable to separate one token from another and then call the inherited `_mint()` and `setTokenURI()`. Lastly we return the token ID:

```solidity
    function mintNFT(address recipient, string memory tokenURI) public returns (uint256) {
        _tokenIds.increment();

        uint256 newItemId = _tokenIds.current();
        _mint(recipient, newItemId);
        _setTokenURI(newItemId, tokenURI);

        return newItemId;
    }
```

Additionally we can override the `_baseURI` of our tokens. It's really simple as we only have the function definition and a return statement with `acala-evm+-tutorial-nft/`, which would represent a common base URI for our tokens:

```solidity
    function _baseURI() internal view virtual override returns (string memory) {
        return "acala-evm+-tutorial-nft/";
    }
```

This concludes our `NFT` smart contract.

<details>

<summary>Your contracts/Token.sol should look like this:</summary>

```solidity
pragma solidity =0.8.9;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";

contract NFT is ERC721URIStorage {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    constructor() ERC721("Example non-fungible token", "eNFT") {}

    function mintNFT(address recipient, string memory tokenURI) public returns (uint256) {
        _tokenIds.increment();

        uint256 newItemId = _tokenIds.current();
        _mint(recipient, newItemId);
        _setTokenURI(newItemId, tokenURI);

        return newItemId;
    }

    function _baseURI() internal view virtual override returns (string memory) {
        return "acala-evm+-tutorial-nft/";
    }
}
```

</details>

As the NFT smart contract is ready to be compiled, we can use the `yarn build` command (like we did in the hello-world) to compile the smart contract, which will create the `build` directory and contain the compiled smart contract.

## Test

Your test file should be named `NFT.test.ts` and the empty test along with the import statements should look like this:

```typescript
import { expect, use } from 'chai';
import { deployContract, solidity } from 'ethereum-waffle';
import { Contract } from 'ethers';

import { evmChai, Signer, TestProvider } from '@acala-network/bodhi';

import NFT from '../build/NFT.json';
import { getTestProvider } from '../utils/setup';

use(solidity);
use(evmChai);

const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

describe("NFT", () => {

});
```

First thing to add to the `NFT` describe block are the `provider`, `deployer`, `user`, `instance`, `deployerAddress` and `userAddress` variables. Within the `before` action we assign the `TestProvider` to `provider`, `Signer` to the `wallet` and `user` variables, `deployerAddress` and `userAddress` are used to store the addresses of `deployer` and `user` and deployed contract to `instance`. The `after` action will disconnect from the `provider`:

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
      instance = await deployContract(deployer, NFT);
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

1. The `name` should equal `Example non-fungible token`.
2. The `symbol` should equal `eNFT`.
3. The initial balance of the `deployer` account should equal `0`.
4. The contract should revert when trying to get the balance of the `0x0` address.

```typescript
      it("should set the correct NFT name", async () => {
        expect(await instance.name()).to.equal("Example non-fungible token");
      });

      it("should set the correct NFT symbol", async () => {
        expect(await instance.symbol()).to.equal("eNFT");
      });

      it("should assign the initial balance of the deployer", async () => {
        expect((await instance.balanceOf(deployerAddress)).toNumber()).to.equal(0);
      });

      it("should revert when trying to get the balance of the 0x0 address", async () => {
        await expect(instance.balanceOf(NULL_ADDRESS)).to
          .revertedWith("ERC721: balance query for the zero address");
      });
```

The `Operation` block in itself is separated into two `describe` blocks, which are separated in itself:

1. `minting`: Validates the correct operation of the smart contract related to minting.
2. `balances and ownerships`: Validates the correct operation of the smart contract related to balances and ownerships.
3. `approvals`: Validates the correct operation of the smart contract related to approvals.
4. `transfers`: Validates the correct operation of the smart contract related to transfers.

The contents of the `Operation` block should look like this:

```typescript
      describe("minting", () => {
        
      });

      describe("balances and ownerships", () => {
        
      });

      describe("approvals", () => {
        
      });

      describe("transfers", () => {
        
      });
```

The `minting` block validates the following:

1. When token is minted, the account balance reflects that.
2. `Transfer` event should be emitted.
3. Base URI should equal `acala-evm+-tutorial-nft/`.
4. Token URI should be built as expected.
5. User should be able to own multiple tokens.
6. Retrieving URI of a nonexistent token results in call being reverted.

These examples should look like this:

```typescript
        it("should emit Transfer event", async () => {
          await expect(instance.connect(deployer).mintNFT(userAddress, "testToken")).to
            .emit(instance, "Transfer")
            .withArgs(NULL_ADDRESS, userAddress, 1);
        });

        it("should mint token to an address", async () => {
          const initialBalance = await instance.balanceOf(userAddress);

          await instance.connect(deployer).mintNFT(userAddress, "");

          const finalBalance = await instance.balanceOf(userAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
        });

        it("should set the expected base URI", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          expect(await instance.tokenURI(1)).to.equal("acala-evm+-tutorial-nft/1")
        });

        it("should set the expected URI", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "amazing-token");

          expect(await instance.tokenURI(1)).to.equal("acala-evm+-tutorial-nft/amazing-token")
        });

        it("should allow user to own multiple tokens", async () => {
          const initialBalance = await instance.balanceOf(userAddress);

          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(deployer).mintNFT(userAddress, "");

          const finalBalance = await instance.balanceOf(userAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(2);
        });

        it("should revert when trying to get an URI of an nonexistent token", async () => {
          await expect(instance.tokenURI(42)).to
            .be.revertedWith("ERC721URIStorage: URI query for nonexistent token");
        });
```

The `balances and ownerships` block validates the following:

1. Call getting the balance of a 0x0 address should be reverted.
2. Call getting the owner of a nonexistent token should be reverted.
3. Token owner should be returned successfully.

These examples should look like this:

```typescript
        it("should revert when trying to get balance of 0x0 address", async () => {
          await expect(instance.balanceOf(NULL_ADDRESS)).to
            .be.revertedWith("ERC721: balance query for the zero address");
        });

        it("should revert when trying to get the owner of a nonexistent token", async () => {
          await expect(instance.ownerOf(42)).to
            .be.revertedWith("ERC721: owner query for nonexistent token");
        });

        it("should return the token owner", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          const owner = await instance.ownerOf(1);

          expect(owner).to.equal(userAddress);
        });
```

The `approvals` block validates the following:

1. Approval should be granted successfully.
2. `Approval` event should be emitted when granting an approval.
3. Transaction granting approval to self should be reverted.
4. Transaction granting approval for someone else's token should be reverted.
5. Trying to get approval of a nonexistent token should revert.
6. Before the approval is given, the query of who has the approval of the token should result in 0x0 address.
7. Approval can be set for all tokens.
8. Approval can be revoked for all tokens.
9. If the approval is set for all tokens, individual tokens don't reflect this.
10. Operator should be allowed to grant approval for a specific token.
11. `Approval` event should be emitted when operator grants approval for a specific token.
12. `ApprovalForAll` event should be emitted when granting approval for all.
13. `ApprovalForAll` event should be emitted when revoking approval for all.

These examples should look like this:

```typescript
        it("should grant an approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).approve(deployerAddress, 1);

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(deployerAddress);
        });
        
        it("should emit Approval event when granting approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(user).approve(deployerAddress, 1)).to
            .emit(instance, "Approval")
            .withArgs(userAddress, deployerAddress, 1);
        });
        
        it("should revert when trying to set token approval to self", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(user).approve(userAddress, 1)).to
            .be.revertedWith("ERC721: approval to current owner");
        });
        
        it("should revert when trying to grant approval for a token that is someone else's", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(deployer).approve(deployerAddress, 1)).to
            .be.revertedWith("ERC721: approve caller is not owner nor approved for all");
        });
        
        it("should revert when trying to get an approval of a nonexistent token", async () => {
          await expect(instance.getApproved(42)).to
            .be.revertedWith("ERC721: approved query for nonexistent token")
        });
        
        it("should return 0x0 address as approved for a token for which no approval is given", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(NULL_ADDRESS);
        });
        
        it("sets approval for all", async () => {
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          const approved = await instance.isApprovedForAll(userAddress, deployerAddress);

          expect(approved).to.be.true;
        });
        
        it("revokes approval for all", async () => {
          await instance.connect(user).setApprovalForAll(deployerAddress, true);
          await instance.connect(user).setApprovalForAll(deployerAddress, false);

          const approved = await instance.isApprovedForAll(userAddress, deployerAddress);

          expect(approved).to.be.false;
        });
        
        it("doesn't reflect operator approval in single token approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(NULL_ADDRESS);
        });
        
        it("should allow operator to grant allowance for a apecific token", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          await instance.connect(deployer).approve(deployerAddress, 1);

          const approval = await instance.getApproved(1);

          expect(approval).to.equal(deployerAddress);
        });
        
        it("should emit Approval event when operator grants approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          await expect(instance.connect(deployer).approve(deployerAddress, 1)).to
            .emit(instance, "Approval")
            .withArgs(userAddress, deployerAddress, 1);
        });
        
        it("should emit ApprovalForAll event when approving for all", async () => {
          await expect(instance.connect(user).setApprovalForAll(deployerAddress, true)).to
            .emit(instance, "ApprovalForAll")
            .withArgs(userAddress, deployerAddress, true);
        });
        
        it("should emit ApprovalForAll event when revoking approval for all", async () => {
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          await expect(instance.connect(user).setApprovalForAll(deployerAddress, false)).to
            .emit(instance, "ApprovalForAll")
            .withArgs(userAddress, deployerAddress, false);
        });
```

The `transfers` block validates the following:

1. Transfer of a token should reflect on the addresses balances.
2. `Transfer` event should be emitted when transferring a token.
3. Tokens should be able to be transferred by an account that was granted an allowance.
4. Allowance should be reset when token is transferred.

These examples should look like this:

We can now add the following test cases to our describe block:

```typescript
        it("should transfer the token", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          const initialBalance = await instance.balanceOf(deployerAddress);

          await instance.connect(user).transferFrom(userAddress, deployerAddress, 1);

          const finalBalance = await instance.balanceOf(deployerAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
        });
        
        it("should emit Transfer event", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(user).transferFrom(userAddress, deployerAddress, 1)).to
            .emit(instance, "Transfer")
            .withArgs(userAddress, deployerAddress, 1);
        });
        
        it("should allow transfer of the tokens if the allowance is given", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).approve(deployerAddress, 1);

          const initialBalance = await instance.balanceOf(deployerAddress);

          await instance.connect(deployer).transferFrom(userAddress, deployerAddress, 1);

          const finalBalance = await instance.balanceOf(deployerAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
        });
        
        it("should reset the allowance after the token is transferred", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).approve(deployerAddress, 1);

          await instance.connect(deployer).transferFrom(userAddress, deployerAddress, 1);

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(NULL_ADDRESS);
        });
```

With that, our test is ready to be run.

<details>

<summary>Your test/NFT.test.ts should look like this:</summary>

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

describe("NFT", () => {
    let provider: TestProvider;
    let deployer: Signer;
    let user: Signer;
    let instance: Contract;
    let deployerAddress: String;
    let userAddress: String;

    beforeEach(async () => {
      provider = await getTestProvider();
      [deployer, user] = await provider.getWallets();
      instance = await deployContract(deployer, NFT);
      deployerAddress = await deployer.getAddress();
      userAddress = await user.getAddress();
    });

    after(async () => {
      provider.api.disconnect();
    });

    describe("Deployment", () => {
      it("should set the correct NFT name", async () => {
        expect(await instance.name()).to.equal("Example non-fungible token");
      });

      it("should set the correct NFT symbol", async () => {
        expect(await instance.symbol()).to.equal("eNFT");
      });

      it("should assign the initial balance of the deployer", async () => {
        expect((await instance.balanceOf(deployerAddress)).toNumber()).to.equal(0);
      });

      it("should revert when trying to get the balance of the 0x0 address", async () => {
        await expect(instance.balanceOf(NULL_ADDRESS)).to
          .revertedWith("ERC721: balance query for the zero address");
      });
    });

    describe("Operation", () => {
      describe("minting", () => {
        it("should emit Transfer event", async () => {
          await expect(instance.connect(deployer).mintNFT(userAddress, "testToken")).to
            .emit(instance, "Transfer")
            .withArgs(NULL_ADDRESS, userAddress, 1);
        });

        it("should mint token to an address", async () => {
          const initialBalance = await instance.balanceOf(userAddress);

          await instance.connect(deployer).mintNFT(userAddress, "");

          const finalBalance = await instance.balanceOf(userAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
        });

        it("should set the expected base URI", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          expect(await instance.tokenURI(1)).to.equal("acala-evm+-tutorial-nft/1")
        });

        it("should set the expected URI", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "amazing-token");

          expect(await instance.tokenURI(1)).to.equal("acala-evm+-tutorial-nft/amazing-token")
        });

        it("should allow user to own multiple tokens", async () => {
          const initialBalance = await instance.balanceOf(userAddress);

          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(deployer).mintNFT(userAddress, "");

          const finalBalance = await instance.balanceOf(userAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(2);
        });

        it("should revert when trying to get an URI of an nonexistent token", async () => {
          await expect(instance.tokenURI(42)).to
            .be.revertedWith("ERC721URIStorage: URI query for nonexistent token");
        });
      });

      describe("balances and ownerships", () => {
        it("should revert when trying to get balance of 0x0 address", async () => {
          await expect(instance.balanceOf(NULL_ADDRESS)).to
            .be.revertedWith("ERC721: balance query for the zero address");
        });

        it("should revert when trying to get the owner of a nonexistent token", async () => {
          await expect(instance.ownerOf(42)).to
            .be.revertedWith("ERC721: owner query for nonexistent token");
        });

        it("should return the token owner", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          const owner = await instance.ownerOf(1);

          expect(owner).to.equal(userAddress);
        });
      });

      describe("approvals", () => {
        it("should grant an approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).approve(deployerAddress, 1);

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(deployerAddress);
        });
        
        it("should emit Approval event when granting approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(user).approve(deployerAddress, 1)).to
            .emit(instance, "Approval")
            .withArgs(userAddress, deployerAddress, 1);
        });
        
        it("should revert when trying to set token approval to self", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(user).approve(userAddress, 1)).to
            .be.revertedWith("ERC721: approval to current owner");
        });
        
        it("should revert when trying to grant approval for a token that is someone else's", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(deployer).approve(deployerAddress, 1)).to
            .be.revertedWith("ERC721: approve caller is not owner nor approved for all");
        });
        
        it("should revert when trying to get an approval of a nonexistent token", async () => {
          await expect(instance.getApproved(42)).to
            .be.revertedWith("ERC721: approved query for nonexistent token")
        });
        
        it("should return 0x0 address as approved for a token for which no approval is given", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(NULL_ADDRESS);
        });
        
        it("sets approval for all", async () => {
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          const approved = await instance.isApprovedForAll(userAddress, deployerAddress);

          expect(approved).to.be.true;
        });
        
        it("revokes approval for all", async () => {
          await instance.connect(user).setApprovalForAll(deployerAddress, true);
          await instance.connect(user).setApprovalForAll(deployerAddress, false);

          const approved = await instance.isApprovedForAll(userAddress, deployerAddress);

          expect(approved).to.be.false;
        });
        
        it("doesn't reflect operator approval in single token approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(NULL_ADDRESS);
        });
        
        it("should allow operator to grant allowance for a apecific token", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          await instance.connect(deployer).approve(deployerAddress, 1);

          const approval = await instance.getApproved(1);

          expect(approval).to.equal(deployerAddress);
        });
        
        it("should emit Approval event when operator grants approval", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          await expect(instance.connect(deployer).approve(deployerAddress, 1)).to
            .emit(instance, "Approval")
            .withArgs(userAddress, deployerAddress, 1);
        });
        
        it("should emit ApprovalForAll event when approving for all", async () => {
          await expect(instance.connect(user).setApprovalForAll(deployerAddress, true)).to
            .emit(instance, "ApprovalForAll")
            .withArgs(userAddress, deployerAddress, true);
        });
        
        it("should emit ApprovalForAll event when revoking approval for all", async () => {
          await instance.connect(user).setApprovalForAll(deployerAddress, true);

          await expect(instance.connect(user).setApprovalForAll(deployerAddress, false)).to
            .emit(instance, "ApprovalForAll")
            .withArgs(userAddress, deployerAddress, false);
        });
      });

      describe("transfers", () => {
        it("should transfer the token", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          const initialBalance = await instance.balanceOf(deployerAddress);

          await instance.connect(user).transferFrom(userAddress, deployerAddress, 1);

          const finalBalance = await instance.balanceOf(deployerAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
        });
        
        it("should emit Transfer event", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");

          await expect(instance.connect(user).transferFrom(userAddress, deployerAddress, 1)).to
            .emit(instance, "Transfer")
            .withArgs(userAddress, deployerAddress, 1);
        });
        
        it("should allow transfer of the tokens if the allowance is given", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).approve(deployerAddress, 1);

          const initialBalance = await instance.balanceOf(deployerAddress);

          await instance.connect(deployer).transferFrom(userAddress, deployerAddress, 1);

          const finalBalance = await instance.balanceOf(deployerAddress);

          expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
        });
        
        it("should reset the allowance after the token is transferred", async () => {
          await instance.connect(deployer).mintNFT(userAddress, "");
          await instance.connect(user).approve(deployerAddress, 1);

          await instance.connect(deployer).transferFrom(userAddress, deployerAddress, 1);

          const approved = await instance.getApproved(1);

          expect(approved).to.equal(NULL_ADDRESS);
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


  NFT
    Deployment
      ✔ should set the correct NFT name (283ms)
      ✔ should set the correct NFT symbol (173ms)
      ✔ should assign the initial balance of the deployer (45ms)
      ✔ should revert when trying to get the balance of the 0x0 address (49ms)
    Operation
      minting
        ✔ should emit Transfer event (372ms)
        ✔ should mint token to an address (1024ms)
        ✔ should set the expected base URI (881ms)
        ✔ should set the expected URI (275ms)
        ✔ should allow user to own multiple tokens (1241ms)
        ✔ should revert when trying to get an URI of an nonexistent token
      balances and ownerships
        ✔ should revert when trying to get balance of 0x0 address (80ms)
        ✔ should revert when trying to get the owner of a nonexistent token
        ✔ should return the token owner (555ms)
      approvals
        ✔ should grant an approval (658ms)
        ✔ should emit Approval event when granting approval (618ms)
        ✔ should revert when trying to set token approval to self (511ms)
        ✔ should revert when trying to grant approval for a token that is someone else's (518ms)
        ✔ should revert when trying to get an approval of a nonexistent token (76ms)
        ✔ should return 0x0 address as approved for a token for which no approval is given (508ms)
        ✔ sets approval for all (140ms)
        ✔ revokes approval for all (296ms)
        ✔ doesn't reflect operator approval in single token approval (602ms)
        ✔ should allow operator to grant allowance for a apecific token (755ms)
        ✔ should emit Approval event when operator grants approval (671ms)
        ✔ should emit ApprovalForAll event when approving for all (122ms)
        ✔ should emit ApprovalForAll event when revoking approval for all (236ms)
      transfers
        ✔ should transfer the token (626ms)
        ✔ should emit Transfer event (621ms)
        ✔ should allow transfer of the tokens if the allowance is given (914ms)
        ✔ should reset the allowance after the token is transferred (1267ms)


  30 passing (26s)

✨  Done in 47.88s.
```

## Deploy script

The `setup.ts` should remain the same as in the hello-world. The `deploy.ts` needs to have the same imports like the hello-world example, except for the smart contract we are importing:

```typescript
import { use } from 'chai';
import { ContractFactory } from 'ethers';

import { evmChai } from '@acala-network/bodhi';

import NFT from '../build/NFT.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {

}

main()
```

Within the definition of the `main` function, we first retrieve the `wallet` and `provider` from the `setup()`. Then we output `Deploy NFT` to the console and deploy the `NFT` smart contract and save it to `instance`. The address of the deployed smart contract is logged into the terminal and we retrieve the token name and symbol from the deployed smart contract. Next we mint a first NFT to the `wallet`'s address and retrieve its full URI. Lastly we output token name, symbol and the minted token's URI to the console. Finally we disconnect from the provider:

```typescript
    const { wallet, provider } = await setup();

    console.log('Deploy NFT');

    const instance = await ContractFactory.fromSolidity(NFT).connect(wallet).deploy();

    console.log("NFT address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();

    await instance.mintNFT(await wallet.getAddress(), "exclusive-first-token");

    const URI = await instance.tokenURI(1);

    console.log("NFT name:", name);
    console.log("NFT symbol:", symbol);
    console.log("Minted NFT URI:", URI);

    provider.api.disconnect();
```

<details>

<summary>Your src/deploy.ts should look like this:</summary>

```typescript
import { use } from 'chai';
import { ContractFactory } from 'ethers';

import { evmChai } from '@acala-network/bodhi';

import NFT from '../build/NFT.json';
import { setup } from '../utils/setup';

use(evmChai);

const main = async () => {
    const { wallet, provider } = await setup();

    console.log('Deploy NFT');

    const instance = await ContractFactory.fromSolidity(NFT).connect(wallet).deploy();

    console.log("NFT address:", instance.address);

    const name = await instance.name();
    const symbol = await instance.symbol();

    await instance.mintNFT(await wallet.getAddress(), "exclusive-first-token");

    const URI = await instance.tokenURI(1);

    console.log("NFT name:", name);
    console.log("NFT symbol:", symbol);
    console.log("Minted NFT URI:", URI);

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


Deploy NFT
NFT address: 0x0230135fDeD668a3F7894966b14F42E65Da322e4
NFT name: Example non-fungible token
NFT symbol: eNFT
Minted NFT URI: acala-evm+-tutorial-nft/exclusive-first-token
✨  Done in 34.74s.
```

## Summary

We have built upon the previous examples, added an ERC721 smart contract and tested all of its functionalities. The tests were more detailed and covered more examples. We also ensured that we can interact with the smart contract and that its storage is modified as expected. We can compile smart contract with `yarn build`, test it with `yarn test` and deploy it with `yarn deploy`.
