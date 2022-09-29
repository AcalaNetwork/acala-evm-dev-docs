---
description: Explanation on how ACA ERC20 ties into Acala native token.
---

# ACA token

The predeployed ACA ERC20 smart contract is tied directly into the native ACA token of the Acala network. This means that you can transfer the ACA as you would the native currency, but you can also transfer it as an ERC20 token.

Any change in the balance of either is reflected in both.

You might see a difference in balance in MetaMask compared to the one you see in Polkadot.js. This is due to existential deposit that equals 1 ACA. So MetaMask balance will be 1 ACA lower than Polkadot.js balance.

![MetaMask balance](<../.gitbook/assets/image (34).png>)

![Polkadot.js balance](<../.gitbook/assets/image (37).png>)

{% hint style="info" %}
Balances of the EVM accounts bound to the Polkadot.js accounts can differ from reasons other than existential deposit. One of such instances is staking.
{% endhint %}
