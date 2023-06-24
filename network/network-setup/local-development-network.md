---
description: Instructions on how to run a local development network
---

# Local development network

## Prerequisites

The tooling needed in order to successfully run a local EVM+ development network are:

* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Node.js](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
* [yarn](https://classic.yarnpkg.com/lang/en/docs/install/)

## Starting the stack

To be able to run the local development network, you need to clone our `bodhi.js` monorepo. As the monorepo contains submodules, we recommend cloning them as well:

```shell
git clone --recurse-submodules git@github.com:AcalaNetwork/bodhi.js.git
cd bodhi.js
```

The local development network consists of:

* Mandala node
* Subquery Services
  * Postgres database
  * Subquery node
  * GraphQL engine
* Eth Rpc Adapter

To spin them up all together, use:

```shell
docker compose up
```

Once you see logs like this, the local development stack is ready.

```
 --------------------------------------------
              🚀 SERVER STARTED 🚀
 --------------------------------------------
 version         : bodhi.js/eth-rpc-adapter/2.7.3
 endpoint url    : ws://mandala-node:9944
 subquery url    : http://graphql-engine:3001
 listening to    : 8545
 max blockCache  : 200
 max batchSize   : 50
 max storageSize : 5000
 safe mode       : false
 local mode      : false
 rich mode       : false
 http only       : false
 verbose         : true
 --------------------------------------------
```

It's ok to see some error messege in the docker logs, since we don't have transactions in the node yet, so subquery will keep crashing and restarting. Once there are transactions, everything will work normally.

{% hint style="info" %}
**In order to have a clean start after every shutdown of the node, run the following command after the node was shut down:**

**`docker-compose down -v`**
{% endhint %}

![Cleaning up the local development network](<../../.gitbook/assets/image (4) (1).png>)

### The local development network services

Once the local development network is up and running, the following services are available:

* An ETH JSON-RPC service:
  * [http://localhost:8545](http://localhost:8545)
  * [ws://localhost:8545](ws://localhost:8545)
* A subquery service: [http://localhost:3001](http://localhost:3001)
* A local mandala node: [ws://localhost:9944](ws://localhost:9944)
* Local network Substrate chain explorer: [Polkadot.js App](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2Flocalhost%3A9944%2Fws#/explorer)

![Local development network in Substrate chain explorer](<../../.gitbook/assets/image (8).png>)

You can now setup [Metamask on localhost](../../tooling/metamask/#localhost) and interact with your local setup.

Try a few of the [EVM tutorials](../../tutorials/tutorials.md) to get familiar with the responses each terminal window provides as feedback.
