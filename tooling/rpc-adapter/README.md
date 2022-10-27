---
description: What is it and how to use it to debug your project
---

# RPC adapter

### RPC Endpoints

* `wss://mandala-rpc.aca-staging.network/ws


### EVM RPC

* `https://eth-rpc-mandala.aca-staging.network`

### RPC Health

Is the RPC Indexing service healthy?&#x20;

```
curl https://eth-rpc-mandala.aca-staging.network \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "method": "net_indexer", "params": [], "id": 1}' \
| json_pp
```

### RPC getTransactionReceipt

Has the transaction been indexed in SubQuery? &#x20;

```
curl https://eth-rpc-mandala.aca-staging.network \
  -H "Content-Type: application/json" \
  -d '{"id": 0,"jsonrpc": "2.0","method": "eth_getTransactionReceipt","params": ["0xf8561dee7e6d2128332e8fe177d834467f10ac03c676daefa1507947c685959b"]}' \
  | json_pp
```

### RPC Cache

Hash to Block & Block to Hash

```
curl https://eth-rpc-mandala.aca-staging.network \
  -H "Content-Type: application/json" \
  -d '{"id": 0, "jsonrpc": "2.0", "method": "net_cacheInfo", "params": []}' \
  | json_pp
```



### Web Socket RPC

* `wss://eth-rpc-mandala.aca-staging.network`

```
wscat -c wss://eth-rpc-mandala.aca-staging.network                                                                                                                   
Connected (press CTRL+C to quit)
> {"jsonrpc":"2.0","method":"eth_getCode","params": ["0x5d8aae4de6ccab1b58c91db8d200aa1577117b2b", "latest"],"id":1}
```
