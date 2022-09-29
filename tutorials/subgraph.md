---
description: Example locally hosted subgraph in Acala EVM+.
---

# Subgraph

The Subgraph that we will establish in this example demonstrates how to establish your own Subgraph for projects running in the Acala EVM+.

{% hint style="info" %}
**NOTE: As this example focuses on Subgraph, we won't be analysing the smart contract used. If you wish to learn more about building smart contracts within Acala EVM+, please refer to our** [**tutorials**](tutorials.md)**.**
{% endhint %}

## Setup

In order to be able to run the example, we need to run the local network first and then spin up the `graph-node` stack.

### Spin up the local development network

You can read up on how to spin up the local development network in the [documentation](https://evmdocs.acala.network/network/node-setup). For those that just want to get started with the Subgraph part, you need to use the following commands:

```shell
git clone --recurse-submodules git@github.com:AcalaNetwork/bodhi.js.git
cd bodhi.js/evm-subql
yarn
yarn build
docker compose up
```

Once the network converges (you should start seeing the `evm-subql-mandala-node-1` start reporting that it's ready), open up another terminal and run the RPC adapter:

```shell
LOCAL_MODE=1 SUBQL_URL=http://localhost:3001 npx @acala-network/eth-rpc-adapter@^2.4.12
```

Once you get the following output, your local development network and RPC adapter are ready:

```shell
  -------------------------------
  🔨 local development mode is ON
  ❌ don't use it for production!
  -------------------------------

2022-06-09 12:20:05        API/INIT: RPC methods not decorated: evm_blockLimits

  --------------------------------------------
               🚀 SERVER STARTED 🚀
  --------------------------------------------
  version        : bodhi.js/eth-rpc-adapter/2.4.12
  endpoint url   : ws://0.0.0.0::9944
  subquery url   : http://localhost:3001
  listening to   : http 8545 | ws 3331
  max cacheSize  : 200
  max batchSize  : 50
  max storageSize: 5000
  safe mode      : false
  local mode     : true
  --------------------------------------------
```

### Clone the example repository

Cloning this repository gives access to the `Gravity` smart contract as well as the migrations needed to deploy it. The `truffle.js` contains the required network configuration and the configuration files for a local subgraph node are included as well.

{% hint style="info" %}
**The example repository is available here:** [**https://github.com/AcalaNetwork/subgraph-example**](https://github.com/AcalaNetwork/subgraph-example)
{% endhint %}

```shell
git clone git@github.com:AcalaNetwork/subgraph-example.git
```

To install the required dependencies simply use:

```shell
yarn
```

This will install all of the dependencies needed to deploy the smart contract to Acala EVM+ and the dependencies required to spin up the subgraph.

The `package.json` contains scripts that allow for compilation and deployment of the smart contracts in the `"scripts"` section:

```json
    "build-contract": "truffle compile",
    "deploy-contract": "truffle migrate --network mandala --reset"
```

## Subgraph

The definition of the subgraph lives within `subgraph.yaml` and doesn't differ from other subgraph definitions. You can learn more about subgraph definitions in the [documentation](https://thegraph.com/docs/en/developer/create-subgraph-hosted/#the-subgraph-manifest).

The `Gravatar` is defined in the `schema.graphql` and doesn't differ from any schema designed for another environment. If you wish to learn more about building schemas, you can do so [here](https://thegraph.com/docs/en/developer/create-subgraph-hosted/#defining-entities).

The subgraph will live in the docker along with docker-hosted IPFS and Postgres. You can check out how the all of them connect to each other and to the already-running local development network in the docker-compose.yml.

## Usage

In order to use the locally hosted subgraph, we need to spin up our Docker containers by using:

```shell
docker compose up
```

Once the containers are running, you should see the following output:

```shell
subgraph-example-graph-node-1  | Jun 14 21:44:32.410 INFO Starting block ingestors with 1 chains [acala]
subgraph-example-graph-node-1  | Jun 14 21:44:32.410 INFO Starting block ingestor for network, network_name: acala
subgraph-example-graph-node-1  | Jun 14 21:44:32.411 INFO Starting firehose block ingestors with 0 chains []
subgraph-example-graph-node-1  | Jun 14 21:44:32.411 INFO Starting firehose block ingestors with 0 chains []
subgraph-example-graph-node-1  | Jun 14 21:44:32.412 INFO Starting job runner with 4 jobs, component: JobRunner
subgraph-example-graph-node-1  | Jun 14 21:44:32.452 INFO Starting JSON-RPC admin server at: http://localhost:8020, component: JsonRpcServer
subgraph-example-graph-node-1  | Jun 14 21:44:32.468 INFO Starting GraphQL HTTP server at: http://localhost:8000, component: GraphQLServer
subgraph-example-graph-node-1  | Jun 14 21:44:32.471 INFO Starting index node server at: http://localhost:8030, component: IndexNodeServer
subgraph-example-graph-node-1  | Jun 14 21:44:32.475 INFO Starting metrics server at: http://localhost:8040, component: MetricsServer
subgraph-example-graph-node-1  | Jun 14 21:44:32.476 INFO Starting GraphQL WebSocket server at: ws://localhost:8001, component: SubscriptionServer
subgraph-example-graph-node-1  | Jun 14 21:44:32.524 INFO Started all assigned subgraphs, node_id: default, count: 1, component: SubgraphRegistrar
subgraph-example-graph-node-1  | Jun 14 21:44:32.695 INFO Resolve subgraph files using IPFS, sgd: 1, subgraph_id: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm, component: SubgraphInstanceManager
subgraph-example-graph-node-1  | Jun 14 21:44:32.702 INFO Resolve schema, link: /ipfs/QmbSFRGGvHM7Cn8YSjDL41diDMxN4LQUDEMqaa5VVc5sC4, sgd: 1, subgraph_id: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm, component: SubgraphInstanceManager
subgraph-example-graph-node-1  | Jun 14 21:44:32.703 INFO Resolve data source, source_start_block: 0, source_address: Some(0xf80a32a835f79d7787e8a8ee5721d0feafd78108), name: Gravity, sgd: 1, subgraph_id: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm, component: SubgraphInstanceManager
subgraph-example-graph-node-1  | Jun 14 21:44:32.704 INFO Resolve mapping, link: /ipfs/Qmf1LHCv7pcM4sVykN4uTocLPSwHN5o9JzWcuLU9pqHgXF, sgd: 1, subgraph_id: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm, component: SubgraphInstanceManager
subgraph-example-graph-node-1  | Jun 14 21:44:32.704 INFO Resolve ABI, link: /ipfs/QmajZTadknSpgsCWRz9fG6bXFHdpVXPMWpx9yMipz3VtMQ, name: Gravity, sgd: 1, subgraph_id: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm, component: SubgraphInstanceManager
subgraph-example-graph-node-1  | Jun 14 21:44:32.728 INFO Successfully resolved subgraph files using IPFS, sgd: 1, subgraph_id: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm, component: SubgraphInstanceManager
subgraph-example-graph-node-1  | Jun 14 21:44:32.728 INFO Data source count at start: 1, sgd: 1, subgraph_id: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm, component: SubgraphInstanceManager
subgraph-example-graph-node-1  | Jun 14 21:44:33.076 ERRO registering metric [deployment_eth_rpc_request_duration] because it was already registered, component: MetricsRegistry
subgraph-example-graph-node-1  | Jun 14 21:44:33.077 ERRO registering metric [deployment_eth_rpc_errors] because it was already registered, component: MetricsRegistry
```

We are now ready to deploy the local subgraph. The steps to deploy it can be separated into two distinct groups:

1. Build and deploy the `Gravitar` smart contract to Acala EVM+
2. Create and deploy the subgraph

### Build and deploy the `Gravitar` smart contract to Acala EVM+

We can optionally reset the project by using the `clean` script:

```json
    "clean": "rm -rf data/ generated/ build/ temp/ lib/ network.json"
```

To be able to easily build the smart contract as well as deploy it, `build-contract` and `deploy-contract` scripts have to be added to `package.json`:

```json
    "build-contract": "truffle compile",
    "deploy-contract": "truffle migrate --network mandala --reset"
```

Running these scripts should succecssfully deploy the smart contract and populate the gravatars:

```shell
yarn clean             #optional
yarn build-contract
yarn deploy-contract
```

Now that the smart contract has been successfully deployed, we can move on to deploying the subgraph.

### Create and deploy the subgraph

Scripts to generate the Graph code, build it, create and deploy it need to be added to the `package.json`:

```json
    "build-graph": "graph build",
    "create-local": "graph create acala/examplesubgraph --node http://127.0.0.1:8020",
    "remove-local": "graph remove acala/examplesubgraph --node http://127.0.0.1:8020",
    "deploy-local": "graph deploy acala/examplesubgraph --ipfs http://127.0.0.1:5001 --node http://127.0.0.1:8020"
```

Running them should deploy the subgraph:

```shell
yarn codegen
yarn build-graph
yarn create-local
yarn deploy-local
```

When running the `deploy-local` script, you can decide to specify the version of the subgraph, but for the example purposes accepting the default value should suffice:

```shell
yarn deploy-local
yarn run v1.22.19
$ graph deploy acala/examplesubgraph --ipfs http://127.0.0.1:5001 --node http://127.0.0.1:8020
✔ Version Label (e.g. v0.0.1) · 
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Compile data source: Gravity => build/Gravity/Gravity.wasm
✔ Compile subgraph
  Copy schema file build/schema.graphql
  Write subgraph file build/Gravity/abis/Gravity.json
  Write subgraph manifest build/subgraph.yaml
✔ Write compiled subgraph to build/
  Add file to IPFS build/schema.graphql
                .. QmbSFRGGvHM7Cn8YSjDL41diDMxN4LQUDEMqaa5VVc5sC4
  Add file to IPFS build/Gravity/abis/Gravity.json
                .. QmajZTadknSpgsCWRz9fG6bXFHdpVXPMWpx9yMipz3VtMQ
  Add file to IPFS build/Gravity/Gravity.wasm
                .. Qmf1LHCv7pcM4sVykN4uTocLPSwHN5o9JzWcuLU9pqHgXF
✔ Upload subgraph to IPFS

Build completed: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm

Deployed to http://127.0.0.1:8000/subgraphs/name/acala/examplesubgraph/graphql

Subgraph endpoints:
Queries (HTTP):     http://127.0.0.1:8000/subgraphs/name/acala/examplesubgraph
Subscriptions (WS): http://127.0.0.1:8001/subgraphs/name/acala/examplesubgraph

✨  Done in 6.19s.
```

As you can see, we can now interact with the locally deployed subgraph.

### Quick build script

As an additionally supported script, we have added the `run-all` script to the `package.json`, which can be used to quickly deploy the smart contract as well as the locally hosted subgraph:

```shell
yarn run-all
yarn run v1.22.19
$ yarn build-all && yarn deploy-all
$ yarn clean && yarn build-contract && yarn codegen && yarn build-graph
$ rm -rf data/ generated/ build/ temp/ lib/ network.json
$ truffle compile

Compiling your contracts...
===========================
> Compiling ./contracts/Gravity.sol
> Compiling ./contracts/Migrations.sol
> Artifacts written to /Users/jan/Acala/subgraph-example/build/contracts
> Compiled successfully using:
   - solc: 0.4.25+commit.59dbf8f1.Emscripten.clang
$ graph codegen
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Load contract ABI from abis/Gravity.json
✔ Load contract ABIs
  Generate types for contract ABI: Gravity (abis/Gravity.json)
  Write types to generated/Gravity/Gravity.ts
✔ Generate types for contract ABIs
✔ Generate types for data source templates
✔ Load data source template ABIs
✔ Generate types for data source template ABIs
✔ Load GraphQL schema from schema.graphql
  Write types to generated/schema.ts
✔ Generate types for GraphQL schema

Types generated successfully

$ graph build
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Compile data source: Gravity => build/Gravity/Gravity.wasm
✔ Compile subgraph
  Copy schema file build/schema.graphql
  Write subgraph file build/Gravity/abis/Gravity.json
  Write subgraph manifest build/subgraph.yaml
✔ Write compiled subgraph to build/

Build completed: /Users/jan/Acala/subgraph-example/build/subgraph.yaml

$ yarn deploy-contract && yarn create-local && yarn deploy-local
$ truffle migrate --network mandala --reset

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.


Starting migrations...
======================
> Network name:    'mandala'
> Network id:      595
> Block gas limit: 15000000 (0xe4e1c0)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0xa4c6295639c832781b5a08d0e1caf7f048de2e11cc486532c3aa5202d78226ad
   > Blocks: 0            Seconds: 0
   > contract address:    0x1a906E71FF9e28d8E01460639EB8CF0a6f0e2486
   > block number:        17
   > block timestamp:     1655246466
   > account:             0x75E480dB528101a381Ce68544611C169Ad7EB342
   > balance:             10000990.675883098224
   > gas used:            239894 (0x3a916)
   > gas price:           11251.291893661 gwei
   > value sent:          0 ETH
   > total cost:          2.699117417537911934 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     2.699117417537911934 ETH


2_deploy_contract.js
====================

   Deploying 'GravatarRegistry'
   ----------------------------
   > transaction hash:    0x0539850eaa0f34b8583fa38af76436876f1e2edd8db584437f82346be1b98417
   > Blocks: 0            Seconds: 0
   > contract address:    0x78b1F763857C8645E46eAdD9540882905ff32Db7
   > block number:        19
   > block timestamp:     1655246478
   > account:             0x75E480dB528101a381Ce68544611C169Ad7EB342
   > balance:             10000988.456154170068
   > gas used:            1214156 (0x1286cc)
   > gas price:           2974.008289142 gwei
   > value sent:          0 ETH
   > total cost:          3.610910008311494152 ETH

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     3.610910008311494152 ETH


3_create_gravatars.js
=====================
Account address: 0x78b1F763857C8645E46eAdD9540882905ff32Db7
   > Saving migration to chain.
   -------------------------------------
   > Total cost:                   0 ETH

Summary
=======
> Total deployments:   2
> Final cost:          6.310027425849406086 ETH


$ graph create acala/examplesubgraph --node http://127.0.0.1:8020
Created subgraph: acala/examplesubgraph
$ graph deploy acala/examplesubgraph --ipfs http://127.0.0.1:5001 --node http://127.0.0.1:8020
✔ Version Label (e.g. v0.0.1) · 
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4
  Skip migration: Bump mapping apiVersion from 0.0.4 to 0.0.5
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Compile data source: Gravity => build/Gravity/Gravity.wasm
✔ Compile subgraph
  Copy schema file build/schema.graphql
  Write subgraph file build/Gravity/abis/Gravity.json
  Write subgraph manifest build/subgraph.yaml
✔ Write compiled subgraph to build/
  Add file to IPFS build/schema.graphql
                .. QmbSFRGGvHM7Cn8YSjDL41diDMxN4LQUDEMqaa5VVc5sC4
  Add file to IPFS build/Gravity/abis/Gravity.json
                .. QmajZTadknSpgsCWRz9fG6bXFHdpVXPMWpx9yMipz3VtMQ
  Add file to IPFS build/Gravity/Gravity.wasm
                .. Qmf1LHCv7pcM4sVykN4uTocLPSwHN5o9JzWcuLU9pqHgXF
✔ Upload subgraph to IPFS

Build completed: QmWbRi1CvZGy2sKbdp549dH99KDbALZKy3133WyK1CgaHm

Deployed to http://127.0.0.1:8000/subgraphs/name/acala/examplesubgraph/graphql

Subgraph endpoints:
Queries (HTTP):     http://127.0.0.1:8000/subgraphs/name/acala/examplesubgraph
Subscriptions (WS): http://127.0.0.1:8001/subgraphs/name/acala/examplesubgraph

✨  Done in 24.44s.
```

## Subgraph interaction

The playground is reachable at [http://127.0.0.1:8000/subgraphs/name/acala/examplesubgraph/graphql](http://127.0.0.1:8000/subgraphs/name/acala/examplesubgraph/graphql). You can query the gravatars from the `Gravity` smart contract using the following query:

```graphql
query {
  gravatars {
    id,
    displayName,
    imageUrl,
    owner
  }
}
```

The result should give information about three gravatars:

```json
{
  "data": {
    "gravatars": [
      {
        "id": "0x0",
        "displayName": "AAAAA",
        "imageUrl": "https://example/AAAAA.png",
        "owner": "0x75e480db528101a381ce68544611c169ad7eb342"
      },
      {
        "id": "0x1",
        "displayName": "BBBBB",
        "imageUrl": "https://example/BBBBB.jpg",
        "owner": "0x0085560b24769dac4ed057f1b2ae40746aa9aab6"
      },
      {
        "id": "0x2",
        "displayName": "CCCCC",
        "imageUrl": "https://example/CCCCC.jpg",
        "owner": "0x0294350d7cf2c145446358b6461c1610927b3a87"
      }
    ]
  }
}
```

![Querying the local subgraph](<../.gitbook/assets/image (39).png>)

This concludes out locally hosted subgraph example in Acala EVM+.
