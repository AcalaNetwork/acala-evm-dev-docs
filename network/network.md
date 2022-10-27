# Network

## Life cycle of a block

Transactions from unfinalised blocks only live in cache, so new transactions will usually be in the cache first (for a couple blocks), then live in both the cache and the subQuery, later, after the cache expires, they only live in subQuery.

Developers who run a mandala node and eth-rpc-adaptor locally, won’t need to start any subQuery or database services for testing, deploying or calling the smart contracts, since new transactions will be in cache and findable without subQuery, as long as there is no need for querying the transactions older than 200 blocks.

![Cache vs. SubQuery representation](../.gitbook/assets/wiki.png)

## Checking if a transaction exists

### In subQuery

```shell
GET https://eth-rpc-mandala.aca-staging.network
{
  "id": 0,
  "jsonrpc": "2.0",
  "method": "eth_getTransactionReceipt",
  "params": ["0x10a36bbfa18cbb2b470a5d301a548feca6279de91013a5bd99e1654976ac013e"]
}

##### or see all of the transactions in the database
POST https://tc7-graphql.aca-dev.network
query {
  transactionReceipts{
    nodes {
      transactionHash
    }
  },
}
```

### In cache

```shell
GET https://eth-rpc-mandala.aca-staging.network
{
  "id": 0,
  "jsonrpc": "2.0",
  "method": "net_cacheInfo",
  "params": []
}
```
