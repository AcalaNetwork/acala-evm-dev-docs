# RPC Calls

We are continuously expanding the list of supported RPC calls. This is the current list. If you wish to dig deeper, you are welcome to check out our [rpc.json](https://github.com/AcalaNetwork/bodhi.js/blob/master/eth-rpc-adapter/src/rpc.json).

## eth namespace

* `eth_accounts`: Returns a list of addresses owned by client
* `eth_blockNumber`: Returns the number of the most recent block
* `eth_call`: Executes a new message call immediately without creating a transaction on the block chain
* `eth_chainId`: Returns the chain ID of the current network
* `eth_coinbase`: Returns the client coinbase address
* `eth_estimateGas`: Generates and returns an estimate of how much gas is necessary to allow the transaction to complete
* `eth_feeHistory`: Returns data relevant for fee estimation based on the specified range of blocks
* `eth_gasPrice`: Returns the current price per gas in wei
* `eth_getBalance`: Returns the balance of the account of given address
* `eth_getBlockByHash`: Returns information about a block by hash
* `eth_getBlockByNumber`: Returns information about a block by number
* `eth_getBlockTransactionCountByHash`: Returns the number of transactions in a block from a block matching the given block hash
* `eth_getBlockTransactionCountByNumber`: Returns the number of transactions in a block matching the given block number
* `eth_getCode`: Returns code at a given address
* `eth_getFilterChanges`: Polling method for a filter, which returns an array of logs which occurred since last poll
* `eth_getFilterLogs`: Returns an array of all logs matching filter with given id
* `eth_getStorage`: Returns the value from a storage position at a given address
* `eth_getTransactionByBlockHashAndIndex`: Returns information about a transaction by block hash and transaction index position
* `eth_getTransactionByBlockNumberAndIndex`: Returns information about a transaction by block number and transaction index position
* `eth_getTransactionByHash`: Returns the information about a transaction requested by transaction hash
* `eth_getTransactionCount`: Returns the number of transactions sent from an address
* `eth_getTransactionReceipt`: Returns the receipt of a transaction by transaction hash
* `eth_getUncleCountByBlockNumber`: Returns the number of transactions in a block matching the given block number
* `eth_getLogs`: Returns an array of all logs matching filter with given id
* `eth_getWork`**:** Returns the hash of the current block, the seedHash, and the boundary condition to be met (“target”)
* `eth_hashrate`: Returns the number of hashes per second that the node is mining with
* `eth_mining`: Returns whether the client is actively mining new blocks
* `eth_newBlockFilter`: Creates a filter in the node, to notify when a new block arrives
* `eth_newFilter`: Creates a filter object, based on filter options, to notify when the state changes (logs)
* `eth_newPendingTransactionFilter`: Creates a filter in the node, to notify when new pending transactions arrive
* `eth_sendRawTransaction`: Submits a raw transaction
* `eth_sendTransaction`: Signs and submits a transaction
* `eth_sign`: Returns an EIP-191 signature over the provided data
* `eth_signTransaction`: Returns an RLP encoded transaction signed by the specified account
* `eth_submitHashrate`: Used for submitting mining hashrate
* `eth_submitWork`: Used for submitting a proof-of-work solution
* `eth_uninstallFilter`: Uninstalls a filter with given id

## net namespace

* `net_version`: Returns the currently configured chain ID, a value used in replay-protected transaction signing as introduced by EIP-155

## Example of use

This is an example on how to use the `eth_getEthGas` call in order to get the gas parameters for the transaction.

So for Truffle, to get the right gas price and limit. Let's ask the RPC:

```
curl https://eth-rpc-mandala.aca-staging.network \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getEthGas","params": [], "id": 1}'
```

You should get this for an answer:

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "gasPrice": "0x2f955803e9",
    "gasLimit": "0x3293440"
  }
}
```

So in your truffle-config.js you can adjust the gas parameters with the values returned:

```
mandala: {
    provider: () => new HDWalletProvider(mnemonic, "https://eth-rpc-mandala.aca-staging.network"),
    network_id: 595,
    gasPrice: 0x2f955803e9,
    gas: 0x3293440
}, 
```

#### What are the three parameters of `eth_getEthGas`?

The parameters are `[gasLimit, storageLimit, validUntil?]`

* `gasLimit` (optional): substrate gasLimit, usually 21000000
* `storageLimit` (optional): substrate storage limit, usually 64001 for contract deployment
* `validUntil` (optional): valid until block, default to (current block + 100)

#### How is **gasLimit** & **storageLimit** calculated?

{% content-ref url="../../network/gas-parameters.md" %}
[gas-parameters.md](../../network/gas-parameters.md)
{% endcontent-ref %}
