# Batch EVM Transactions

One of the advanced feature of EVM+, compared to traditional EVM, is the ability to do batch transaction.

This example will show how to batch transactions with polkadot wallet and [bodhi.js](https://github.com/AcalaNetwork/bodhi.js/tree/master/bodhi#create-a-wallet) SDK，so users can deploy multiple contracts at once, and perform approve token and add liquidity transactions within a single transaction.

{% hint style="info" %}
Before diving into this advanced example, we suggest going over the [basic example](./using-bodhi.js-to-deploy-smart-contract-and-interact-with-it.md) first
{% endhint %}

The example is [here](https://github.com/AcalaNetwork/bodhi-examples/tree/master/batch-transactions).