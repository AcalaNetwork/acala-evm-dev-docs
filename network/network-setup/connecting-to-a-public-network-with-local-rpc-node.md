---
description: Instructions on how to setup local RPC node that connects to a public network
---

# Connecting to a public network with local RPC node

Except from connecting to local development network, we can also run a local RPC node connecting to public networks. 

```
npx @acala-network/eth-rpc-adapter \
  --endpoint <node-endpoint-ws-url> \
  --subql <subquery-url>
```

Node endpoints and subquery urls can be found in [network configuration](../network-configuration.md).

For example to connect to Mandala testnet:
```shell
npx @acala-network/eth-rpc-adapter \
  --endpoint wss://mandala-tc9-rpc.aca-staging.network \
  --subql https://subql-query-tc9.aca-staging.network
```

Also checkout the help command for more details
```
npx @acala-network/eth-rpc-adapter --help
```

{% hint style="info" %}
**If you wish to learn more about the RPC adapter, you can find more info in the** [**RPC adapter documentation**](../../tooling/rpc-adapter/running-the-rpc-adapter.md)**.**
{% endhint %}
