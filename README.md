# Learning Foundry 😛

## Running a local node:

```sh
anvil
```

## Creating / Importing a new account for testing

```sh
cast wallet import quietRat --interactive

cast wallet list

quietRat
```

After typing the private key, and a password to encrypt it, a new account will be created in the `~/.foundry/keystores/` directory under the name `quietRat`.

## Deploy 🚀 a contract into the anvil node

```sh
# Dry run 🌵
forge create Counter --account quietRat

# Deploy to local node
forge create Counter --account quietRat --broadcast
```

Deploy using a script. This script runs only on a simulated one-time chain, after the script ends the simulation stops and the deployment is not persistent.

```sh
forge script script/Counter.s.sol --broadcast
```

If you want to point to an specific chain, even if it's our local anvil chain, you need to declare an RPC URL.

```sh
forge script script/Counter.s.sol --broadcast --rpc-url 127.0.0.1:8545 --account quietRat
```

Make sure that the code in the deployer script returns the contract, this will give you address where the contract was deployed.

```sol
contract CounterScript is Script {
    Counter public counter;

    function setUp() public {}

    function run() public returns (Counter){
        vm.startBroadcast();

        counter = new Counter();

        vm.stopBroadcast();

        return counter;
    }
}
```

If you want to verify the contracts in etherscan make sure you follow [**this instructions**](https://book.getfoundry.sh/guides/scripting-with-solidity#configuring-foundrytoml).

Add this to the `.env` file:

```txt
SEPOLIA_RPC_URL=
ETHERSCAN_API_KEY=
```

And, this to the `foundry.toml`:

```toml
[rpc_endpoints]
sepolia = "${SEPOLIA_RPC_URL}"

[etherscan]
sepolia = { key = "${ETHERSCAN_API_KEY}" }
```

## Post-deploy

The details of the latest deploy are stored in our repository in `broadcast/Counter.s.sol/31337/run-latest.json`. If you open the file, you will see the transaction details of the deployment.

```json
"transaction": {
    "from": "0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266",
    "gas": "0x31c46",
    "value": "0x0",
    "input": "0x608060405234801561000f575f80fd5b506101e18061001d5f395ff3fe608060405234801561000f575f80fd5b506004361061003f575f3560e01c80633fb5c1cb146100435780638381f58a1461005f578063d09de08a1461007d575b5f80fd5b61005d600480360381019061005891906100e4565b610087565b005b610067610090565b604051610074919061011e565b60405180910390f35b610085610095565b005b805f8190555050565b5f5481565b5f808154809291906100a690610164565b9190505550565b5f80fd5b5f819050919050565b6100c3816100b1565b81146100cd575f80fd5b50565b5f813590506100de816100ba565b92915050565b5f602082840312156100f9576100f86100ad565b5b5f610106848285016100d0565b91505092915050565b610118816100b1565b82525050565b5f6020820190506101315f83018461010f565b92915050565b7f4e487b71000000000000000000000000000000000000000000000000000000005f52601160045260245ffd5b5f61016e826100b1565b91507fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff82036101a05761019f610137565b5b60018201905091905056fea264697066735822122027cbd9e933004c12c8a29bc76a35040846ac8f3c38cbffcd3b6d470c45cc612f64736f6c63430008150033",
    "nonce": "0x0",
    "chainId": "0x7a69"
}
```

In order to decode the hex, we can use the `cast` commands available with foundry. Let's decode the gas `"gas": "0x31c46"`. This will convert the hex value to the decimal value.

```sh
cast --to-base 0x31c46 dec

203846
```

## Interact with the deployed contract

This is the data of our just deployed contract:

```sh
##### anvil-hardhat
✅  [Success] Hash: 0x15cce041ee3f4778eaa34a23a7f893faba9071d665216e091dd4cf966b6e43f6
Contract Address: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

We need to use the `cast send` and the `cast call` commands, if we check the `--help` this is the arguments to interact with our just deployed contract.

```sh
Arguments:
  [TO]
          The destination of the transaction.

          If not provided, you must use cast send --create.

  [SIG]
          The signature of the function to call

  [ARGS]...
          The arguments of the function to call
```

Let's build the command to call a `view` function:

```sh
cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 "number() returns(uint256)" --rpc-url 127.0.0.1:8545 --account quietRat
```

If the signature is not complete, then you may get a result in hex, use `cast` to decode the value.

```sh
cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 "number()" --rpc-url 127.0.0.1:8545 --account quietRat

0x0000000000000000000000000000000000000000000000000000000000001a14

cast --to-base 0x0000000000000000000000000000000000000000000000000000000000001a14 dec

6676
```

And now let's update the contract state:

```sh
cast send 0x5FbDB2315678afecb367f032d93F642f64180aa3 "setNumber(uint256)" 6676 --rpc-url 127.0.0.1:8545 --account quietRat
```




