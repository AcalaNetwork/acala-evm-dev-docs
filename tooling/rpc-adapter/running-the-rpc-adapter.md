---
description: Instructions on how to locally run the RPC adapter
---

# Running the RPC adapter

Running the RPC adapter allows for interaction with the Acala EVM+, be it with the local development network, public test network or either of the main networks.

There are three ways of running the RPC node in a local environment:

1. [Using npm to run the RPC adapter as a package](running-the-rpc-adapter.md#running-the-rpc-adapter-as-a-package)
2. [Building the RPC adapter from source](running-the-rpc-adapter.md#building-the-rpc-adapter-from-source)
3. [Running the RPC adapter as a Docker container](running-the-rpc-adapter.md#undefined)

## Running the RPC adapter as a package

Running the RPC adapter as a package allows you to use the RPC adapter without having to set up anything else. To run it in a this way, use:

```shell
npx @acala-network/eth-rpc-adapter
```

{% hint style="info" %}
**Flags can be appended to the command in order to adapt the default configuration to your needs. Flags are documented at the** [**bottom**](running-the-rpc-adapter.md#undefined) **of this page.**
{% endhint %}

## Building the RPC adapter from source

Building the RPC adapter from source requires you to first clone the [bodhi.js](https://github.com/AcalaNetwork/bodhi.js) repository:

```shell
git clone --recurse-submodules git@github.com:AcalaNetwork/bodhi.js.git
```

Once the repository is cloned install the necessary dependencies as well as build the adapter:

```shell
rush update
rush build -t @acala-network/eth-rpc-adapter
```

{% hint style="warning" %}
**In order to be able to locally build the RPC adapter, you are required to have** [**@microsoft/rush**](https://rushjs.io/pages/intro/get\_started/) **installed.**
{% endhint %}

The RPC adapter is ready to be run. Navigate to the `eth-rpc-adapter` and use:

```shell
yarn start
```

{% hint style="info" %}
**Flags can be appended to the command in order to adapt the default configuration to your needs. Flags are documented at the** [**bottom**](running-the-rpc-adapter.md#undefined) **of this page.**
{% endhint %}

## Running the RPC adapter in Docker

Running the RPC adapter in Docker required Docker to be running and issuing the following command:

```shell
docker run -it --rm -e LOCAL_MODE=1 -p 8545:8545 acala/eth-rpc-adapter:aa2c8d7 yarn start
```

{% hint style="info" %}
**Flags can be appended to the command in order to adapt the default configuration to your needs. Flags are documented at the** [**bottom**](running-the-rpc-adapter.md#undefined) **of this page.**
{% endhint %}

## Options

Options can be passed to the RPC adapter in three ways:

1. As environment variables in the terminal
2. As environment variables in the `.env` file
3. As flags in the terminal

### Passing the options as environment variables in the terminal

If you wish to pass the options as environment variables in the terminal, you can add them with the `export` statements or by prepending the command used to spin up RPC adapter:

```shell
ENDPOINT_URL=ws://localhost:9944
LOCAL_MODE=1 yarn start
```

Both the environment variable that is defined by itself as well as the one defined inline with the `yarn start` will be used in this case.

### Setting the options as a variables in the `.env` file

In case you are building the RPC adapter from source,  you can set the options as variables within the `.env` file. There is already a `.env.sample` available, but the variables are set like this:

```
ENDPOIN_URL=ws://localhost:9944
LOCAL_MODE=1
```

All the options set in this way will be used.

### Passing the options as flags in the terminal

Passing the options as flags in terminal is done by appending the values to the command when running the RPC adapter:

```shell
npx @acala-network/eth-rpc-adapter -l -e we://localhost:9944
```

Running the RPC adapter with these flags is equal to the previous two examples.

### List of options

Following is the list of available options with the default values and explanations of each option.

| Environment variable | Flag format        | Default value       | Explanation                                                                                                                                                                                                                                                |
| -------------------- | ------------------ | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ENDPOINT\_URL        | -e, --endpoint     | ws://localhost:9944 | URL of the node endpoint                                                                                                                                                                                                                                   |
| SUBQL\_URL           | --subql            | undefined           | URL of the SubQuery graphql-service (usually http://localhost:3001)                                                                                                                                                                                        |
| HTTP\_PORT           | -h, --http-port    | 8545                | HTTP port to receive the RPC requests                                                                                                                                                                                                                      |
| WS\_PORT             | -w, --ws-port      | 3331                | WS port to receive the RPC requests                                                                                                                                                                                                                        |
| MAX\_CACHE\_SIZE     | --cache-size       | 200                 | Maximum number of blocks to be cached                                                                                                                                                                                                                      |
| MAX\_BATCH\_SIZE     | --max-batch-size   | 50                  | Maximum size of a batch in a RPC request                                                                                                                                                                                                                   |
| STORAGE\_CACHE\_SIZE | --max-storage-size | 5000                | Maximum size of a storage cache                                                                                                                                                                                                                            |
| SAFE\_MODE           | -s, --safe         | 0                   | When enabled, transaction logs can only be found after they have been finalised                                                                                                                                                                            |
| FORWARD\_MODE        | -f, --forward      | 0                   | When enabled, the Substrate RPC calls will be forwarded to the Substrate node                                                                                                                                                                              |
| LOCA\_MODE           | -l, --local        | 0                   | Enables faster development and testing in a local network by supporting instant sealing and calls only supported in the local  development environment                                                                                                     |
| RICH\_MODE           | -r, --rich         | 0                   | Enables overloaded gas parameters mode. The overloaded gas parameters mode reports the maximum gas parameters for transactions, no matter what they contain. It allows for testing uninhibited by the requirement to set custom deployment gas parameters. |
| VERBOSE              | -v, --verbose      | 1                   | Enables verbose output mode                                                                                                                                                                                                                                |

Using the `-e` flag and passing the URL of a Mandala, Acala and Karura node allows for running the local RPC adapter while communicating with a public network.

{% hint style="danger" %}
Using `-r` flag to speed up the development should be used with the understanding, that deployment transaction gas overrides should be added to the deploy scripts, and that the deploy scripts should be validated in the local development networks prior to attempting to run them on public networks.
{% endhint %}
