# About Acala EVM+

## Introduction

The vision of Acala is to build a decentralized permissionless DeFi platform. Ethereum is currently the largest DeFi platform. We would like to integrate with it by bridging the assets and liquidity of it. Our goal is to be compatible with Ethereum’s toolchains and tap into its large developer community. By building Acala EVM+, Ethereum developers will be able to deploy their solutions to Acala without friction.

Many blockchains also aims to be the better Ethereum. Aiming to providing full Ethereum compatibility to allow developers to reuse their existing toolchains and deploy existing contracts with no or minimal changes to deploy on their network. However, Ethereum is not yet perfect and it wouldn’t make sense for us to just ignore all of the issues that Ethereum users are experiencing currently and just build another, faster, Ethereum.

It is clear to us that just building a faster Ethereum is not what we want. With all the powerful and advanced features from Substrate, we aim to do better. All the while being friendly to existing Ethereum and Solidity developer community. EVM compatibility on Acala will be a stepping stone for the Solidity developers to tap into Polkadot ecosystem and have a taste of the new features that are simply not possible on Ethereum, such as bring your own gas (pay transaction fee with any supported token, powered by Acala Swap), powerful governance tools (no more locked funds) and full interoperability with the Polkadot ecosystem (no more centralized bridges).

> _Learn once, write anywhere._

Note that _“write once, run anywhere”_ is not our goal. Instead, we are inspired by the React Native approach _“learn once, write anywhere”_. Acala, and all Substrate based chains, are fundamentally different from Ethereum. We have our own trade-offs and therefore restrictions (in exchange for some other benefits). If we are trying to emulate an Ethereum node, we will be suffering from the worst of both worlds. It will be a step backwards for us to inherit all the restrictions from a legacy blockchain platform. Therefore it makes more sense to make some necessary compatibility sacrifices, to not be limited by decisions made by Ethereum developers many years ago. As a result, developers can take advantage of all of the advanced features that we are offering, while still using a familiar language (Solidity or another compile-to-evm languages). This means changes are likely required to port over existing projects. However with many new advanced features added to EVM via precompiles from Acala runtime, we hope the contracts can be significantly simplified and offer improved features and better usability to users.

## Challenges <a href="#challenges" id="challenges"></a>

There were two categories of challenges we were facing when design & building Acala EVM+. The first area stemed from the things we do not want to inherit from the legacy decisions of Ethereum. The second area originated from the inherent conflicts between Substrate and Ethereum.

### Issues <a href="#issues" id="issues"></a>

#### **Storage**

One of the main issues preventing people from running their own Ethereum nodes is that it requires a very large storage. This is partially due to the cost of on-chain storage being uneconomic.

On-chain storage is always expensive because it has to be replicated by every node and stored for many years, potentially forever. This is an ongoing cost and therefore the token economics should factor it in to avoid state explosion problem.

