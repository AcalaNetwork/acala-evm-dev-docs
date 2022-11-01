# EVM+ RPC Adapter
Many of the EVM tools rely on [JSON-RPC](https://eth.wiki/json-rpc/API) to communicate with the chain. Acala chain node doesn't provide these RPCs out of the box, so we implemented the EVM+ RPC Adapter, which is a service that wrap substrate RPC calls to provide these ETH JSON-RPCs. As a result, existing Ethereum dApp and tools can interact with EVM+ with minumum changes.
