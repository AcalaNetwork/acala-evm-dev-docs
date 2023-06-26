# Gas parameters

The main difference between Acala EVM+ and legacy EVM is that we need to use encoded `gasPrice` and `gasLimit` (we will refer them as "gas parameters"). Manually inputting random gas parameters is discouraged.

## Context

As Acala EVM+ is running on a substrate chain, gas parameters need to encode four substrate parameters: `gasLimit`, `storageLimit`, `validUntil`, and `tip`. We need to provide a delicate `gasPrice` to `gasLimit`, so that they can be decoded correctly into the substrate parameters. Randomly changing gas parameters might result in a bad parameter decoding.

For example, when user sends a transaction with gas parameter
```
{
  gasPrice: 100.004623375 gwei,
  gasLimit: 100106,
}
```
these gas parameters will be decoded to substrate params
```
{
  validUntil: 4623375,
  gasLimit: 30000,
  storageLimit: 64,
  tip: 0,
}
```

Although this part is incompatible with the legacy EVM, it is actually an advantage of the Acala EVM+, which is able to utilise some features that cannot be found in the legacy EVM. By using `validUntil` parameter we can avoid transaction being stuck in the transaction pool indefinitely, and by using `storageLimit` we are able to encourage the developers to remove the data they don't need from the chain, in order to reduce the chain bloat.

## Getting the Gas parameters

### For users

Users never need to compute gas parameters by themselves:

* When sending tokens, MetaMask will call the RPC endpoints to get the correct gas parameters for them automatically.
* When signing a transaction, dApps will provide the correct gas parameters and pass these values to the MetaMask, so users just sign a valid transaction.

The only thing that need to pay attention to, is to not randomly modify the transaction parameters within MetaMask, otherwise the transaction will fail due to bad decoding. We thus encourage you to add the warning against such actions within the user interface of your dApps.

### For developers

Most toolings and libraries (such as `ethers`, `hardhat`, `truffle`) should automatically compute the correct gas parameters under the hood when sending a transaction, so developers usually don't need to do anything.

In case gas params are not auto computed, developers can easily compute the gas parameters in the following way:
```ts
const gasPrice = await provider.getGasPrice();
const gasLimit = await contractInstance.estimateGas.functionName(...args);
```

## Gas Decoding Details
### when there is no tip (default case)
Let's say eth `gasLimit` is encoded as `aaaabbbcc` and gasPrice is encoded as `100yyyyyyyyy`, which can be decoded to substrate gas params as following:
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

### when there is tip
`gasLimit` won't be affected by tip, since tip is encoded into gasPrice `ab0yyyyyyyyy`.
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

## Gas Modification
### for users
It generally not recommded for users to modify the gas parameters manually, since it might result in bad decoding. Dapps should trigger signature request with valid gas params, so users won't need to worry about gas calculation at all. However, if a user is familiar with the encoding, he can still manually modify `ab0` part of the gasPrice to increase tip, which can speed up the transaction when network is busy.

### for developers
Developers can provide different priority options for users, and compute the corresponding gasPrice for users, so users themselves don't need to modify the gas parameters manually. For example:
- default priority: `ab0 = 100`, in which case gasPrice is calculated automatically by the tooling
- high priority: `ab0 = 120`, in which case tip = 20% of the original cost
- super high priority: `ab0 = 200`, in which case tip = 100% of the original cost

However, if developers decide not to provide such options, they can just use the default gasPrice direclty.

Developers should also calculate valid gasLimit for users when prompt user signature. Usually gasLimit should also be calcualted automatically by the toolings, but if not, developers can simply hardcode a valid gasLimit instead (this should rarely happen, if so please report to Acala team). For example, if auto calculated `gasLimit = 100106` failed the transaction with error `storage limit not enough`, that means the transaction requires more storage than the auto computed `storageLimit = 2 ^ 6 = 64`, and let's say the actually storage cost is 100, devs can use `cc = 7` instead, so gasLimit becomes `100107`.