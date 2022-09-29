---
description: Instructions on how to run a local development network
---

# Local development network

## Setting up a localhost EVM+ Node

### Prerequisites

The tooling needed in order to successfully run a local EVM+ development network are:

* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Node.js](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
* [yarn](https://classic.yarnpkg.com/lang/en/docs/install/)
* [docker-compose](https://docs.docker.com/compose/install/)&#x20;

{% hint style="info" %}
**NOTE: If you are using a Mac device with Apple silicon, you might need to install** [**rosetta**](https://osxdaily.com/2020/12/04/how-install-rosetta-2-apple-silicon-mac/) **in order to be able to use the required tools.**
{% endhint %}

### Starting the network node

To be able to run the local development network, you need to clone our `bodhi.js` monorepo. Fortunately the network resides within the `evm-subql` directory, so the full project doesn't have to be built. As the monorepo contains submodules, we recommend cloning them as well:

```shell
git clone --recurse-submodules git@github.com:AcalaNetwork/bodhi.js.git
cd bodhi.js
```

{% hint style="info" %}
**NOTE: We suggest regularly running `git pull` and re-running `update` and `build` tasks in order to use the latest version of the local development network.**
{% endhint %}

The local development network consists of Mandala node, Postgres database, SubQuery node and GraphQL engine. The docker compose script that runs and connects them resides within the `evm-subql` directory. To spin them up, use:

```shell
cd evm-subql
yarn
yarn build
docker compose up
```

![Running a local development network](<../../.gitbook/assets/image (12).png>)

{% hint style="info" %}
**NOTE: Use `Control + c` combination of keys to shut down the node.**

**In order to have a clean start after every shutdown of the node, run the following command after the node was shut down:**

**`docker compose down -v`**
{% endhint %}

![Cleaning up the local development network](<../../.gitbook/assets/image (4).png>)

### Starting the RPC node

In addition to the local development network, we need to run an Ethereum RPC adapter, so that the tools used to interact with most of the EVMs work (these tools include, Metamask, Remix, development frameworks, block explorers,..). We run it in another window by using:

```shell
npx @acala-network/eth-rpc-adapter -l 1 --subql http://localhost:3001
```

![Starting up the RPC node](<../../.gitbook/assets/image (15).png>)

{% hint style="info" %}
**If you wish to learn more about the RPC adapter, you can find the info in the** [**RPC adapter documentation**](../../tooling/rpc-adapter/running-the-rpc-adapter.md)**.**
{% endhint %}

### The local development network services

Once the local development network is up and running, the following services are available:

* An ETH RPC node: [http://localhost:8545](http://localhost:8545)
* A subquery web interface: [http://localhost:3001](http://localhost:3001)
* A WS RPC node: [ws://127.0.0.1:9944](ws://127.0.0.1:9944)&#x20;
* Local network Substrate chain explorer: [Polkadot{js} Explorer](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2Flocalhost%3A9944%2Fws#/explorer)

![Local development network in Substrate chain explorer](<../../.gitbook/assets/image (8).png>)

You can now setup [Metamask on localhost](../../tooling/metamask/#localhost) and interact with your local setup.

Try a few of the [EVM tutorials](../../tutorials/tutorials.md) to get familiar with the responses each terminal window provides as feedback.
