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

This is an example that builds upon the [token](broken-reference) example. `Token` was a simple example on building an ERC20 compatible token smart contract. `NFT` is an example of [ERC721](https://eips.ethereum.org/EIPS/eip-721) token implementation in Acala EVM+. We won't be building an administrated or upgradeable token and we will use OpenZeppelin [ERC721 implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol). For the setup and naming, replace the `token` with `NFT`. Let's jump into it!

{% hint style="info" %}
NOTE: You can refer to the complete code of this tutorial at [https://github.com/AcalaNetwork/truffle-tutorials/tree/master/NFT](https://github.com/AcalaNetwork/truffle-tutorials/tree/master/NFT)
{% endhint %}

## Smart contract

In this tutorial we will be adding a simple smart contract that imports the ERC721 smart contract from `openzeppelin/contracts`, `Counters` utility from `@openzeppelin/conracts` and `ERC721URIStorage` and has a constructor that sets the name of the token and its abbreviation:

Your empty smart contract should look like this:

```solidity
// SPDX-License-Identifier: MIT
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

<summary>Your contracts/NFT.sol should look like this:</summary>

```solidity
    // SPDX-License-Identifier: MIT
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

As the NFT smart contract is ready to be compiled, we can use the `yarn build` command (like we did in the hello-world) to compile the smart contract, which will create the `artifacts` directory and contain the compiled smart contract.

## Test

Your test file should be called `NFT.js` and the empty test along with the import statements and null address constant should look like this:

```javascript
const NFT = artifacts.require("NFT");
const truffleAssert = require('truffle-assertions');
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

/*
 * uncomment accounts to access the test accounts made available by the
 * Ethereum client
 * See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
 */
contract("NFT", function (accounts) {
  
});
```

To prepare for the testing, we have to define `instance`, `deployer` and `user` global variables. The `instance` will store the deployed NFT smart contract and `deployer` and `user` variables are used to store accounts. Let's assign them values in the `beforeEach` action:

```javascript
  let instance;
  let deployer;
  let user;

  beforeEach("setup development environment", async function () {
    instance = await NFT.deployed();
    deployer = accounts[0];
    user = accounts[1];
  });
```

Our test will be split into two sections, `Deployment` and `Operation`:

```javascript
  describe("Deployment", function () {

  });

  describe("Operation", function () {
    
  });
```

`Deployment` block contains the examples that validate the expected initial state of the smart contract:

1. The contract should successfully deploy.
2. The `name` should equal `Example non-fungible token`.
3. The `symbol` should equal `eNFT`.
4. The initial balance of the `deployer` account should equal `0`.
5. The contract should revert when trying to get the balance of the `0x0` address.

```javascript
    it("should assert true", async function () {
      return assert.isTrue(true);
    });

    it("should set the correct NFT name", async function () {
      const name = await instance.name();

      expect(name).to.equal("Example non-fungible token");
    });

    it("should set the correct NFT symbol", async function () {
      const symbol = await instance.symbol();

      expect(symbol).to.equal("eNFT");
    });

    it("should assign the initial balance of the deployer", async function () {
      const balance = await instance.balanceOf(deployer);

      expect(balance.toNumber()).to.equal(0);
    });

    it("should revert when trying to get the balance of the 0x0 address", async function () {
      await truffleAssert.reverts(
        instance.balanceOf(NULL_ADDRESS),
        "ERC721: balance query for the zero address"
      );
    });
```

The `Operation` block in itself is separated into two `describe` blocks, which are separated in itself:

1. `minting`: Validates the correct operation of the smart contract related to minting.
2. `balances and ownerships`: Validates the correct operation of the smart contract related to balances and ownerships.
3. `approvals`: Validates the correct operation of the smart contract related to approvals.
4. `transfers`: Validates the correct operation of the smart contract related to transfers.

The contents of the `Operation` block should look like this:

```javascript
    describe("minting", function () {

    });

    describe("balances and ownerships", function () {
      
    });

    describe("approvals", function () {

    });

    describe("transfers", function () {
      
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

```javascript
      it("should emit Transfer event", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const event = response.logs[0].event;
        const from = response.logs[0].args.from;
        const to = response.logs[0].args.to;
        const tokenId = response.logs[0].args.tokenId;
        
        expect(event).to.equal("Transfer");
        expect(from).to.equal(NULL_ADDRESS);
        expect(to).to.equal(user);
        expect(tokenId.toNumber()).to.equal(1);
      });

      it("should mint token to an address", async function () {
        const initialBalance = await instance.balanceOf(user);

        await instance.mintNFT(user, "", { from: deployer });

        const finalBalance = await instance.balanceOf(user);

        expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
      });

      it("should set the expected base URI", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const URI = await instance.tokenURI(tokenId);

        expect(URI).to.equal("acala-evm+-tutorial-nft/" + tokenId);
      });

      it("should set the expected URI", async function () {
        const response = await instance.mintNFT(user, "testToken", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const URI = await instance.tokenURI(tokenId);

        expect(URI).to.equal("acala-evm+-tutorial-nft/testToken");
      });

      it("should allow user to own multiple tokens", async function () {
        const initialBalance = await instance.balanceOf(user);

        await instance.mintNFT(user, "", { from: deployer });
        await instance.mintNFT(user, "", { from: deployer });

        const finalBalance = await instance.balanceOf(user);

        expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(2);
      });

      it("should revert when trying to get an URI of an nonexistent token", async function () {
        await truffleAssert.reverts(
          instance.tokenURI(42),
          "ERC721URIStorage: URI query for nonexistent token"
        );
      });
```

The `balances and ownerships` block validates the following:

1. Call getting the balance of a 0x0 address should be reverted.
2. Call getting the owner of a nonexistent token should be reverted.
3. Token owner should be returned successfully.

These examples should look like this:

```javascript
      it("should revert when trying to get balance of 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.balanceOf(NULL_ADDRESS),
          "ERC721: balance query for the zero address"
        );
      });

      it("should revert when trying to get the owner of a nonexistent token", async function () {
        await truffleAssert.reverts(
          instance.ownerOf(42),
          "ERC721: owner query for nonexistent token"
        );
      });

      it("should return the token owner", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const owner = await instance.ownerOf(tokenId);

        expect(owner).to.equal(user);
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

```javascript
      it("should grant an approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        await instance.approve(deployer, tokenId, { from: user });

        const approved = await instance.getApproved(tokenId);

        expect(approved).to.equal(deployer);
      });

      it("should emit Approval event when granting approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const approval = await instance.approve(deployer, tokenId, { from: user });

        const event = approval.logs[0].event;
        const owner = approval.logs[0].args.owner;
        const approved = approval.logs[0].args.approved;
        const eventTokenId = approval.logs[0].args.tokenId;

        expect(event).to.equal("Approval");
        expect(owner).to.equal(user);
        expect(approved).to.equal(deployer);
        expect(eventTokenId.toNumber()).to.equal(tokenId);
      });

      it("should revert when trying to set token approval to self", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId;

        await truffleAssert.reverts(
          instance.approve(user, tokenId, { from: user }),
          "ERC721: approval to current owner"
        );
      });

      it("should revert when trying to grant approval for a token that is someone else's", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId;

        await truffleAssert.reverts(
          instance.approve(deployer, tokenId, { from: deployer }),
          "ERC721: approve caller is not owner nor approved for all"
        );
      });

      it("should revert when trying to get an approval of a nonexistent token", async function () {
        await truffleAssert.reverts(
          instance.getApproved(42),
          "ERC721: approved query for nonexistent token"
          );
      });

      it("should return 0x0 address as approved for a token for which no approval is given", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId;

        const approved = await instance.getApproved(tokenId.toNumber());

        expect(approved).to.equal(NULL_ADDRESS);
      });

      it("sets approval for all", async function () {
        await instance.setApprovalForAll(deployer, true, { from: user });

        const approved = await instance.isApprovedForAll(user, deployer);

        expect(approved).to.be.true;
      });

      it("revokes approval for all", async function () {
        await instance.setApprovalForAll(deployer, true, { from: user });

        const initiallyApproved = await instance.isApprovedForAll(user, deployer);

        expect(initiallyApproved).to.be.true;

        await instance.setApprovalForAll(deployer, false, { from: user });

        const finallyApproved = await instance.isApprovedForAll(user, deployer);

        expect(finallyApproved).to.be.false;
      });

      it("doesn't reflect operator approval in single token approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.setApprovalForAll(deployer, true, { from: user });

        const approved = await instance.getApproved(tokenId);

        expect(approved).to.equal(NULL_ADDRESS);
      });

      it("should allow operator to grant allowance for a apecific token", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.setApprovalForAll(deployer, true, { from: user });

        const initiallyApproved = await instance.getApproved(tokenId);

        await instance.approve(deployer, tokenId, { from: deployer });

        const finallyApproved = await instance.getApproved(tokenId);

        expect(initiallyApproved).to.equal(NULL_ADDRESS);
        expect(finallyApproved).to.equal(deployer);
      });

      it("should emit Approval event when operator grants approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.setApprovalForAll(deployer, true, { from: user });

        const initiallyApproved = await instance.getApproved(tokenId);

        const approval = await instance.approve(deployer, tokenId, { from: deployer });

        const event = approval.logs[0].event;
        const owner = approval.logs[0].args.owner;
        const approved = approval.logs[0].args.approved;
        const eventTokenId = approval.logs[0].args.tokenId;

        expect(event).to.equal("Approval");
        expect(owner).to.equal(user);
        expect(approved).to.equal(deployer);
        expect(eventTokenId.toNumber()).to.equal(tokenId);
      });

      it("should emit ApprovalForAll event when approving for all", async function () {
        const response = await instance.setApprovalForAll(deployer, true, { from: user });

        const event = response.logs[0].event;
        const owner = response.logs[0].args.owner;
        const operator = response.logs[0].args.operator;
        const approved = response.logs[0].args.approved;

        expect(event).to.equal("ApprovalForAll");
        expect(owner).to.equal(user);
        expect(operator).to.equal(deployer);
        expect(approved).to.be.true;
      });

      it("should emit ApprovalForAll event when revoking approval for all", async function () {
        await instance.setApprovalForAll(deployer, true, { from: user });
        const response = await instance.setApprovalForAll(deployer, false, { from: user });

        const event = response.logs[0].event;
        const owner = response.logs[0].args.owner;
        const operator = response.logs[0].args.operator;
        const approved = response.logs[0].args.approved;

        expect(event).to.equal("ApprovalForAll");
        expect(owner).to.equal(user);
        expect(operator).to.equal(deployer);
        expect(approved).to.be.false;
      });
```

The `transfers` block validates the following:

1. Transfer of a token should reflect on the addresses balances.
2. `Transfer` event should be emitted when transferring a token.
3. Tokens should be able to be transferred by an account that was granted an allowance.
4. Allowance should be reset when token is transferred.

These examples should look like this:

We can now add the following test cases to our describe block:

```javascript
      it("should transfer the token", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();

        const initialBalance = await instance.balanceOf(deployer);

        await instance.transferFrom(user, deployer, tokenId, { from: user });

        const finalBalance = await instance.balanceOf(deployer);

        expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
      });
      
      it("should emit Transfer event", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();

        const transfer = await instance.transferFrom(user, deployer, tokenId, { from: user });

        const event = transfer.logs[1].event;
        const from = transfer.logs[1].args.from;
        const to = transfer.logs[1].args.to;
        const responseTokenId = transfer.logs[1].args.tokenId.toNumber();

        expect(event).to.equal("Transfer");
        expect(from).to.equal(user);
        expect(to).to.equal(deployer);
        expect(responseTokenId).to.equal(tokenId);
      });
      
      it("should allow transfer of the tokens if the allowance is given", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.approve(deployer, tokenId, { from: user });

        const initalBalance = await instance.balanceOf(deployer);

        await instance.transferFrom(user, deployer, tokenId, { from: deployer });

        const finalBalance = await instance.balanceOf(deployer);

        expect(finalBalance.toNumber() - initalBalance.toNumber()).to.equal(1);
      });
      
      it("should reset the allowance after the token is transferred", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.approve(deployer, tokenId, { from: user });

        const initiallyApproved = await instance.getApproved(tokenId);

        await instance.transferFrom(user, deployer, tokenId, { from: user });

        const finallyApproved = await instance.getApproved(tokenId);

        expect(initiallyApproved).to.equal(deployer);
        expect(finallyApproved).to.equal(NULL_ADDRESS);
      });
```

With that, our test is ready to be run.

<details>

<summary>Your test/NFT.js should look like this:</summary>

```javascript
const NFT = artifacts.require("NFT");
const truffleAssert = require('truffle-assertions');
const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";

/*
* uncomment accounts to access the test accounts made available by the
* Ethereum client
* See docs: https://www.trufflesuite.com/docs/truffle/testing/writing-tests-in-javascript
*/
contract("NFT", function (accounts) {
  let instance;
  let deployer;
  let user;

  beforeEach("setup development environment", async function () {
    instance = await NFT.deployed();
    deployer = accounts[0];
    user = accounts[1];
  });

  describe("Deployment", function () {
    it("should assert true", async function () {
      return assert.isTrue(true);
    });

    it("should set the correct NFT name", async function () {
      const name = await instance.name();

      expect(name).to.equal("Example non-fungible token");
    });

    it("should set the correct NFT symbol", async function () {
      const symbol = await instance.symbol();

      expect(symbol).to.equal("eNFT");
    });

    it("should assign the initial balance of the deployer", async function () {
      const balance = await instance.balanceOf(deployer);

      expect(balance.toNumber()).to.equal(0);
    });

    it("should revert when trying to get the balance of the 0x0 address", async function () {
      await truffleAssert.reverts(
        instance.balanceOf(NULL_ADDRESS),
        "ERC721: balance query for the zero address"
      );
    });
  });

  describe("Operation", function () {
    describe("minting", function () {
      it("should emit Transfer event", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const event = response.logs[0].event;
        const from = response.logs[0].args.from;
        const to = response.logs[0].args.to;
        const tokenId = response.logs[0].args.tokenId;
        
        expect(event).to.equal("Transfer");
        expect(from).to.equal(NULL_ADDRESS);
        expect(to).to.equal(user);
        expect(tokenId.toNumber()).to.equal(1);
      });

      it("should mint token to an address", async function () {
        const initialBalance = await instance.balanceOf(user);

        await instance.mintNFT(user, "", { from: deployer });

        const finalBalance = await instance.balanceOf(user);

        expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
      });

      it("should set the expected base URI", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const URI = await instance.tokenURI(tokenId);

        expect(URI).to.equal("acala-evm+-tutorial-nft/" + tokenId);
      });

      it("should set the expected URI", async function () {
        const response = await instance.mintNFT(user, "testToken", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const URI = await instance.tokenURI(tokenId);

        expect(URI).to.equal("acala-evm+-tutorial-nft/testToken");
      });

      it("should allow user to own multiple tokens", async function () {
        const initialBalance = await instance.balanceOf(user);

        await instance.mintNFT(user, "", { from: deployer });
        await instance.mintNFT(user, "", { from: deployer });

        const finalBalance = await instance.balanceOf(user);

        expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(2);
      });

      it("should revert when trying to get an URI of an nonexistent token", async function () {
        await truffleAssert.reverts(
          instance.tokenURI(42),
          "ERC721URIStorage: URI query for nonexistent token"
        );
      });
    });

    describe("balances and ownerships", function () {
      it("should revert when trying to get balance of 0x0 address", async function () {
        await truffleAssert.reverts(
          instance.balanceOf(NULL_ADDRESS),
          "ERC721: balance query for the zero address"
        );
      });

      it("should revert when trying to get the owner of a nonexistent token", async function () {
        await truffleAssert.reverts(
          instance.ownerOf(42),
          "ERC721: owner query for nonexistent token"
        );
      });

      it("should return the token owner", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const owner = await instance.ownerOf(tokenId);

        expect(owner).to.equal(user);
      });
    });

    describe("approvals", function () {
      it("should grant an approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        await instance.approve(deployer, tokenId, { from: user });

        const approved = await instance.getApproved(tokenId);

        expect(approved).to.equal(deployer);
      });

      it("should emit Approval event when granting approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId.toNumber();

        const approval = await instance.approve(deployer, tokenId, { from: user });

        const event = approval.logs[0].event;
        const owner = approval.logs[0].args.owner;
        const approved = approval.logs[0].args.approved;
        const eventTokenId = approval.logs[0].args.tokenId;

        expect(event).to.equal("Approval");
        expect(owner).to.equal(user);
        expect(approved).to.equal(deployer);
        expect(eventTokenId.toNumber()).to.equal(tokenId);
      });

      it("should revert when trying to set token approval to self", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId;

        await truffleAssert.reverts(
          instance.approve(user, tokenId, { from: user }),
          "ERC721: approval to current owner"
        );
      });

      it("should revert when trying to grant approval for a token that is someone else's", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId;

        await truffleAssert.reverts(
          instance.approve(deployer, tokenId, { from: deployer }),
          "ERC721: approve caller is not owner nor approved for all"
        );
      });

      it("should revert when trying to get an approval of a nonexistent token", async function () {
        await truffleAssert.reverts(
          instance.getApproved(42),
          "ERC721: approved query for nonexistent token"
          );
      });

      it("should return 0x0 address as approved for a token for which no approval is given", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });

        const tokenId = response.logs[0].args.tokenId;

        const approved = await instance.getApproved(tokenId.toNumber());

        expect(approved).to.equal(NULL_ADDRESS);
      });

      it("sets approval for all", async function () {
        await instance.setApprovalForAll(deployer, true, { from: user });

        const approved = await instance.isApprovedForAll(user, deployer);

        expect(approved).to.be.true;
      });

      it("revokes approval for all", async function () {
        await instance.setApprovalForAll(deployer, true, { from: user });

        const initiallyApproved = await instance.isApprovedForAll(user, deployer);

        expect(initiallyApproved).to.be.true;

        await instance.setApprovalForAll(deployer, false, { from: user });

        const finallyApproved = await instance.isApprovedForAll(user, deployer);

        expect(finallyApproved).to.be.false;
      });

      it("doesn't reflect operator approval in single token approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.setApprovalForAll(deployer, true, { from: user });

        const approved = await instance.getApproved(tokenId);

        expect(approved).to.equal(NULL_ADDRESS);
      });

      it("should allow operator to grant allowance for a apecific token", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.setApprovalForAll(deployer, true, { from: user });

        const initiallyApproved = await instance.getApproved(tokenId);

        await instance.approve(deployer, tokenId, { from: deployer });

        const finallyApproved = await instance.getApproved(tokenId);

        expect(initiallyApproved).to.equal(NULL_ADDRESS);
        expect(finallyApproved).to.equal(deployer);
      });

      it("should emit Approval event when operator grants approval", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.setApprovalForAll(deployer, true, { from: user });

        const initiallyApproved = await instance.getApproved(tokenId);

        const approval = await instance.approve(deployer, tokenId, { from: deployer });

        const event = approval.logs[0].event;
        const owner = approval.logs[0].args.owner;
        const approved = approval.logs[0].args.approved;
        const eventTokenId = approval.logs[0].args.tokenId;

        expect(event).to.equal("Approval");
        expect(owner).to.equal(user);
        expect(approved).to.equal(deployer);
        expect(eventTokenId.toNumber()).to.equal(tokenId);
      });

      it("should emit ApprovalForAll event when approving for all", async function () {
        const response = await instance.setApprovalForAll(deployer, true, { from: user });

        const event = response.logs[0].event;
        const owner = response.logs[0].args.owner;
        const operator = response.logs[0].args.operator;
        const approved = response.logs[0].args.approved;

        expect(event).to.equal("ApprovalForAll");
        expect(owner).to.equal(user);
        expect(operator).to.equal(deployer);
        expect(approved).to.be.true;
      });

      it("should emit ApprovalForAll event when revoking approval for all", async function () {
        await instance.setApprovalForAll(deployer, true, { from: user });
        const response = await instance.setApprovalForAll(deployer, false, { from: user });

        const event = response.logs[0].event;
        const owner = response.logs[0].args.owner;
        const operator = response.logs[0].args.operator;
        const approved = response.logs[0].args.approved;

        expect(event).to.equal("ApprovalForAll");
        expect(owner).to.equal(user);
        expect(operator).to.equal(deployer);
        expect(approved).to.be.false;
      });
    });

    describe("transfers", function () {
      it("should transfer the token", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();

        const initialBalance = await instance.balanceOf(deployer);

        await instance.transferFrom(user, deployer, tokenId, { from: user });

        const finalBalance = await instance.balanceOf(deployer);

        expect(finalBalance.toNumber() - initialBalance.toNumber()).to.equal(1);
      });
      
      it("should emit Transfer event", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();

        const transfer = await instance.transferFrom(user, deployer, tokenId, { from: user });

        const event = transfer.logs[1].event;
        const from = transfer.logs[1].args.from;
        const to = transfer.logs[1].args.to;
        const responseTokenId = transfer.logs[1].args.tokenId.toNumber();

        expect(event).to.equal("Transfer");
        expect(from).to.equal(user);
        expect(to).to.equal(deployer);
        expect(responseTokenId).to.equal(tokenId);
      });
      
      it("should allow transfer of the tokens if the allowance is given", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.approve(deployer, tokenId, { from: user });

        const initalBalance = await instance.balanceOf(deployer);

        await instance.transferFrom(user, deployer, tokenId, { from: deployer });

        const finalBalance = await instance.balanceOf(deployer);

        expect(finalBalance.toNumber() - initalBalance.toNumber()).to.equal(1);
      });
      
      it("should reset the allowance after the token is transferred", async function () {
        const response = await instance.mintNFT(user, "", { from: deployer });
        const tokenId = response.logs[0].args.tokenId.toNumber();
        await instance.approve(deployer, tokenId, { from: user });

        const initiallyApproved = await instance.getApproved(tokenId);

        await instance.transferFrom(user, deployer, tokenId, { from: user });

        const finallyApproved = await instance.getApproved(tokenId);

        expect(initiallyApproved).to.equal(deployer);
        expect(finallyApproved).to.equal(NULL_ADDRESS);
      });
    });
  });
});
```

</details>

{% hint style="warning" %}
**NOTE: You need to add a deployment script for `NFT` before you can run the tests.**
{% endhint %}

When you run the test with (for example) `yarn test`, your tests should pass with the following output:

```shell
yarn test


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ truffle test
Using network 'development'.


Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.

Deploying NFT
NFT deployed at: 0x9a1803e5A48E72e044AFE833B0aa6dDB3E474429


  Contract: NFT
    Deployment
      ✓ should assert true
      ✓ should set the correct NFT name (42ms)
      ✓ should set the correct NFT symbol (44ms)
      ✓ should assign the initial balance of the deployer (42ms)
      ✓ should revert when trying to get the balance of the 0x0 address (2035ms)
    Operation
      minting
        ✓ should emit Transfer event (888ms)
        ✓ should mint token to an address (947ms)
        ✓ should set the expected base URI (1013ms)
        ✓ should set the expected URI (229ms)
        ✓ should allow user to own multiple tokens (1738ms)
        ✓ should revert when trying to get an URI of an nonexistent token (46ms)
      balances and ownerships
        ✓ should revert when trying to get balance of 0x0 address
        ✓ should revert when trying to get the owner of a nonexistent token (44ms)
        ✓ should return the token owner (1017ms)
      approvals
        ✓ should grant an approval (1245ms)
        ✓ should emit Approval event when granting approval (1074ms)
        ✓ should revert when trying to set token approval to self (1151ms)
        ✓ should revert when trying to grant approval for a token that is someone else's (1063ms)
        ✓ should revert when trying to get an approval of a nonexistent token (46ms)
        ✓ should return 0x0 address as approved for a token for which no approval is given (1015ms)
        ✓ sets approval for all (228ms)
        ✓ revokes approval for all (537ms)
        ✓ doesn't reflect operator approval in single token approval (1210ms)
        ✓ should allow operator to grant allowance for a apecific token (1298ms)
        ✓ should emit Approval event when operator grants approval (1078ms)
        ✓ should emit ApprovalForAll event when approving for all (174ms)
        ✓ should emit ApprovalForAll event when revoking approval for all (329ms)
      transfers
        ✓ should transfer the token (976ms)
        ✓ should emit Transfer event (875ms)
        ✓ should allow transfer of the tokens if the allowance is given (1115ms)
        ✓ should reset the allowance after the token is transferred (1041ms)


  31 passing (26s)

✨  Done in 35.22s.
```

## Deploy script

This deployment script will deploy the contract and output its address.

Within the `x_NFT.js` we will import the `NFT` smart contract and have the blank migration ready. We do this by placing the following code within the file:

```javascript
const NFT = artifacts.require("NFT");

module.exports = async function (deployer) {

};
```

Within the script, we first log the `Deploying NFT` to the console, to signal the start of the deployment, then we deploy it and log its address to the console:

```javascript
  console.log("Deploying NFT");
  
  await deployer.deploy(NFT);

  console.log("NFT deployed at:", NFT.address);
```

<details>

<summary>Your script/x_NFT.js should look like this:</summary>

```javascript
const NFT = artifacts.require("NFT");

module.exports = async function (deployer) {
  console.log("Deploying NFT");
  
  await deployer.deploy(NFT);

  console.log("NFT deployed at:", NFT.address);
};
```

</details>

Running the `yarn deploy` script should return the following output:

```shell
yarn deploy


yarn run v1.22.15
warning ../../../../../package.json: No license field
$ truffle migrate

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      1639425246877
> Block gas limit: 6721975 (0x6691b7)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0xc767735e0d7f220c50507b0338dac1d74708cc44fceb917a8187cb9cd15f5cf6
   > Blocks: 0            Seconds: 0
   > contract address:    0x5f462Fc809693470Ef4E95538cF468d2b777ab33
   > block number:        211
   > block timestamp:     1639438130
   > account:             0xc8BBa1082FE1BFB529d22Db0E5EfCAF8fdB8F93B
   > balance:             99.57694606
   > gas used:            248842 (0x3cc0a)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00497684 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00497684 ETH


1639355398_NFT.js
=================
Deploying NFT

   Deploying 'NFT'
   ---------------
   > transaction hash:    0xf4e2496b61bd9a6e7f4266679b7e9432d0814dd31162ab514dd827115e12a22a
   > Blocks: 0            Seconds: 0
   > contract address:    0xbedf5bDd4e153eE77fC5b37438F913bacAaB18dD
   > block number:        213
   > block timestamp:     1639438136
   > account:             0xc8BBa1082FE1BFB529d22Db0E5EfCAF8fdB8F93B
   > balance:             99.5259023
   > gas used:            2509675 (0x264b6b)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.0501935 ETH

NFT deployed at: 0xbedf5bDd4e153eE77fC5b37438F913bacAaB18dD

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:           0.0501935 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.05517034 ETH


✨  Done in 59.49s.
```

## Summary

We have built upon the previous examples and added an ERC721 smart contract and tested all of its functionalities. The tests were more detailed and covered more examples. We also ensured that we can interact with the smart contract and that its storage is modified as expected. We can compile smart contract with `yarn build`, test it with `yarn test` or `yarn test-mandala` and deploy it with `yarn deploy` or `yarn deploy-mandala`.
