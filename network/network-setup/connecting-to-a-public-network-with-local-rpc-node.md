---
description: Instructions on how to setup local RPC node that connects to a public network
---

# Connecting to a public network with local RPC node

Except from connecting to local development network, we can also run a local RPC node connecting to public networks.

```
npx @acala-network/eth-rpc-adapter@latest \
  --endpoint <node-endpoint-ws-url> \
  --subql <subquery-url>
```

Node endpoints and subquery urls can be found in [network configuration](../network-configuration.md).

For example to connect to acala mainnet:
```
npx @acala-network/eth-rpc-adapter@latest \
  --endpoint wss://acala-rpc.aca-api.network \
  --subql https://subql-query-acala.aca-api.network
```

Also checkout the help command for more details
```
npx @acala-network/eth-rpc-adapter@latest --help
```

{% hint style="info" %}
**If you wish to learn more about the RPC adapter, you can find more info in the** [**RPC adapter documentation**](../../tooling/rpc-adapter/running-the-rpc-adapter.md)**.**
{% endhint %}
