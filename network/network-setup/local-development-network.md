---
description: Instructions on how to run a full local development network
---

# Local development network

The local development network consists of following services:

* Local Acala Fork
* Eth Rpc Adapter
* \*Subquery Services
  * Postgres database
  * Subquery node
  * GraphQL engine

## Starting the stack
You can download or copy + paste this [docker compose file](https://github.com/AcalaNetwork/bodhi.js/blob/master/docker-compose.yml), and then

```
docker compose up
```

Once you see logs like this, the local development stack is ready.

```
--------------------------------------------
             🚀 SERVER STARTED 🚀
--------------------------------------------
version         : bodhi.js/eth-rpc-adapter/2.9.4
endpoint url    : ws://node:9944
subquery url    : undefined
server host     : localhost
server port     : 8545
max blockCache  : 200
max batchSize   : 50
max storageSize : 5000
cache capacity  : 1000
safe mode       : false
local mode      : false
http only       : false
verbose         : true
--------------------------------------------
```

This contains a local Acala fork, eth rpc adapter, without subquery services. For local testing purpose, we usually do not need subquery services, and if you would like to setup a local subquery service, you can refer to the [subquery docs]([../../tooling/subquery/running-subquery-locally.md](https://github.com/AcalaNetwork/bodhi.js/tree/master/packages/evm-subql#run-with-docker)).

{% hint style="info" %}
**In order to have a clean start after every shutdown of the node, run the following command after the node was shut down:**

**`docker compose down -v`**
{% endhint %}

![Cleaning up the local development network](<../../.gitbook/assets/image (4) (1).png>)

## The local development network services
Once the full local development network is up and running, the following services are available:

* A local mandala node: [ws://localhost:9944](ws://localhost:9944)
* \*A subquery service: [http://localhost:3001](http://localhost:3001)
* An ETH JSON-RPC service:
  * [http://localhost:8545](http://localhost:8545)
  * [ws://localhost:8545](ws://localhost:8545)
* Local network substrate chain explorer: [Polkadot.js App](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2Flocalhost%3A9944%2Fws#/explorer)

You can now setup [Metamask on localhost](../../tooling/metamask/#localhost) and interact with your local setup, or try to deploy or interact with a smart contract.
