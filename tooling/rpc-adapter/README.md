---
description: What is it and how to use it to debug your project
---

# RPC adapter

Many of the EVM tools rely on [JSON-RPC](https://eth.wiki/json-rpc/API) to communicate with the chain. Acala chain node doesn't provide these RPCs out of the box, so we implemented the EVM+ RPC Adapter, which is a service that wrap substrate RPC calls to provide these ETH JSON-RPCs. As a result, existing Ethereum dApp and tools can interact with EVM+ with minumum changes.

For more information, checkout:
- [running the rpc adapter](./running-the-rpc-adapter.md)
- [github document](https://github.com/AcalaNetwork/bodhi.js/tree/master/packages/eth-rpc-adapter)