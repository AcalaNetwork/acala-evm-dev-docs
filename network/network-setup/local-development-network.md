---
description: Instructions on how to run a full local development network
---

# Local development network

The local development network consists of following services:

* Mandala node
* Subquery Services
  * Postgres database
  * Subquery node
  * GraphQL engine
* Eth Rpc Adapter

## Starting the stack
### start the whole stack together
You can download or copy + paste this [docker compose file](https://github.com/AcalaNetwork/bodhi.js/blob/master/examples/docker-compose-bodhi-stack.yml), and then

```shell
docker compose -f docker-compose-bodhi-stack.yml up
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

**`docker compose down -v`**
{% endhint %}

![Cleaning up the local development network](<../../.gitbook/assets/image (4) (1).png>)

### start a light stack
A light stack is a stack without subquery services, it only contains mandala node and eth rpc adapter. It's useful when you want to test some simple transactions, but it lacks the ability to fetch logs. ([when do I need subquery?](https://evmdocs.acala.network/miscellaneous/faqs#when-do-i-need-to-provide-subquery-url-for-eth-rpc-adpater-or-evmrpcprovider))

To start a light stack, first start a local mandala node
```
docker run -it --rm -p 9944:9944 -p 9933:9933 ghcr.io/acalanetwork/mandala-node:sha-a32c40b --dev --ws-external --rpc-port=9933 --rpc-external --rpc-cors=all --rpc-methods=unsafe -levm=debug --pruning=archive --instant-sealing
```

Then start an eth rpc adapter
```
docker run -it --rm -p 8545:8545 acala/eth-rpc-adapter:2.6.8 --endpoint ws://host.docker.internal:9944 --local-mode
```

## The local development network services

Once the full local development network is up and running, the following services are available:

* A local mandala node: [ws://localhost:9944](ws://localhost:9944)
* A subquery service: [http://localhost:3001](http://localhost:3001)
* An ETH JSON-RPC service:
  * [http://localhost:8545](http://localhost:8545)
  * [ws://localhost:8545](ws://localhost:8545)
* Local network substrate chain explorer: [Polkadot.js App](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2Flocalhost%3A9944%2Fws#/explorer)

![Local development network in Substrate chain explorer](<../../.gitbook/assets/image (8).png>)

You can now setup [Metamask on localhost](../../tooling/metamask/#localhost) and interact with your local setup.

Try a few of the [EVM tutorials](../../examples/examples.md) to get familiar with the responses each terminal window provides as feedback.
