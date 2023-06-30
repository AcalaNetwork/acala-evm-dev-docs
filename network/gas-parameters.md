# Gas Parameters

The primary distinction between Acala EVM+ and the legacy EVM lies in the usage of encoded `gasPrice` and `gasLimit`, together referred to as "gas parameters".

## Context

Acala EVM+ operates on a substrate chain. Consequently, the gas parameters must encode four substrate parameters: `gasLimit`, `storageLimit`, `validUntil`, and `tip`. It's crucial to supply precise `gasPrice` to `gasLimit` values to ensure accurate decoding into substrate parameters. Arbitrary changes to these parameters could lead to incorrect decoding.

For instance, when a user sends a transaction with the following gas parameters:
```
{
  gasPrice: 100.004623375 gwei,
  gasLimit: 100106,
}
```

These parameters are decoded into substrate parameters as follows:
```
{
  validUntil: 4623375,
  gasLimit: 30000,
  storageLimit: 64,
  tip: 0,
}
```

Despite being inconsistent with the legacy EVM, this aspect is advantageous for Acala EVM+. It utilizes features unavailable in the legacy EVM. For example, the `validUntil` parameter prevents transactions from indefinitely lingering in the transaction pool. Additionally, the `storageLimit` encourages developers to remove redundant data from the chain, thereby reducing chain bloat.

## Retrieving Gas Parameters
### for users

Users are not required to calculate gas parameters:

- When sending tokens, MetaMask automatically retrieves the correct gas parameters by calling ETH RPC endpoints.
- During transaction signing, dApps provide the correct gas parameters to MetaMask, enabling users to sign a valid transaction.

However, users should avoid arbitrary modification of transaction parameters within MetaMask, as it could lead to transaction failure due to incorrect decoding. We recommend highlighting this warning within your dApp's user interface.

### for developers

Most tools and libraries (like `ethers`, `hardhat`, `truffle`) automatically calculate the correct gas parameters when sending a transaction. Developers typically do not need to intervene.

If gas parameters are not auto-computed, developers can calculate them as follows:

```ts
const gasPrice = await provider.getGasPrice();
const gasLimit = await contractInstance.estimateGas.functionName(...args);
```

## Gas Decoding Details
### without tip (default case)
Assume the Ethereum gasLimit is encoded as `aaaabbbcc` and gasPrice is encoded as `100yyyyyyyyy`. They can be decoded into substrate gas parameters as follows:
- `validUntil = yyyyyyyyy`
- `gasLimit = 30000 * bbb`
- `storageLimit = 2^min(21, cc)`

for example:
```
{
  gasPrice: 100004623375,   // 100yyyyyyyyy, where yyyyyyyyy = 004623375
  gasLimit: 100106,         // aaaabbbcc, where bbb = 001 and cc = 06
}
```
will be decoded as 
```
{
  validUntil: 4623375,  // yyyyyyyyy
  gasLimit: 30000,      // 30000 * bbb = 30000 * 1 = 30000
  storageLimit: 64,     // 2 ^ min(21, cc) = 2 ^ 6 = 64
  tip: 0,
}
```

### with tip
The `gasLimit` remains unaffected by the tip, as the tip is encoded into `gasPrice` as `ab0yyyyyyyyy`.
- `tip = (ab0 / 100 - 1)% of the original cost`

for example:
```
{
  gasPrice: 120004623375,   // ab0yyyyyyyyy, where ab0 = 120
  gasLimit: 100106,         // same as above
}
```
will be decoded as 
```
{
  validUntil: 4623375,  // same as above
  gasLimit: 30000,      // same as above
  storageLimit: 64,     // same as above
  tip: 20%,             // ab0 / 100 - 1 = 120 / 100 - 1 = 20%
}
```

## Modifying Gas Parameters
### for users
Users generally should avoid manually modifying gas parameters to prevent incorrect decoding. dApps should initiate signature requests with valid gas parameters, relieving users of any concern about gas calculation. Nonetheless, knowledgeable users can manually modify the `ab0` part of the gasPrice to increase the tip, which can expedite the transaction when the network is busy.

### for developers
Developers can offer different priority options to users and compute the corresponding `gasPrice`, eliminating the need for users to manually modify gas parameters. For instance:
- default priority: `ab0 = 100`, in which case gasPrice is calculated automatically by the toolings
- high priority: `ab0 = 120`, in which case tip = 20% of the original cost
- super high priority: `ab0 = 200`, in which case tip = 100% of the original cost

If developers choose not to offer such options, they can use the default gasPrice directly.

When prompting user signatures, developers should also calculate a valid gasLimit. Most often, the tools should auto-calculate the gasLimit. If not, developers can hardcode a valid gasLimit (a rare occurrence, but if it happens, please report to the Acala team). For example, if an auto-calculated `gasLimit = 100106` fails the transaction with an error like `storage limit not enough`, it implies the transaction requires more storage than the auto-computed storageLimit = `2 ^ 6 = 64`. If the actual storage cost is `100`, developers can use `cc = 7`, making `gasLimit = 100107`.
