---
description: Instructions on how to locally run the RPC adapter
---

# Running the RPC adapter

Running the RPC adapter allows for interaction with the Acala EVM+, be it with the local development network, public test network or either of the main networks.

{% hint style="warning" %}
In previous [local development setup](../../network/network-setup/local-development-network.md) section, we run the whole stack (node + subquery + rpc adapter) with docker compose, which already includes the RPC adapter. This sections is for those who want to run the RPC adapter separately, or learn more about the RPC adapter.
{% endhint %}

## Running a local mandala node
RPC adapter needs to connect to a node, so we first run a local mandala node at port `9944`
```
docker run -it --rm -p 9944:9944 -p 9933:9933 ghcr.io/acalanetwork/mandala-node:sha-89ef1e5 --dev --ws-external --rpc-port=9933 --rpc-external --rpc-cors=all --rpc-methods=unsafe -levm=debug --pruning=archive --instant-sealing
```

## Running the RPC adapter
### with docker
```
docker run -it --rm -p 8545:8545 acala/eth-rpc-adapter:2.8.1 --endpoint ws://host.docker.internal:9944 --local-mode
```

### or via npm
```
npx @acala-network/eth-rpc-adapter \
  --endpoint ws://localhost:9944 \
  --local-mode
```

Flags can be appended to the command in order to adapt the default configuration to your needs. Flags are documented at the [bottom](running-the-rpc-adapter.md#list-of-options) of this page.
## Options

Options can be passed to the RPC adapter in three ways:

1. As CLI options (recommended)
2. As environment variables

### Passing the options as CLI options

Passing the options as CLI options is done by appending the values to the command when running the RPC adapter:

```
npx @acala-network/eth-rpc-adapter -l -e ws://localhost:9944
```

### Passing the options as environment variables

If you wish to pass the options as environment variables in the terminal, you can add them with the `export` statements, or by prepending the command used to spin up RPC adapter:

```
export ENDPOINT_URL=ws://localhost:9944
LOCAL_MODE=1 npx @acala-network/eth-rpc-adapter
```

Both the environment variable that is defined by itself as well as the one defined inline with the `yarn start` will be used in this case.

### List of options

{% hint style="warning" %}
Please don't mix using ENVs and CLI options. **CLI options are preferred**, and will overwrite ENVs.
{% endhint %}

More details can also be found by `npx @acala-network/eth-rpc-adapter --help`.

| ENV                | cli options equivalent | default             | explanation                                                                                             |
|--------------------|------------------------|---------------------|---------------------------------------------------------------------------------------------------------|
| ENDPOINT_URL       | -e, --endpoint         | ws://localhost:9944 | Node websocket endpoint(s): can provide one or more endpoints, seperated by comma url        |
| SUBQL_URL          | --subql                | undefined           | Subquery url: *optional* if testing contracts locally that doesn\'t query logs or historical Tx, otherwise *required* |
| PORT               | -p, --port             | 8545                | port to listen for http and ws requests                                    |
| MAX_CACHE_SIZE     | --max-cache-size       | 200                 | max number of blocks that lives in the cache [more info](https://evmdocs.acala.network/network/network) |
| MAX_BATCH_SIZE     | --max-batch-size       | 50                  | max batch size for RPC request                                                                          |
| STORAGE_CACHE_SIZE | --max-storage-size     | 5000                | max storage cache size                                                                                  |
| SAFE_MODE          | -s, --safe-mode        | 0                   | if enabled, TX and logs can only be found after they are finalized                                      |
| LOCAL_MODE         | -l, --local-mode       | 0                   | enable this mode when testing with locally running instant-sealing mandala                              |
| HTTP_ONLY          | --http-only            | 0                   | only allow http requests, disable ws connections                  |
| VERBOSE            | -v, --verbose          | 1                   | print some extra info                                                                                   |

Using the `-e` flag and passing the URL of a Mandala, Acala and Karura node allows for running the local RPC adapter while communicating with a public network.

### Modes
#### local mode
For local testing, we usually turn this mode on, together with a local `--instant-sealing` mandala node. It has some optimization to run faster with local node, and some minor bug prevention.

#### safe mode (deprecated)
In this mode, Txs and logs can only be found after they are finalized. Now deprecated in favor for the `finalized` and `safe` block tags.
