# RPC Calls

There are two types of RPCs available:
- [ETH compatible RPCs](#eth-compatible-rpcs), where interface and functionalities match the [JSON-RPC spec](https://eth.wiki/json-rpc/API)
- [Custom RPCs](#custom-rpcs), which are EVM+ specific RPCs that don't exist in traditional EVM world.

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
- `eth_getEthResources`: calculate eth transaction gas params from transaction details, params: [TransactionRequest](https://docs.ethers.io/v5/api/providers/types/#providers-TransactionRequest)
- `net_indexer`: get subql indexer metadata
- `net_cacheInfo`: get the cache info
- `net_isSafeMode`: check if this RPC is running in safe mode
- `net_health`: check the health of the RPC endpoint
- `net_runtimeVersion`: check the current runtime version of the underlying polkadot.js api
- `eth_isBlockFinalized`: check if a block is finalized, params: [BlockTag](https://docs.ethers.io/v5/api/providers/types/#providers-BlockTag)
- `eth_isTransactionFinalized`: check if a transaction is finalized, note that it also returns false for non-exist tx, params: string
