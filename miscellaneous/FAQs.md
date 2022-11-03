# Frequently Asked Questions

## when do I need to provide subquery url for `eth-rpc-adpater` or `EvmRpcProvider`?
**short answer**

you need it when you ...
- need to query for logs
- OR need to query for historical tx receipt that exist **before** you start the provider or rpc adapter
- OR need to query for tx receipt that's older than [MAX_CACHE_SIZE](../tooling/rpc-adapter/running-the-rpc-adapter.md#list-of-options), which defaults to 200.

**explanation**

Transactions from unfinalized blocks only live in cache, so new transactions will usually be in the cache first (for a couple blocks), then live in both the cache and the subquery, later, after the cache expires, they only live in subquery.

Developers who run a mandala node and eth-rpc-adaptor locally, usually won’t need to start any subquery or database services for testing, since new transaction receipt will be in cache and findable without subquery, as long as there is no need for querying the transactions older than 200 blocks.

Also, logs are **NOT** cached (we will probably implement log cache in the future). So if your tests rely on logs, for example if they call `eth_getLogs`, you will need subquery.

![Cache vs. SubQuery representation](../.gitbook/assets/wiki.png)

## why tx failed after I manually changed gas params in metamask?
TODO

## how to fetch the valid gas params in DApp

## why metamask tx doesn't confirm with local mandala?
TODO

## I have balance on metamask, but transfer failed
TODO

## tx failing reason not showing in blockscout
TODO

## how to check if a transaction is finalized?
TODO