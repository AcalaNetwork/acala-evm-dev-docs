---
description: >-
  Instructions how to request support for your project in a format that will
  allow for the most efficient support.
---

# Request support

In order for us to be able to lend a hand when you hit a snag, we want to share a simple checklist of what to include in your support request.

## Brief description

A brief description of your issue that illustrates what issues you have encountered and how it impacts you. For example:

We have tried deploying a smart contract using Truffle framework, but the transaction has failed with `Error: 1010: Invalid Transaction: Transaction is outdated` error.

## Source

Providing us with the source code will allow for an easier debugging by our engineers. Being able to look at the code and manipulate it in order to provide a solution will eliminate the guesswork.

Source can be a link to a branch in your repository that encountered the issue or a mock of it (in case you don't feel comfortable sharing your source code with us). If you are sharing a mock branch it is important that you preserve the dependencies you are using and the configuration as well as the flow of the script that encounters the issue (you may change the contract and variable names and values, but keep the statements as close to the original as possible).

## Reproduction instructions

Step by step instructions on how to reproduce the issue. Be as detailed as possible. Reproduction instructions should start with cloning the source and end with reproducing the issue. For example:

1. `git clone REPOSITORY`
2. `yarn`
3. `yarn build`
4. `yarn reproduce-issue`

## Additional details

Details about your environment that could be specific to you are important. Make sure you have bound your Substrate and EVM accounts and enabled the development mode. Do you have sufficient balance? Hash of the transaction that failed (not screenshot, as the hash can be copy-pasted) can greatly increase the chances of resolving your issue.

## How to request support?

Remember to gather all of the information above and have it ready.

We encourage you to first post your question to [Substrate stack exchange](https://substrate.stackexchange.com/) or to check it out whether or not your question has already been answered. Please use `acala`, `karura` or `mandala` tags (depending on the network you are using) and the `evm+` tag, to make sure one of our engineers sees your question.

If you are unable to get sufficient support, reach out to us in one of our many channels of communication and we will assist you.
