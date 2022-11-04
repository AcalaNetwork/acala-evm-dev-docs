---
description: Instructions on how to run a local development network
---

# Local development network

## Setting up a local EVM+ Stack

### Prerequisites

The tooling needed in order to successfully run a local EVM+ development network are:

* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Node.js](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
* [yarn](https://classic.yarnpkg.com/lang/en/docs/install/)
* [docker-compose](https://docs.docker.com/compose/install/)&#x20;

### Starting the mandala node and subquery

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
docker-compose up
```

It's ok to see some error messege in the docker logs, since we don't have transactions in the node yet, so subquery will keep crashing and restarting. Once there are transactions, everything will work normally.

![Running a local development network](<../../.gitbook/assets/image (12).png>)

{% hint style="info" %}
**NOTE: Use `Control + c` combination of keys to shut down the node.**

**In order to have a clean start after every shutdown of the node, run the following command after the node was shut down:**

**`docker-compose down -v`**
{% endhint %}

![Cleaning up the local development network](<../../.gitbook/assets/image (4).png>)

### Starting the RPC adapter

In addition to the local development network, we need to run an [Ethereum RPC adapter](https://github.com/AcalaNetwork/bodhi.js/tree/master/eth-rpc-adapter#acala-networketh-rpc-adapter), which provides [JSON-RPC](https://eth.wiki/json-rpc/API), so the tools that rely on JSON-RPC (such as Metamask, truffle, hardhat, etc...) will work. 

We run it in another window by using:

```shell
npx @acala-network/eth-rpc-adapter --local-mode --subql http://localhost:3001
```

![Starting up the RPC node](<../../.gitbook/assets/image (15).png>)

{% hint style="info" %}
**If you wish to learn more about the RPC adapter, you can find the info in the** [**RPC adapter documentation**](../../tooling/rpc-adapter/running-the-rpc-adapter.md)**.**
{% endhint %}

### The local development network services

Once the local development network is up and running, the following services are available:

* An ETH JSON-RPC service: 
  * [http://localhost:8545](http://localhost:8545)
  * [ws://localhost:8545](ws://localhost:8545)
* A subquery service: [http://localhost:3001](http://localhost:3001)
* A local mandala node: [ws://localhost:9944](ws://localhost:9944)&#x20;
* Local network Substrate chain explorer: [Polkadot.js App](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2Flocalhost%3A9944%2Fws#/explorer)

![Local development network in Substrate chain explorer](<../../.gitbook/assets/image (8).png>)

You can now setup [Metamask on localhost](../../tooling/metamask/#localhost) and interact with your local setup.

Try a few of the [EVM tutorials](../../tutorials/tutorials.md) to get familiar with the responses each terminal window provides as feedback.
