---
description: >-
  Instructions on how to connect MetaMask to Mandala EVM+ in order to interact
  with the smart contracts deployed on it.
---

# Connect to the network

You can add the network to the MetaMask using the Chainlist  service or follow the instructions on how to add the network manually.

## Automated process

Follow the link to the network you are trying to add, connect your wallet and the network will be added to your wallet.

{% embed url="https://chainlist.org/chain/595" %}
Automated process of adding the Mandala EVM+ network to your wallet
{% endembed %}

{% embed url="https://chainlist.org/chain/686" %}
Automated process of adding the Karura EVM+ network to your wallet
{% endembed %}

{% embed url="https://chainlist.org/chain/787" %}
Automated process of adding the Acala EVM+ network to your wallet
{% endembed %}

## Manual process

{% hint style="info" %}
NOTE: The manual process example connects to the Mandala EVM+. For other networks can substitute the values with any of the values provided at the [network configuration](../../network/network-configuration.md) section.
{% endhint %}

In order to be able to interact with the Acala EVM+ in Mandala TC8, you first need to navigate to the **Add network** section of the MetaMask. You can find it at the bottom of the list of available networks after clicking on the currently active network.

![MetaMask => Currently active network => Add network](https://1503523808-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MAz4EenwXLth\_HO\_hmJ-887967055%2Fuploads%2FWz1rByJAVVr5MOdxgEaS%2FScreenshot%202022-03-02%20at%2002.22.49.png?alt=media\&token=365d2c22-49d2-4952-94cf-54a7fe154ad8)

This should open up a form to add a new network to your MetaMask (you might have to unlock MetaMask before it opens). Once the form is opened, use the following information to add the Mandala TC8 network.

### Mandala TC8 connection details

| **Network name**       | Mandala TC8                                              |   |
| ---------------------- | -------------------------------------------------------- | - |
| **New RPC URL**        | `https://eth-rpc-mandala.aca-staging.network` |   |
| **Chain ID**           | 595                                                      |   |
| **Currency symbol**    | ACA                                                     |   |
| **Block Explorer URL** | `https://blockscout.mandala.acala.network/`              |   |

![Mandala TC8 connection details](<../../.gitbook/assets/Screenshot 2022-07-07 at 11.48.11.png>)

Mandala TC8 should now be connected and you should see your ACA balance (if you already have it).

![MetaMask connected to Mandala TC8](https://1503523808-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MAz4EenwXLth\_HO\_hmJ-887967055%2Fuploads%2FUIMYw8u7yCY6RFEhXJa3%2Fimage.png?alt=media\&token=e0bb8dd9-d6d6-4e45-a9f6-974d6299eb69)

You might have to bind your MetaMask account to your Substrate account in order to see your balance. Documentation on how to do it is available [here](../development-account/#bind-accounts).
