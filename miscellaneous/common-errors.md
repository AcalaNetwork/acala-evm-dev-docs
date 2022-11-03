# Common Errors
This page summarises common errors that you might encounter while developing on Acala EVM+. If an error occurs that is not listed here, please reach out, so we might lend a hand and include it on this page.

## ProviderError: Error: -32603: execution fatal
### `{ index: 180, error: 0, message: None }`

**Error name:** AddressNotMapped

**Error explanation:** This means that maintenance features are called on a smart contract, that doesn't have an EVM address bound to it

**Common causes:**

* Should not happen, because even if no user account is associated with the smart contract, it should bind to the `0x0` address

**Suggested actions:**

* We suggest reaching out to us, so we can help you investigate the issue

### `{ index: 180, error: 1, message: None }`

**Error name:** ContractNotFound

**Error explanation:** This means that maintenance actions are being preformed on an address that is not a smart contract

**Common causes:**

* Usually happens when trying to maintain a non-existent or already deleted smart contract

**Suggested actions:**

* We suggest reviewing the address you are trying to maintain



### `{ index: 180, error: 2, message: None }`

**Error name:** NoPermission

**Error explanation:** This means that the account is not allowed to interact with the smart contract

**Common causes:**

* This means that the account is not allowed to interact with the smart contract

**Suggested actions:**

* If the user wants to interact with the smart contract before it is published, they can turn on the development mode
* It the smart contract maintainer wishes to enable non-developer users to interact with the smart contract, they can publish the smart contract



### `{ index: 180, error: 3, message: None }`

**Error name:** ContractDevelopmentNotEnabled

**Error explanation:** This means that the account doesn't have the development mode enabled

**Common causes:**

* Usually happens when account doesn't have development mode enabled and the user tries to disable the development mode

**Suggested actions:**

* If you wish for the development mode to be disabled, you don't need to do anything
* If you wish to enable the development mode, you can enable it by following the documentation



### `{ index: 180, error: 4, message: None }`

**Error name:** ContractDevelopmentAlreadyEnabled

**Error explanation:** This means that the account already has the development mode enabled

**Common causes:**

* Usually happens when account has development mode enabled and the user tries to enable the development mode

**Suggested actions:**

* If you wish for the development mode to be enabled, you don't need to do anything
* If you wish to disable the development mode, you can enable it by following the documentation



### `{ index: 180, error: 5, message: None }`

**Error name:** ContractAlreadyPublished

**Error explanation:** This means that the smart contract is already published

**Common causes:**

* Usually happens when trying to publish an already published smart contract

**Suggested actions:**

* If you wish for the smart contract to be published, you don't need to do anything
* If you wish that the smart contract wouldn't be reachable by users that don't have development mode enabled, you can delete it and redeploy it



### `{ index: 180, error: 6, message: None })&#x20`

**Error name:** ContractExceedsMaxCodeSize

**Error explanation:** This means that the smart contract file size is too big

**Common causes:**

* Usually happens when trying to deploy a smart contract that is too big

**Suggested actions:**

* We suggest refactoring your smart contract, so that the file size is decreased



### `{ index: 180, error: 7, message: None }`

**Error name:** ContractAlreadyExisted

**Error explanation:** This means that the same address has already been used and can't be used again

**Common causes:**

* Usually happens when the substrate account, to which the EVM+ account has been bound, has been reaped, which resulted in the EVM+ account nonce being reset. This in turn can cause the EVM+ to try and create a smart contract at an address that has already been used

**Suggested actions:**

* We suggest using another EVM+ account

{% hint style="warning" %}
**NOTE: This behaviour should be made obsolete. If you encounter this error, please reach out to us, so we can investigate.**
{% endhint %}



### `{ index: 180, error: 8, message: None }`

**Error name:** OutOfStorage

**Error explanation:** This means that the storage usage of the transaction is greater than the storage limit

**Common causes:**

* Usually happens when transaction changes a number of states and the storage limit value is too low

**Suggested actions:**

* We suggest increasing the storage limit value of the transaction
* If the storage limit exceeds the maximum storage limit we suggest reviewing the call initiated by the transaction and adapting it to change less states



