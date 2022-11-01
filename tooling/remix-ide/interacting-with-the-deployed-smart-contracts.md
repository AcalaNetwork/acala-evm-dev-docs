---
description: Walk through on how to interact with an already deployed smart contract.
---

# Interacting with the deployed smart contracts

Now that Remix IDE is connected to the Mandala test network, we can interact with the smart contracts deployed on it. As you complete the setting up, you can take a look at the `File explorers` section. It should already include folders named `contracts`, `scripts` and `tests`, as well as `REAMDE.txt`.

{% hint style="info" %}
File explorers section is represented by the <img src="../../.gitbook/assets/Screenshot from 2022-01-29 00-08-33.png" alt="" data-size="line"> icon.
{% endhint %}

We can use a very simple smart contract that is further explained in the tutorials section and is already deployed on the Mandala TC8 network. The smart contract is called Echo and it has one function that stores a value passed to it in a public variable, which we are able to get using it's getter function. In order to use Remix IDE to interact with it, we need to add it into the `contracts` folder. We do this by option-clicking onto the folder and selecting `New file` option. the file should be named `Echo.sol`. You can now copy-paste the following code into the file:

{% code title="Echo.sol" %}
```solidity
pragma solidity =0.8.9;

contract Echo{
    string public echo;
    uint echoCount;

    event NewEcho(string message, uint count);

    constructor() {
        echo = "Deployed successfully!";
    }

    function scream(string memory message) public returns(string memory){
        echo = message;
        echoCount += 1;
        emit NewEcho(message, echoCount);
        return message;
    }
}
```
{% endcode %}

Once you save the file, the Remix IDE built-in compiler will run and compile the smart contract. If the compilation fails (you can see that by a red error indicator appearing over the Solidity compiler section <img src="../../.gitbook/assets/Screenshot from 2022-01-29 00-19-48.png" alt="" data-size="line">), you might have to manually set the compiler version to `0.8.9.`

![](<../../.gitbook/assets/Screenshot from 2022-01-29 00-21-43.png>)

As the smart contract compiles as expected, we need to point to the address to which it is deployed to. There is an instance deployed at `0x87c8Dc09548195A3B1222ab5c3905c01595D5516`, so you can use this one. To use it, navigate to `Deploy & run transactions` tab and paste the address into the `At Address` section.

{% hint style="info" %}
The Deploy & run transactions is represented by <img src="../../.gitbook/assets/Screenshot from 2022-01-30 23-04-41.png" alt="" data-size="line"> icon.
{% endhint %}

![](<../../.gitbook/assets/Screenshot from 2022-01-30 23-06-16.png>)

Once you click on the `At Address` button, you should see `ECHO` in the `Deployed contracts` section.

![](<../../.gitbook/assets/Screenshot from 2022-01-30 23-08-00.png>)

Once you expand the view, you can interact with the smart contract. If you select the `echo` getter, there will be no MetaMask confirmation needed, as no transaction is executed. If you select scream, then you should pass a string to it within the `""` and confirm the transaction in MetaMask as the prompt appears.
