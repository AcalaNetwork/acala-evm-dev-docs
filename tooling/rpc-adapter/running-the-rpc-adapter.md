---
description: Instructions on how to locally run the RPC adapter
---

# Running the RPC adapter

Running the RPC adapter allows for interaction with the Acala EVM+, be it with the local development network, public test network or either of the main networks.

There are three ways of running the RPC node in a local environment:

1. [Using npm to run the RPC adapter as a package](running-the-rpc-adapter.md#running-the-rpc-adapter-as-a-package)
2. [Building the RPC adapter from source](running-the-rpc-adapter.md#building-the-rpc-adapter-from-source)
3. [Running the RPC adapter as a Docker container](running-the-rpc-adapter.md#running-the-rpc-adapter-in-docker)

## Running the RPC adapter as a package

{% hint style="warning" %}
Instructions below assumes a local mandala node is already running at `ws://localhost:9944`. ([how?](../../network/network-setup/local-development-network.md))
{% endhint %}

Running the RPC adapter as a package allows you to use the RPC adapter without having to set up anything else. To run it in a this way, use:

```shell
npx @acala-network/eth-rpc-adapter \
  --endpoint ws://localhost:9944 \
  --local-mode
```

Flags can be appended to the command in order to adapt the default configuration to your needs. Flags are documented at the [bottom](running-the-rpc-adapter.md#list-of-options) of this page.

## Building the RPC adapter from source

Building the RPC adapter from source requires you to first clone the [bodhi.js](https://github.com/AcalaNetwork/bodhi.js) repository:

```shell
git clone --recurse-submodules git@github.com:AcalaNetwork/bodhi.js.git
```

Once the repository is cloned install the necessary dependencies as well as build the adapter:

{% hint style="warning" %}
In order to be able to locally build the RPC adapter, you are required to have [@microsoft/rush](https://rushjs.io/pages/intro/get\_started/) installed.
{% endhint %}

```shell
rush update
rush build -t @acala-network/eth-rpc-adapter
```
The RPC adapter is ready run:

```shell
cd eth-rpc-adapter
yarn start --local-mode [--other-options]
```

## Running the RPC adapter in Docker

Running the RPC adapter in Docker required Docker to be running and issuing the following command:

```shell
docker run -it --rm -e LOCAL_MODE=1 -p 8545:8545 acala/eth-rpc-adapter:v2.5.3 yarn start
```

## Options

Options can be passed to the RPC adapter in three ways:

1. As CLI options (recommended)
2. As environment variables
3. As environment variables in the `.env` file

### Passing the options as CLI options

Passing the options as CLI options is done by appending the values to the command when running the RPC adapter:

```shell
npx @acala-network/eth-rpc-adapter -l -e ws://localhost:9944
```

### Passing the options as environment variables

If you wish to pass the options as environment variables in the terminal, you can add them with the `export` statements, or by prepending the command used to spin up RPC adapter:

```shell
export ENDPOINT_URL=ws://localhost:9944
LOCAL_MODE=1 yarn start
```

Both the environment variable that is defined by itself as well as the one defined inline with the `yarn start` will be used in this case.

### Setting the options as variables in the `.env` file

In case you are building the RPC adapter from source, you can set the options as variables within the `.env` file, like this

```
ENDPOIN_URL=ws://localhost:9944
LOCAL_MODE=1
```

### List of options

{% hint style="warning" %}
Please don't mix using ENVs and CLI options. **CLI options are preferred**, and will overwrite ENVs.
{% endhint %}

More details can also be found by `yarn start --help` or `npx @acala-network/eth-rpc-adapter --help`.

| ENV                | CLI options equivalent | default             | explanation                                                                                             |
|--------------------|------------------------|---------------------|---------------------------------------------------------------------------------------------------------|
| ENDPOINT_URL       | -e, --endpoint         | ws://localhost:9944 | Node websocket endpoint(s): can provide one or more endpoints, seperated by comma url        |
| SUBQL_URL          | --subql                | undefined           | Subquery url: *optional* if testing contracts locally that doesn\'t query logs or historical Tx, otherwise *required*. [more info](../../miscellaneous/FAQs.md#when-do-i-need-to-provide-subquery-url-for-eth-rpc-adpater-or-evmrpcprovider) |
| PORT               | -p, --port             | 8545                | port to listen for http and ws requests                                    |
| MAX_CACHE_SIZE     | --max-cache-size       | 200                 | max number of blocks that lives in the cache [more info](https://evmdocs.acala.network/network/network) |
| MAX_BATCH_SIZE     | --max-batch-size       | 50                  | max batch size for RPC request                                                                          |
| STORAGE_CACHE_SIZE | --max-storage-size     | 5000                | max storage cache size                                                                                  |
| SAFE_MODE          | -s, --safe-mode        | 0                   | if enabled, TX and logs can only be found after they are finalized                                      |
| LOCAL_MODE         | -l, --local-mode       | 0                   | enable this mode when testing with locally running instant-sealing mandala                              |
| RICH_MODE          | -r, --rich-mode        | 0                   | if enabled, default gas params is big enough for most contract deployment and calls, so contract tests from traditional evm world can run unchanged. Note this mode is helpful for testing contracts, but is different than production envionment. [more info](https://evmdocs.acala.network/network/gas-parameters) |
| HTTP_ONLY          | --http-only            | 0                   | only allow http requests, disable ws connections                  |
| VERBOSE            | -v, --verbose          | 1                   | print some extra info                                                                                   |

Using the `-e` flag and passing the URL of a Mandala, Acala and Karura node allows for running the local RPC adapter while communicating with a public network.

### Modes
#### safe mode (deprecated)
In this mode, Txs and logs can only be found after they are finalized. Now deprecated in favor for the `finalized` and `safe` block tags.

#### local mode
For local testing, we usually turn this mode on, together with a local `--instant-sealing` mandala node. It has some optimization to run faster with local node, and some minor bug prevention.

#### rich mode
We usually need to specify [gas params](https://evmdocs.acala.network/network/gas-parameters) for some transactions, this is sometimes time-consuming for contract testing, especially with `hardhat` where we can't override gas params in config. 

In rich mode, default gas params are much bigger than normal, so we don't need to worry about overriding gas params when running tests. As a result, all truffle/hardhat tests can be run **unchanged**. 

{% hint style="danger" %}
Rich mode is convenient for contract testing, but we still recommend reading through the [gas params](https://evmdocs.acala.network/network/gas-parameters) and understand how gas works in EVM+, since in prod we might still need to override gas params.
{% endhint %}
