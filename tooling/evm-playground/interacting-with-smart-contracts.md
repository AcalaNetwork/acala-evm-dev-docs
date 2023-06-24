---
description: >-
  Instructions on how to use EVM playgrounds to interact with smart contracts
  deployed on Acala EVM+.
---

# Interacting with smart contracts

You can use the [EVM playground](https://evm.acala.network/) in order to interact with smart contracts that have been deployed to the Acala EVM+.

To do that, open [https://evm.acala.network/#/execute](https://evm.acala.network/#/execute):

![EVM Playground => Execute](<../../.gitbook/assets/image (7).png>)

Select the **Add and Existing Contract** option, which will open the form to add an existing smart contract to the collection of the executable smart contracts. For the sake of the documentation, aUSD ERC20 smart contract will be added. You can get its address from the [ADDRESS utility](https://github.com/AcalaNetwork/predeploy-contracts/blob/master/contracts/utils/Address.sol) and the precompiled ABI of the smart contract within the [AcalaNetwork/predeploy-contracts](https://github.com/AcalaNetwork/predeploy-contracts) repository. Once you fill out the form you can click **Save** and the smart contract will be added to your collection.

![Filled out form for adding aUSD smart contract to the EVM Playground](<../../.gitbook/assets/image (6) (1).png>)

{% hint style="info" %}
In some rare cases, the collection page won't load after you save a new smart contract, so you need to refresh the page.
{% endhint %}

Once you locate your newly saved smart contracts to the executable smart contracts collection, you can interact with it by pressing the **Execute** button below its address.

![Saved Executable smart contract](<../../.gitbook/assets/Screenshot 2022-03-03 at 21.25.37.png>)

If you are using the EVM Playground for the first time you will need to connect your MetaMask to it in order to be able to use it. To connect your MetaMask, click the **Connect to MetaMask** button, which will prompt the MetaMask to open and you can select the desired account that you want to connect with the EVM Playgrounds.

![Connect MetaMask before you interact with the EVM Playgrounds](<../../.gitbook/assets/Screenshot 2022-03-03 at 21.28.20.png>)

Once your MetaMask is connected to the EVM Playgrounds, you can interact with the smart contract, by selecting the function that you would like to call from the **Message to Send** dropdown menu and filling out the required parameters (if there are any). Once you are satisfied with the call and the values passed to it, you can press the **Call** button, which will initiate the transaction.

![Selecting the Message to Send and sending the Call](<../../.gitbook/assets/Screenshot 2022-03-03 at 21.35.04.png>)
