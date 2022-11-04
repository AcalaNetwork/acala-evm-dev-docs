# Security considerations of developing in Acala EVM+

## Introduction

As Acala EVM+ is not a simple copy-paste clone of the Ethereum's EVM, there are additional security considerations to the ones from Ethereum's EVM. The SWCs still apply and you should familiarise yourself with them if you haven't already. The directory can be found [here](https://swcregistry.io/).

Acala EVM+ interacts with Substrate layer of the chain, so this introduces some of the considerations that we have to be aware of. We also refined some of the settings and mechanics of the chain, so some of these differences may impact your smart contracts. You also need to be aware how storage rent and storage deposit work when building, deploying and maintaining your smart contracts. There is more and we will analyse as much as we can in this article.

This article assumes you have familiarised yourself with the Acala EVM+ and its functions.

## Considerations

While we will to outline as much of the security considerations as we can, there is still a possibility that additional ones exist. We will expand the list if that is the case.

### 1. Double entry consideration

Our native tokens ACA and KAR have a mirrored [predeployed smart contract](../network/precompiled-and-predeployed-smart-contracts/), which means that a change in balance of the native currency is also reflected in the mirrored ERC20 predeployed smart contract and vice versa. A DeFi protocol that supports operation of the native currency as well as the predeployed ERC20 smart contracts may encounter unexpected or unreliable behaviour because of this.

We suggest to disable operations with native currency and only support the mirrored ERC20 version.

### 2. Dust balance consideration

All of our mirror tokens, as well as the native tokens, have [existential deposit](https://wiki.acala.network/get-started/acala-network/acala-account#existential-deposit). This means that the balance of an account that falls below the threshold will be 'dusted'. This preserves space of chaindata and ensures that there are no accounts with balances lower than the cost of transferring these balances; if an account has balance lower than the transaction cost needed to clear the balance, the record of this balance can never be cleared.

Not accounting for existential deposit can lead to mismatched numbers in your own protocol records or to transactions failing unexpectedly.

We suggest implementing a mechanic for verification of these tokens being successfully received and not dusted or to enforce a minimum value of tokens being transferred to ensure that the tokens are not dusted upon receipt when transferring the mirror tokens.

### 3. Storage deposit consideration

[Storage deposit](about-acala-evm+.md#renting-storage) is used to protect against chain bloat. It represents a staked amount of tokens that are returned upon clearing of storage. Having unbounded storage write ability could potentially allow for users causing a state of smart contact to require a change so big, that the interaction with the smart contract is too expensive.

We suggest ensuring that the users are unable to put the smart contract in a state that could lead to so many states having to be changed at once, that the smart contract becomes unusable.

On top of that, removing storage entries or destructing smart contracts will refund storage deposits. Smart contracts that allow potentially large storage removals, may also need to be gated, to ensure the storage deposit refund can be handled appropriately.

Additionally we suggest that you design your smart contracts to change as little states as possible as this will reduce the cost of storage deposit incurred on users.

### 4. Unexpected upgrades consideration

Acala EVM+ supports [publication](../tooling/development-account/publishing-a-smart-contract.md#mark-a-given-contract-as-published-in-the-developer-section-of-the-polkadot-app) of smart contracts. This allows the team deploying the smart contract to verify, test and configure the smart contract before the non-developer users are able to interact with it. This also means that the unpublished smart contract can be upgraded even if it is not upgradeable in a smart contract upgradeable sense (e.g. written in a proxy-upgrade or diamond pattern).

We suggest [verifying](../tooling/development-account/publishing-a-smart-contract.md#verify-that-the-smart-contract-has-been-published-successfully) that the smart contract, your smart contract is interacting with, is published before interacting with it.

### 5. Decimals consideration

Both ACA and KAR have 12 decimal points natively, however the native token decimals in EVM is always 18. This means that if you transfer `1000000 wei` in your smart contract, you are transferring the lowest denomination of ACA or KAR. Transfer of `1 ether` will result transfer of 1 ACA or KAR. Additionally, in order to avoid unexpected transfer amounts, any transfer with value that cannot be exactly represented with 12 decimals will fail. This means any transfer of value that is not a multiple of `10^6 wei` will be reverted. If you rely on such calls for transferring the native currency in your smart contract, your calls might potentially fail.

We suggest reviewing your code for the `transfer()` calls and making sure you are transferring the expected amounts. The contracts needs to gracefully handle transfer amounts that are either result of user input or arithmetic operations, because the value may not be a valid transferable amount.

### 6. Two management accounts consideration

EVM and the Substrate accounts [bound](../tooling/development-account/#bind-accounts) to each other in more that just sharing the balances of the native tokens. They are used to manage the deployed smart contracts. The EVM account can manage a smart contract using the owned smart contract development pattern while the Substrate account can [publish](../tooling/development-account/publishing-a-smart-contract.md) or [delete](../tooling/development-account/deleting-a-smart-contract.md) the smart contract.

We suggest that, in case you are transfer the ownership of your smart contract in the EVM to another account, you transfer the management of the smart contract to the corresponding substrate account as well.

{% hint style="info" %}
**NOTE: This only applies to unpublished contracts.**
{% endhint %}

### 7. Duplicated transaction hashes consideration

There is no possibility for two or more transactions to share a same transaction hash. In Substrate based chains this is possible due to non-zero existential deposit. Once the account is [_dusted_](security-considerations-of-developing-in-acala-evm+.md#2.-dust-balance-consideration), its nonce is reset to 0. This means that people can craft replayable transactions, which could have the same transaction hash.

We suggest that, if your dApp processes the transaction hashes, make sure that it can handle the replayed transactions gracefully.

### 8. Unintended address binding consideration

The intertwining of the Substrate level and the EVM level means that, for users to access the full functionality of Acala's networks, their Substrate and EVM accounts need to be bound. Some of the Substrate assets might return an EVM asset to the account. When this happens the EVM assets are allocated to the EVM account associated to the Substrate account. If no such binding exists, the default EVM account will be bound to it.

{% hint style="info" %}
The default EVM account can only be used to [interact with the EVM through Substrate calls](../special-features/using-bodhi.js-to-deploy-smart-contract-and-interact-with-it.md) and cannot be imported into an EVM wallet as no private key or mnemonic to generate it from is available.
{% endhint %}

The assets associated with the default EVM address are fully interactable with, but can't be managed by the EVM wallet. This means that unlocking their full functionality might require user to transfer them, to a wallet they are able to restore, one by one.

We suggest you implement a check in your dApp to verify that the user has bound their own EVM address and if they didn't, direct them to instructions on how to do that. As any smart contract interaction from a Substrate account binds the default EVM account, make sure to implement this check even if no EVM assets are allocated and the user is just interacting with a smart contract.

## Conclusion

While the preexisting security considerations of developing in the EVM still apply, Acala EVM+'s improvements also introduce new ones. The list of security considerations in this article should give you a good overview on what to expect.

If you think a security consideration could be explained into more depth or if you have questions about other potential security considerations, not mentioned in this article, we encourage you to get in touch over [Discord](https://www.acala.gg/) or any other communication channel.