### `{ index: 180, error: 9, message: None }`

**Error name:** ChargeFeeFailed

**Error explanation:** This means that the smart contract, that is trying to use Schedule predeployed smart contract, doesn't have enough funds to pay for the Schedule transaction fees

**Common causes:**

* Usually happens when either the smart contract has too low of a balance or the scheduled call is too complex

**Suggested actions:**

* We suggest additionally funding the smart contract or reviewing the scheduled call in order to make it more efficient



### `{ index: 180, error: 10, message: None }`

**Error name:** CannotKillContract

**Error explanation:** This means that killing the smart contract has failed

**Common causes:**

* Usually happens when trying to kill a non-existent function

**Suggested actions:**

* We suggest reviewing the address you are using to kill the smart contract and checking for typos or missed characters



### `{ index: 180, error: 11, message: None }`

**Error name:** ReserveStorageFailed

**Error explanation:** This means that the account doesn't have enough funds to put data into storage

**Common causes:**

* Usually happens when trying to change a lot of states in a single transaction

**Suggested actions:**

* We suggest reviewing the call and reducing the number of states that are changed within it
* If the number of states changed by the call can't be reduced, we suggest adding funds to the account



### `{ index: 180, error: 12, message: None }`

**Error name:** UnreserveStorageFailed

**Error explanation:** This means that releasing the storage has failed

**Common causes:**

* Usually happens when the storage rent is increased after the storage has been rented, but not before it has been released

**Suggested actions:**

* We suggest reaching out to the team as this shouldn't happen under normal circumstances



### `{ index: 180, error: 13, message: None }`

**Error name:** ChargeStorageFailed

**Error explanation:** This means that charging the storage rent has failed

**Common causes:**

* None

**Suggested actions:**

* We suggest reaching out to the team if you encounter this error



### `{ index: 180, error: 14, message: None }`

**Error name:** InvalidDecimals

**Error explanation:** This means that the value provided was too low

**Common causes:**

* Usually happens when trying to convert wei to ACA or KAR. As ACA and KAR have 12 decimal spaces and the EVM+ expects the native currency to have 18, this error might occur during the conversion.

**Suggested actions:**

* We suggest only using the values greater than `1_000_000` when referring to a native currency in wei

## Named Errors
### `Error: 1010: Invalid transaction: Transaction is outdated`

**Error name:** Transaction is outdated

**Error explanation:** This means that the transaction's `validUntil` value is too low or that there is already a transaction with the same nonce in the chain.

**Common causes:**

* `validUntil` value of the transaction is lower than current block number
* Transaction nonce is the same as one of the preexisting transactions had

**Suggested actions:**

* Verify that the transaction has a valid `validUntil` value and update it if the block number of the chain is higher
* Reset the account nonce, to make sure it corresponds to the one associated with the account nonce on chain

### `Error: 1012: Invalid transaction: Transaction is temporary banned`

**Error name:** Transaction is temporary banned

**Error explanation:** This means that an identic transaction to this one has already failed, so mining this transaction won't be attempted. This behaviour is narrated by Substrate and has to be supported.

**Common causes:**

* Transaction identic to this one has recently failed

**Suggested actions:**

* Review logs and identify the original error to address it
* If the previous error can't be found, wait for 15 minutes and re-attempt sending the transaction. The original error message should be returned

## Other Errors
### `Transaction hash mismatch from Provider.sendTransaction`
**Common causes:**

This is usually cause by using `ethers.JsonRpcProvider` as provider when sending a transaction. 

**Suggested actions:**

Substitute `JsonRpcProvider` by bodhi.js sdk's [evmRpcProvider](https://github.com/AcalaNetwork/bodhi.js/tree/master/eth-providers#getting-started). For example:

```ts
const provider = new EvmRpcProvider('chain node ws url');
await provider.isReady();
const signer = new ethers.Wallet('private key', provider);

// use the signer to send transaction ...
await provider.disconnect();
```

Note that `JsonRpcProvider` should still work in most cases, except when sending a transaction.