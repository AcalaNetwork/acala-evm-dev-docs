---
description: Explanation on how ACA ERC20 ties into Acala native token.
---

# ACA token

ACA token is synced in 3 places:
- acala native token
- evm native token
- evm predeployed ACA ERC20 token

Any change in the balance of any of the above, is reflected in all.

The predeployed ACA ERC20 smart contract is tied directly into the native ACA token of the Acala network. This means that you can transfer the ACA as you would the native currency, but you can also transfer it as an ERC20 token.


{% hint style="info" %}
Balances of the EVM accounts bound to the Polkadot.js accounts can differ from reasons other than existential deposit. One of such instances is staking.
{% endhint %}
