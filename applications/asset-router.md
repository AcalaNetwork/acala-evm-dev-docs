# Acala Multichain Asset Router

## What is the Acala Multichain Asset Router?

Acala Multichain Asset Router is a cross-chain asset transfer protocol built on Acala EVM+. It links the Wormhole Bridge and Cross-Consensus Message (XCM), enabling users to seamlessly transfer assets between any EVM chain and parachains in the Polkadot ecosystem.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

**Benefits for users:** use one application rather than multiple, better efficiency in transactions and fees.\
**Benefits for applications/networks:** integrate with Asset Router and expose to liquidity on networks supported by Wormhole, and Polkadot/Kusama networks supported by Acala/Karura.&#x20;

## Routing User Experience (UX)

Users can navigate to the Acala DApps UI, select the source and target chains, and choose the token to transfer. If the token is from EVM chains, the user will sign a transaction on MetaMask to approve the transfer. Conversely, if the token is from parachains, the user will sign a transaction with polkadot wallet. After the transaction confirmation, the user can click the "route" button on the UI to automatically transfer the asset to the target chain.

## Technical Details

### Components

The routing process involves several components:

* **Wormhole protocol**: This allows users to transfer assets between EVM chains and Acala/Karura.
* **Relayer**: This service automates wormhole token transfers, so users do not need to send extra redeem transactions on the target chain. It also encapsulates all interactions with the wormhole and asset router contracts, so the UI only needs to call its endpoints to complete the routing process, without worrying about low level details.
* **Asset router contracts**:
  * A router address on Acala/Karura, which is computed by the factory contract.
  * A factory contract, which calculates the router address based on the XCM or wormhole instructions. After the asset arrives at the router address, it deploys the router contract to the router address, and calls it to execute the routing.
* **Xtokens contracts**: A pre-deployed contract on Acala/Karura, which empowers EVM+ to call XCM that transfers tokens among parachains.

### Routing Process

The endpoints mentioned below are all associated with the relayer's endpoints.

#### EVM => Parachain

1. The user selects the source and target chains, as well as the token to transfer.
2. UI calls [/shouldRouteXcm](asset-router.md#shouldroutexcm) with the encoded config to get the router address on Acala/Karura.
3. UI prompts the user to sign a transaction with MetaMask that bridges through the wormhole from the EVM chain to the Acala/Karura router address.
4. UI fetches the Wormhole VAA.
5. UI calls [/relayAndRoute](asset-router.md#relayandroute) with the VAA. This performs two actions behind the scenes:
   * Relays (redeems) the token from the wormhole to the Acala/Karura router address.
   * Performs routing, which involves calling the Xtokens contract to XCM the token to the target parachain.

#### Parachain => EVM

1. The user selects the source and target chains, as well as the token to transfer.
2. UI calls [/shouldRouteWormhole](asset-router.md#shouldroutewormhole) with the encoded config to get the router address on Acala/Karura.
3. UI prompts the user to sign a transaction with the Polkadot wallet that XCMs the token from the source parachain to the router address on Acala/Karura.
4. UI calls [/routewormhole](asset-router.md#routewormhole). This sends the tokens to the wormhole from the router address.
5. UI fetches the Wormhole VAA.
6. UI prompts the user to sign a transaction with MetaMask that redeems the token on the target EVM chain.

## More References

* [asset router contracts](https://github.com/AcalaNetwork/asset-router/tree/master/src)
* [relayer source code](https://github.com/AcalaNetwork/wormhole-relayer)
* [wormhole docs](https://docs.wormhole.com/wormhole/)
* [wormhole development book](https://book.wormhole.com/)
* [Cross-Consensus Message (XCM)](https://wiki.polkadot.network/docs/learn-xcm)
* [xtokens pre-deployed contracts](https://github.com/AcalaNetwork/predeploy-contracts/blob/master/contracts/docs/xtokens/Xtokens.md)
