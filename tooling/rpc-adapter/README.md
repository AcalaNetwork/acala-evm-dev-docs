---
description: What is it and how to use it to debug your project
---

# RPC adapter

### RPC Endpoints

* `wss://node-6870830370282213376.rz.onfinality.io/ws?apikey=0f273197-e4d5-45e2-b23e-03b015cb7000`
* `wss://acala-mandala-adapter.api.onfinality.io/public-ws`

### EVM RPC

* `https://acala-mandala-adapter.api.onfinality.io/public`

### RPC Health

Is the RPC Indexing service healthy?&#x20;

```
curl https://acala-mandala-adapter.api.onfinality.io/public \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "method": "net_indexer", "params": [], "id": 1}' \
| json_pp
```

### RPC getTransactionReceipt

Has the transaction been indexed in SubQuery? &#x20;

```
curl https://acala-mandala-adapter.api.onfinality.io/public \
  -H "Content-Type: application/json" \
  -d '{"id": 0,"jsonrpc": "2.0","method": "eth_getTransactionReceipt","params": ["0xf8561dee7e6d2128332e8fe177d834467f10ac03c676daefa1507947c685959b"]}' \
  | json_pp
```

### RPC Cache

Hash to Block & Block to Hash

```
curl https://acala-mandala-adapter.api.onfinality.io/public \
  -H "Content-Type: application/json" \
  -d '{"id": 0, "jsonrpc": "2.0", "method": "net_cacheInfo", "params": []}' \
  | json_pp
```



### Web Socket RPC

* `wss://tc7-eth.aca-dev.network/ws`

```
wscat -c wss://tc7-eth.aca-dev.network/ws                                                                                                                    
Connected (press CTRL+C to quit)
> {"jsonrpc":"2.0","method":"eth_getCode","params": ["0x5d8aae4de6ccab1b58c91db8d200aa1577117b2b", "latest"],"id":1}
```
