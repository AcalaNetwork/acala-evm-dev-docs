# Gas parameters

The main difference between Acala EVM+ and legacy EVM is that we need to use pre-computed `gasPrice` and `gasLimit` (we will refer them as "gas parameters"), and manually inputting random gas parameters is discouraged.

<details>

<summary>If you just need a quick help with the gas parameters and are not interested in the explanation, you can expand this section.</summary>

Most of the gas parameter issues that you might encounter, can be solved in two steps:

1. Get valid gas parameters:

```shell
curl --location --request GET 'https://acala-mandala-adapter.api.onfinality.io/public' \
--header 'Content-Type: application/json' \
--data-raw '{
    "jsonrpc": "2.0",
    "method": "eth_getEthGas",
    "params": [],
    "id": 1
}'
```

2\. Override your transaction parameters with the ones returned as a result of this call.

</details>

## Context

As Acala EVM+ is running on a substrate chain, gas parameters need to encode three substrate parameters: `gasLimit`, `storageLimit`, and `validUntil`. We need to provide a delicate ratio of `gasPrice` to `gasLimit`, so that they can be decoded correctly into the substrate parameters. Randomly changing just one of the gas parameters likely to results in a bad parameter decoding.

For example, when user sends token with MetaMask, they will get the gas parameter values of  `201.996894218 gwei` for `gasPrice`, and a `gasLimit` of `342312`. After the user sends this transaction, these gas parameters will be decoded to `validUntil` value of `914096`, `gasLimit` value of `21000` and `storageLimit` value of `641`.

Although this part is incompatible with the legacy EVM, it is actually an advantage of the Acala EVM+, which is able to utilise some features that cannot be found in the legacy EVM. By using `validUntil` parameter we can avoid transaction being stuck in the transaction pool indefinitely, and by using `storageLimit` we are able to encourage the developers to remove the data they don't need from the chain, in order to reduce the chain bloat.

## Getting the Gas parameters

### For users

Users don't have to compute gas parameters by themselves:

* When sending tokens, MetaMask will call the RPC endpoint to get the correct gas parameters for them automatically.
* When signing a transaction, dApps should compute the correct gas parameters and pass these values to the MetaMask, so users just sign a valid transaction.

The only thing that need to pay attention to, is to not modify the transaction parameters within MetaMask, otherwise the transaction will fail due to bad decoding. We thus encourage you to add the warning against such actions within the user interface of your dApps.

### For developers

**Developers need to compute correct gas parameters when deploying contracts**.

This is because the default gas parameters computed by the tools (MetaMask, Truffle, Hardhat, etc..) have a `storageLimit` of `641`, which is not a limit big enough to store a contract.

On the other hand, smart contract calls won't need this step in most cases, since the default parameters are sufficient for most of the smart contract calls. In some cases if a smart contract call uses an amount of storage that is greater than the default `storageLimit`, valid gas parameters have to be computed.

There are three ways to get valid gas parameters for a transaction:

* Using an RPC call
* Using hardcoded values
* Using an SDK helper

#### **Using an RPC call**

Getting valid gas parameters is as easy as initiating the following call:

```shell
curl --location --request GET 'https://acala-mandala-adapter.api.onfinality.io/public' \
--header 'Content-Type: application/json' \
--data-raw '{
    "jsonrpc": "2.0",
    "method": "eth_getEthGas",
    "params": [],
    "id": 1
}'
```

The following call will return the same result as using default substrate parameters:

```shell
curl --location --request GET 'https://acala-mandala-adapter.api.onfinality.io/public' \
--header 'Content-Type: application/json' \
--data-raw '{
    "jsonrpc": "2.0",
    "method": "eth_getEthGas",
    "params": [
      {
        "gasLimit": 21000000,
        "storageLimit": 64100,
        "validUntil": current block + 100
      }
    ],
    "id": 1
}'
```

Custom substrate parameters can be passed as well. For example if we want to get the gas parameters for a transaction that should be valid until block `10000000`:

```shell
curl --location --request GET 'https://acala-mandala-adapter.api.onfinality.io/public' \
--header 'Content-Type: application/json' \
--data-raw '{
    "jsonrpc": "2.0",
    "method": "eth_getEthGas",
    "params": [
        {
            "gasLimit": 21000000,
            "storageLimit": 64100,
            "validUntil": 10000000
        }
    ],
    "id": 1
}'
```

