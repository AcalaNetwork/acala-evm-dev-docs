---
description: Example on how to transfer ACA using MetaMask.
---

# Simple transactions

MetaMask can be used to transfer ACA in the same manner, you would transfer Ether in Ethereum network:

1. Open and unlock the MetaMask
2. Click on **Send** option
3. Paste the EVM account address that you want to send the ACA to and input the amount of ACA you want to send and click **Next**
4. Verify that the transaction data is correct and click **Confirm**
5. You are done! The ACA has been sent to the desired address

![MetaMask => Send => Input amount & Next => Confirm](<../../.gitbook/assets/image (40).png>)


{% hint style="info" %}
before sending any transaction, please don't change the default `gasPrice` or `GasLimit`, otherwise transaction will fail. ([why?](../../miscellaneous/FAQs.md#why-tx-failed-after-i-manually-changed-gas-params-in-metamask))
{% endhint %}

{% hint style="info" %}
if we are using local mandala, everytime we restart the local mandala node, we need to reset metamask for local network, so the nonce and cache will be cleared: `settings => advanced => reset account`. ([why?](../../miscellaneous/FAQs.md#why-metamask-tx-doesnt-confirm-with-local-mandala))
{% endhint %}