On Ethereum, the cost of inserting data into on-chain state is a one-off payment based on the gas price at the time when the transaction is processed. It doesn’t consider future costs and has a poor incentive to remove unused stored data. As a result, very little smart contracts actively purge unused states and no one actively removes unused contracts. The poorly designed storage gas refund makes [Gas Token](https://cointelegraph.com/news/a-new-token-lets-you-save-on-ethereum-fees-by-storing-gas) possible, which encourages people to store useless data when gas prices are low and to remove them when gas prices are high. This benefits individuals and miners, but raises the transaction cost for everyone else and increases the cost of operating Ethereum nodes at the same time. There are also a lot of dust accounts containing too little funds to send a transaction, and therefore no one is willing to reclaim those dust funds and these accounts just waste the on-chain storage.

#### **Gas price**

Another usability issue of Ethereum is that it is hard estimate how much gas price to pay. There are multiple services available ([\[1\]](https://ethgas.watch/), [\[2\]](https://gitcoin.co/gas/intro), [\[3\]](https://ethgasstation.info/), [\[4\]](https://www.gasnow.org/), [\[5\]](https://etherscan.io/gastracker)), but it is still not easy for people to balance between high gas prices and long confirmation times. Most of the people are just using the whatever value is provided by their wallet and wallets usually overestimate the gas price to ensure a fast confirmation time. However the throughput of the network is limited and this means everyone is overpaying and no one is getting faster confirmation times.

[EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) is one of the solution that addresses this issue. It proposes an algorithm to calculate a base fee for transaction, using the gas usage of previous blocks, instead of requiring users to estimate and supply the gas price. This is very similar to the [Fee Adjustment](https://wiki.polkadot.network/docs/en/learn-transaction-fees#fee-adjustment) mechanism implemented by Polkadot. Acala will adapt the slow-adjusting fee mechanism with tips from Polkadot to provide a better UX. This mechanism will be incorporated into the Acala EVM+ so the smart contract developers and users will not need to worry about gas price anymore.

### Conflicts <a href="#conflicts" id="conflicts"></a>

#### **Dust accounts**

While Substrate is indeed very customisable, it still comes with constraints and technical limitations. Some of those constraints are due to lessons learnt from building Ethereum and it is not something we would like to revert. For example, Polkadot implements [Existential Deposit](https://wiki.polkadot.network/docs/build-protocol-info#existential-deposit) which aims to solve the dust account issue. However, this also comes with a trade-off that when a dust account is wiped, the storage and dust fund are reclaimed. It means nonce can be reset and introduce a possibility of a replay attack. To prevent replay attacks, Polkadot supports mortal transactions (expiring transactions), so that it is no longer possible to replay old transactions. This fundamentally conflicts with Ethereum’s account model and transaction format.

#### **Compatible extensions**

There are few multi-chain wallets that support both Ethereum and Polkadot, but the most commonly used wallets, for both Ethereum and Polkadot, are still a single network wallets, namely MetaMask and Polkadot.js extension. This creates an issue where Polkadot.js extension cannot handle Ethereum transaction format and MetaMask cannot handle Substrate transaction format. As the result, user may need to use two browser extensions at same time, one for the Substrate runtime and one for EVM interactions. This is not something that is acceptable.

#### **Forkless upgrades**

One of the most advanced features offered by Substrate is forkless upgrades, that allow blockchain to evolve without forks. This reduces amount of coordination required and risk of network instability when performing upgrades. We also have an experimental Karura network that holds real value, on which we can try out new features. This means we are actually able to quickly (yet securely) iterate features, to improve the user’s and developer’s experiences of the network. We can try a lot of new things that are simply not feasible on Ethereum, which means we are no longer limited by what Ethereum is able to offer. This requires a change of mindset (why copy what Ethereum did, if we can do it better?) and opens a lot of possibilities and increases potential of the Acala EVM+.

## Solutions & Enhancements <a href="#solutions-amp-enhancements" id="solutions-amp-enhancements"></a>

We would like to implement a smart contract platform that takes the best of both worlds, while avoiding all of the pitfalls. This is actually very hard because we need to carefully balance all of the decisions and incompatibilities. Remember, most of the issues are not due to incompetent developers, but instead of people balancing trade-offs due to technical limitations. A solution of one problem usually leads to another set of problems. The art of Engineering is balancing trade-offs and building something best suited for the target audiences. Therefore we do not aim to build a perfect solution, but instead trying to make the right trade-offs to avoid all the pitfalls while keeping as much of the compatibility as possible.

### bodhi.js <a href="#bodhijs" id="bodhijs"></a>

One key component to be Ethereum compatible is RPC compatibility. i.e. Support Ethereum RPC. This is not easy to achieve due to many low level decisions made by Substrate. e.g. Ethereum RPC supports to get transaction by hash, which is not supported by Substrate RPC. This is because Substrate aims to be as lightweight as possible, and decided to not store those extra information by default. It is possible to customize Substrate to support all those extra requirements by using [Frontier](http://github.com/paritytech/frontier), but it does add a lot of extra complexity to the node and undos a lot of work to make the node lightweight. Therefore, we decided to go with an alternative approach.

Instead of emulating the full Ethereuem RPC on node side and to solve the problem of requiring multiple extensions, we have developed [bodhi.js](http://github.com/AcalaNetwork/bodhi.js), a JS SDK that offers an implementation of [Web3Provider](https://docs.ethers.io/v5/api/providers/) using polkadot.js and Substrate RPC. This means existing dApps that can use the Web3Provider offered by bodhi.js just like any other providers, such as the one provided by MetaMask. This will allow dApps to use bodhi.js to sign transactions using Polkadot.js extension as well as querying data from Acala nodes. As a result, a minimal change is required to port an existing Ethereum-based dApp to support Acala EVM+.

### Weight system <a href="#weight-system" id="weight-system"></a>

Substrate offers a weight system which is somewhat similar to gas fee system. We decided to use the weight system for charging gas fees by defining a gas-to-weight conversion ratio. As the result, we can completely ignore the gas price and use the weight system to handle the fees. The priority fee from EIP-1559 can be directly translated to the tip of the Substrate transaction. This allows the blockchain to handle EVM transactions like any other standard transactions. Some changes will be required on the dApp side to handle the transaction fee estimation and display, but it should be very straightforward.

### Renting storage <a href="#renting-storage" id="renting-storage"></a>

There are many researches on how to solve the state explosion problem and some of the possible solutions are [storage rent](https://wiki.polkadot.network/docs/build-smart-contracts#storage-rent) (which is deprecated) and [state expiry](https://notes.ethereum.org/@vbuterin/verkle\_and\_state\_expiry\_proposal). Both of them are complicated and not battle tested yet so we are not comfortable with using them in Acala EVM+ at this stage. Therefore we decided to use a much simpler model that charges a storage deposit for every byte used by the smart contract and refund the deposit upon clearing the storage used. This is able to create an incentive to free up unused storage while keeping compatibility with existing EVM smart contracts. The storage deposit in some sense can be viewed as the gas use/refund for storage operations, but the cost is a fixed amount of tokens per byte, instead of depending on the gas price, which depends on the block fullness utilisation. Similarly, the state deposit is also required for the code of new contracts, and encourage developers to remove any unused contracts from the on-chain storage.

### Storage meter <a href="#storage-meter" id="storage-meter"></a>

In order to accurately handle the storage deposit, we have implemented a storage meter in Acala EVM+. Similar to gas meter, which measures the gas usage, storage meter measures the storage usage and handles the storage deposit. This also adds a new input to transaction: storage limit. Gas limit is used to limit how much gas can be consumed by a transaction and storage limit is used to ensure the contract does not incur more than specified storage deposit.

### Developer mode <a href="#developer-mode" id="developer-mode"></a>

Being a DeFi chain, Acala always considers the security of our user’s funds the top priority. In Ethereum network, deploying a fraudulent contract is rather cheap. There is only an upfront development cost and even if the contract is banned or shunned by the users, the malicious developer can always deploy a new smart contract.

We feel that censorship is the wrong approach, so we developed a different approach to solving this issue. Our solution is to raise the bar of deploying new publicly accessible contracts, so that it becomes expensive to deploy malicious token contracts and useless forks. Contracts will initially only be accessible by the contract developers, a special role that anyone can opt-in to. Once the contracts have been deployed and tested, they can be made public by paying a relatively large amount of ACA (the exact value still has to be determined) in order for them to become publicly accessible. Genuine teams that want to deploy contracts but cannot pay the upfront cost, can request a grant from the Acala Treasury. This means that there is no additional cost of deploying contracts for internal testing and personal usage, but iteratively deploying fraudulent contracts to be publicly used will be expensive. Our hope is that this approach achieves the right balance between permissionlessnes and security of network user’s funds.

In the future we expect to be able to work with Identity parachains in order to give identity to the contracts. This should greatly improve the trustworthiness of verified contracts, because users will know they are interacting with a canonical contract instead of a malicious fork.

### Custom precompiles and predeploys <a href="#custom-precompiles" id="custom-precompiles"></a>

One of the things differentiating Acala EVM+ from the EVM is custom precompiles. Custom precompiles are smart contracts that are already compiled and deployed to the Acala EVM+ and available to the developers to incorporate within their own smart contracts. Additionally we are able to update these contracts via governance, which means that they will always be up to date with the latest security features and fixes. These predeploy contracts include [ERC-20 contracts](https://wiki.acala.network/build/development-guide/smart-contracts/advanced/use-native-tokens), like ACA, AUSD, DOT, LDOT and many more, as well as systemic contracts, like [State Rent](https://wiki.acala.network/build/development-guide/smart-contracts/advanced/use-flexi-fee#state-rent), [Oracle Price Feed](https://https/wiki.acala.network/build/development-guide/smart-contracts/advanced/use-oracle-feeds), [Scheduler](https://wiki.acala.network/build/development-guide/smart-contracts/advanced/use-on-chain-scheduler), [DEX](https://wiki.acala.network/learn/basics/dex). We expect DeFi contracts to join the list shortly.

Usage of these precompiled contracts opens a whole new area of possibilities for smart contract developers. The built-in Scheduler allows for automation of the smart contract business logic execution (no more external scripts to call smart contract functions periodically) and the Oracle can be used to set a stable price for services which are in turn paid with unstable tokens. The possibilities are truly endless.

Differences between EVM and Acala EVM+ mean that not every aspect of a transaction is compatible. If we wish for the users to be able to use Metamask, we have to provide a mechanic for it to work with the storage limit and expiration information. To do that we aim to use an [ERC-712](https://eips.ethereum.org/EIPS/eip-712) standard to sign the transactions and provide these kinds of information using it. This introduces another step when sending a transaction, compared to EVM, but allows us to support the existing extensions and browsers that are used on the Ethereum network rather than forcing the end user to migrate to another extension and/or wallet.

## Decisions <a href="#decisions" id="decisions"></a>

We had to make some decisions that are not fully compatible with EVM to make the step forward. These have been carefully considered and we want to share why we did them.

### Disable native token <a href="#disable-native-token" id="disable-native-token"></a>

Native token in Ethereum has 18 decimal spaces and Karura’s native token has 12 decimal spaces. This means that the two are incompatible. We could have used a conversion logic where we would multiply or divide the value by 1,000,000, depending on the way in which we are converting the tokens, but that could cause some serious errors. Imagine having to handle a value of 1 wei (the lowest possible value of Ethereum’s native token). Converting this to Karura’s we would divide this by 1,000,000 and get the value of 10^(-6). Since we are operating in a space where we don’t deal with floating point, the value would be rounded down, so the value would be 0. This could potentially break the flow of the operation that is running. Instead of accepting such a risk, we decided to disable native token in the Acala EVM+. This way we don’t compromise robustness of our virtual machine for support of an inherited trait.

Additionally we have taken into account that many of the DeFi contracts don’t operate using the native token, but all of them use ERC-20 tokens. Which means we are not disabling any of their functionality, while improving the operation of the EVM+.

### Dust accounts clearing <a href="#dust-accounts-clearing" id="dust-accounts-clearing"></a>

Clearing the dust accounts opens these addresses to a possibility of a replay attack, since the nonce is reset alongside the assets. To prevent that, each transaction has an expiration time. It would be easy to implement it as an additional parameter that has to be passed when calling a function, but that would put additional stress to the developers. We can’t add an additional field to Metamask and we can not force it to have a default value for the transaction expiration. Thus we devised a solution that uses the existing and supported operation by Metamask. Using ERC-712 typed signing standard will allow people to sign Acala EVM+ transaction with Metamask in a user friendly way. This allow us to provide the mechanics that we need, to provide as much security to user’s funds as possible, while serving users with an operation they already know when using Metamask.

### ERC-712 transactions <a href="#erc-712-transactions" id="erc-712-transactions"></a>

Using ERC-712 format of the transaction allows us to store storage limit within it as well. Since the user is paying a per-byte fee for the data they are storing, it is important to support a mechanic where the user has control over the maximum amount they are willing to deposit to rent the storage, much like the maximum amount of gas they are prepared to pay for in Ethereum network.

#### Embedding storage limit and transaction expiry into gas price <a href="#embedding-storage-limit-and-transaction-expiry-into-gas-price" id="embedding-storage-limit-and-transaction-expiry-into-gas-price"></a>

In addition to the ERC-712 transactions we foresee a method that both, storage limit and transaction expiry information, within the gas price of the transaction. This would allow for smoother network usage for bots and servers, as the transaction would be streamlined for programmatic usage.

Additionally we are able to use gas price to set the storage limit, since we’ve adopted the slow-adjusting fee mechanism with tips from Polkadot to provide a better UX for users. This mechanism will be incorporated into Acala EVM+, so the contract developers and users won’t have to worry about the gas price. This in turn sets the stage for using the gas price for something else, like the before-mentioned passing of the storage limit in the gas price field of the transaction. What is known as a priority fee value in Ethereum transactions, directly corresponds to Acala EVM+ tips, providing an additional familiarity for users and developers coming over from the Ethereum network.
