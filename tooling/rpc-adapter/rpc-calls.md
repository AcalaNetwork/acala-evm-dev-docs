# RPC Calls

There are two types of RPCs available:
- [ETH compatible RPCs](#eth-compatible-rpcs), where interface and functionalities match the [JSON-RPC spec](https://eth.wiki/json-rpc/API)
- [Custom RPCs](#custom-rpcs), that doesn't exist in traditional EVM world.

### ETH compatible RPCs
- `web3_clientVersion`
- `net_version`
- `eth_blockNumber`
- `eth_chainId`
- `eth_getTransactionCount`
- `eth_getCode`
- `eth_call`
- `eth_getBalance`
- `eth_getBlockByHash`
- `eth_getBlockByNumber`
- `eth_gasPrice`
- `eth_accounts`
- `eth_getStorageAt`
- `eth_getBlockTransactionCountByHash`
- `eth_getBlockTransactionCountByNumber`
- `eth_sendRawTransaction`
- `eth_estimateGas`
- `eth_getTransactionByHash`
- `eth_getTransactionReceipt`
- `eth_getTransactionByBlockHashAndIndex`
- `eth_getTransactionByBlockNumberAndIndex`
- `eth_getUncleCountByBlockHash`
- `eth_getUncleCountByBlockNumber`
- `eth_getUncleByBlockHashAndIndex`
- `eth_getUncleByBlockNumberAndIndex`
- `eth_getLogs`
- `eth_subscribe`
- `eth_unsubscribe`
- `eth_newFilter`
- `eth_newBlockFilter`
- `eth_getFilterLogs` (doesn't support unfinalized logs yet)
- `eth_getFilterChanges` (doesn't support unfinalized logs yet)
- `eth_uninstallFilter`

### Custom RPCs
- `eth_getEthGas`: calculate eth transaction gas params from substrate gas params. More details please refer [here](https://evmdocs.acala.network/network/gas-parameters)
- `eth_getEthResources`: calculate eth transaction gas params from transaction details, params: [TransactionRequest](https://docs.ethers.io/v5/api/providers/types/#providers-TransactionRequest)
- `net_indexer`: get subql indexer metadata
- `net_cacheInfo`: get the cache info
- `net_isSafeMode`: check if this RPC is running in safe mode
- `net_health`: check the health of the RPC endpoint
- `net_runtimeVersion`: check the current runtime version of the underlying polkadot.js api
- `eth_isBlockFinalized`: check if a block is finalized, params: [BlockTag](https://docs.ethers.io/v5/api/providers/types/#providers-BlockTag)
- `eth_isTransactionFinalized`: check if a transaction is finalized, note that it also returns false for non-exist tx, params: string

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

For more details about the gas params, please refer to the [gas params section](../../network/gas-parameters.md).

{% hint style="info" %}
We can also send the RPC call with [Postman](https://www.postman.com/), so that we can save the calls and reuse them later.
{% endhint %}

