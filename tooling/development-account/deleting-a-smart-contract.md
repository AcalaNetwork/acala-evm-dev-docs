# Deleting a smart contract

Deleting a smart contract means that its data is erased from the chain. By doing so, you release a storage used by the smart contract and the storage rent paid for the smart contract is refunded to the address that deleted the smart contract.

To delete a smart contract, you need to be its maintainer. If the maintainer role hasn't been transferred, the deployer of the smart contract automatically assumes this role.

In order to delete the smart contract, you need to open the Polkadot.js chain explorer and open the [Extrinsics](https://polkadot.js.org/apps/#/extrinsics) section.

Within the `submit the following extrinsic` field, select the `evm` option. Select the `selfdestruct` option in the adjacent dropdown menu and paste the address of the smart contract to delete in the input field below it.

All that is left to do is press the `Submit transaction` button, which should open your Polkdadot.js wallet and prompt you to sign the transaction. Once you sign and submit it, you smart contract will be deleted and the storage rent refunded.

![Developer => Extrinsics => evm => selfdestruct](<../../.gitbook/assets/image (38).png>)
