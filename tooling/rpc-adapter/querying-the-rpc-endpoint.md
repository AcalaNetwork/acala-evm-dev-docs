# Querying the RPC endpoint

Information about the chain can be queried using the RPC calls through the RPC endpoint. The easiest way to do so is using [Postman](https://www.postman.com/), so that we can save the calls and reuse them later.

For example, you can query the RPC endpoint of the Mandala TC7 network using the following address: `https://eth-rpc-mandala.aca-staging.network`.

As the RPC request is in the JSON format, we have to select the correct body setting. The correct setting is raw body with JSON formatting. The request for getting the [encoded](../../network/gas-parameters.md) gas parameters is:

```json5
{
  "id": 0,
  "jsonrpc": "2.0",
  "method": "eth_getEthGas"
}
```

The prepared call within the user interface of the application looks like this:

![eth\_getEthGas RPC call in Postman](<../../.gitbook/assets/image (48).png>)

After pressing `Send`, the _Postman_ will send an RPC request to the RPC response and that should return a JSON formatted response similar to this one:

```json5
{
    "id": 0,
    "jsonrpc": "2.0",
    "result": {
        "gasPrice": "0x2f145b03ea",
        "gasLimit": "0x329b140"
    }
}
```
