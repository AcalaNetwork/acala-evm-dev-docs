---
description: Instructions on how to setup local RPC node that connects to a public network
---

# Connecting to a public network with local RPC node

Running a local RPC node allows you for a greater reliability and availability. To be able to run it, you need to install the `eth-rpc-adapter` dependency:

```shell
yarn global add @acala-network/eth-rpc-adapter
```

Running the local RPC node requires the `ENDPOINT_URL` and `SUBQL_URL` values to be passed before startup:

```shell
eth-rpc-adapter -e [URL] --suql [URL]
```

Any of the values listed in the [Network configuration section](../network-configuration.md) is valid. Select the network that you want to connect to and fill in the required values.

When your local RPC node is running, you can pass it as a network endpoint with the URL: `http://localhost:8545`.The node can now be used for deployment and interaction with the smart contracts as well as your gateway to interact with the network with a wallet like [Metamask](../../tooling/metamask/).

{% hint style="info" %}
**If you wish to learn more about the RPC adapter, you can find more info in the** [**RPC adapter documentation**](../../tooling/rpc-adapter/running-the-rpc-adapter.md)**.**
{% endhint %}