This is the result, with is the valid `gasPrice` and `gasLimit` for transactions to use:

```json5
{
  "id": 1,
  "jsonrpc": "2.0",
  "result":{
    "gasPrice": "0x33a70303ea",
    "gasLimit": "0x329b140"
  }
}
```

#### **Using hardcoded values**

Let's use the successfully computed a gas parameters from the previous example:

```json5
"result":{
  "gasPrice": "0x33a70303ea",
  "gasLimit": "0x329b140"
}
```

these parameters are valid until block number `10000000` and can be reused until this block number is reached.

#### **Using an SDK helper**

A thorough explanation about this method can be found in the [tutorials](broken-reference). This is the snippet of how the sample code that returns the transaction parameters looks like:

```typescript
import { calcEthereumTransactionParams } from '@acala-network/eth-providers';
import { ApiPromise, WsProvider } from '@polkadot/api';

const MANDALA_NODE_URL = 'https://acala-mandala-adapter.api.onfinality.io/public';
const wsProvider = new WsProvider(MANDALA_NODE_URL);
const api = await ApiPromise.create({ provider: wsProvider });

const storageByteDeposit = (api.consts.evm.storageDepositPerByte).toString();
const txFeePerGas = (api.consts.evm.txFeePerGas).toString();
const blockNumber = (await api.rpc.chain.getHeader()).number.toNumber();
// const blockNumber = 1000000;     // hard coded option

const { txGasPrice, txGasLimit } = calcEthereumTransactionParams({
  gasLimit: 1000010,
  storageLimit: 640010,
  validUntil: blockNumber + 100,
  txFeePerGas,
  storageByteDeposit,
});

console.log({
  txGasPrice: txGasPrice.toNumber(),
  txGasLimit: txGasLimit.toNumber(),
});
```

## Overriding transaction Gas parameters

After getting the valid gas parameters, we can override the smart contract deployment transaction gas parameters so that the smart contract gets successfully deployed.

### Hardhat

Transaction parameters need to be overriden for each transaction:

```typescript
- const instance = await HelloWorld.deploy(callParams)
+ const instance = await HelloWorld.deploy(callParams, gasOverrides)
```

### Truffle

Gas overrides can be specified globally in `truffle-config.js`

```javascript
{
  ...
  gasPrice: xxx,
  gas: yyy,
}
```

### MetaMask

The details on how to use the modified gas parameters can be found [here](../tooling/metamask/).

### Remix IDE

The details on how to use the modified gas parameters can be found [here](../tooling/remix-ide/).

## Limitations and Further Improvements

Since users are unable to change the gas parameters, we currently don't support speeding up the transactions. In the future we might be able to use EIP-1559 or a new algorithm to support the Substrate `tip`.

## Gas price calculation

If you are interested exactly how the gas price is calculated, you are welcome to expand the following section:

<details>

<summary>Gas parameters calculation equation</summary>

* User Inputs
  * `gas_limit`
  * `storage_byte_limit`
  * `valid_until`
    * Block number

<!---->

* Tx data
  * `tx_gas_price`
  * `tx_gas_limit`

<!---->

* User input to tx data
  * `block_period = valid_until / 30`
    * `storage_entry_limit = storage_byte_limit / 64`
      * storage\_count\_limit is u16
      * max value 0xffff = 4194240 bytes = 4MB
  * `storage_byte_deposit = 100000000000000`
  * `storage_entry_deposit = storage_byte_deposit * 64`
  * `tx_fee_per_gas = 200000000000`
  * `tx_gas_price = tx_fee_per_gas + block_period << 16 + storage_entry_limit`
  * `tx_gas_limit = gas_limit + storage_entry_limit * storage_entry_deposit / tx_fee_per_gas`

<!---->

* Tx data to user input
  * `storage_entry_limit = tx_gas_price | 0xffff`
  * `block_period = (tx_gas_price - stroage_entry_limit - tx_fee_per_gas) >> 16`
  * `valid_until = block_period * 30`
  * `gas_limit = tx_gas_limit - storage_entry_limit * storage_entry_deposit / tx_fee_per_gas`

</details>
