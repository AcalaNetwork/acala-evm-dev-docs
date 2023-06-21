# Asset Router
Asset router is a cross-chain asset transfer protocol built on Acala EVM+. It connects wormhole bridge and XCM to allows users to smoothly transfer assets between EVM+ and other parachains in polkadot ecosystem.

## Routing UX
User can go to Acala DApps UI and select source and target chain, as well as the token to transfer. If the token is from EVM chains, user will sign a transaction on metamask to approve the transfer. If the token is from parachains, user will sign a transaction on polkadot.js extension instead. And after the transaction is confirmed, user can then click "route" button on the UI, so the asset will be transferred to the target chain automatically.

## Techinal Details
### Components
There are a couple component in the routing process:
- **wormhole protocol**, through which user can transfer assets between EVM chains and Acala/Karura.
- **relayer**, which is a service that help automate wormhole token transfer, so that users won't need to send an extra transtions on target chain. It also encapsulates all interactions with wormhole and asset router contracts, so UI don't need to worry about low level details, and only need to call it's endpoints to comlete the whole routing process.
- **asset router contracts**:
  - a router address on Acala/Karura, which is computed by the factory contract
  - a factory contract, which computes the router address based on the xcm or wormhole instructions. After the asset has arrived at the router address, it will deploy the router contract on the router address, and call the router contract to perform the routing.
- **Xtokens contracts**, which is a pre-deployed contracts on Acala/Karura, which enpowers EVM+ to call XCM that trasnfer tokens among parachains.

### Routing Process
A complete working flow can be found in [routing e2e tests](./src/__tests__/route.test.ts). The endpoints mentioned below are all relayer's endpoints.

### evm => parachain
1) User select source and target chain, as well as the token to transfer
2) UI call [/shouldRouteXcm](#shouldroutexcm) with the encoded above config to get the router address on Acala/Karura
3) UI prompt user to sign a tx with Metamask that bridges through wormhole from evm chain to Acala/Karura router address
4) UI fetchs wormhole VAA
5) UI call [/relayAndRoute](#relayandroute) with the VAA, this does two things under the hood:
   - relay (redeem) the token from wormhole to Acala/Karura router address
   - then perform routing, which calls xtokens contract to xcm token to the target parachain

### parachain => evm
1) User select source and target chain, as well as the token to transfer
2) UI call [/shouldRouteWormhole](#shouldroutewormhole) with the encoded above config to get the router address on Acala/Karura
3) UI prompt user to sign a tx with polkadot wallet that XCM token from source parachain to the router address on Acala/Karura
4) UI call [/routewormhole](#routewormhole), this will send the tokens to wormhole from router address
5) UI fetchs wormhole VAA
6) UI prompt user to sign a tx with Metamask that redeems the token on the target evm chain

## More References
- [asset router contracts](https://github.com/AcalaNetwork/asset-router/tree/master/src)
- [relayer source code](https://github.com/AcalaNetwork/wormhole-relayer)
- [wormhole docs](https://docs.wormhole.com/wormhole/)
- [wormhole development book](https://book.wormhole.com/)