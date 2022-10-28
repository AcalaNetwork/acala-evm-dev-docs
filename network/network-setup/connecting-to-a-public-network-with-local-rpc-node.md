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

For example to connect to Mandala testnet:
```shell
npx @acala-network/eth-rpc-adapter \
  --endpoint wss://mandala-rpc.aca-staging.network/ws \
  --subql https://subql-query-mandala.aca-staging.network
```

Subquery urls can be found in [network configuration section](../network-configuration.md), and node urls can be found here(TODO).

Also checkout the help command info for more details
```
npx @acala-network/eth-rpc-adapter --help
```

{% hint style="info" %}
**If you wish to learn more about the RPC adapter, you can find more info in the** [**RPC adapter documentation**](../../tooling/rpc-adapter/running-the-rpc-adapter.md)**.**
{% endhint %}
