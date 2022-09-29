---
description: Introduction into using Remix IDE with Acala EVM+.
---

# Remix IDE

You can use the [Remix IDE](https://remix.ethereum.org/), online smart contract development IDE, to deploy and interact with your smart contracts on Acala EVM+. Prerequisite for using Remix with Acala EVM+ is that you have your MetaMask wallet connected to the network. If you haven't done it yet, you can follow the [MetaMask setup guide](../metamask/).

## Connecting Remix IDE to MetaMask

To connect the Remix IDE to your MetaMask, and subsequently to the Acala EVM+, you have to first go to [Remix IDE](https://remix.ethereum.org/) and open the `Deploy & run transactions` tab.

{% hint style="info" %}
The `Deploy & run transactions` tab is represented by ![](<../../.gitbook/assets/Screenshot from 2022-01-27 12-23-19.png>) icon.
{% endhint %}

Once the tab opens, locate the `Environment menu` and select the `Injected Web3` option.

![](<../../.gitbook/assets/Screenshot from 2022-01-27 18-14-42.png>)

As you select the `Injected Web3` option, MetaMask window should pop up and prompt you to select accounts that you want to connect to Remix IDE.

![](<../../.gitbook/assets/Screenshot from 2022-01-27 18-18-12.png>)

The second screen should provide which information is received by Remix and you can finally connect the two. Once you do, the account should appear in the side `Deploy & run transactions` menu, as well as it's balance.

![](<../../.gitbook/assets/Screenshot from 2022-01-27 18-24-59.png>)

Now that we are ready to use the Remix IDE, we can take a look on how to interact with the smart contracts that are already deployed on the network and how to deploy our own.
