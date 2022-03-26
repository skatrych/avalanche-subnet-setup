# avalanche-subnet-setup
Setting up Avalanche network with custom subnet

## Pre-requisites

- Folder ${HOME}/go/src/github.com/ava-labs exists.
- The following git repos are cloned under it:
- https://github.com/ava-labs/avalanche-network-runner
- https://github.com/ava-labs/avalanchego
- ENV VARS defined: $GOPATH, $GOBIN
- Assumption on macOS: $GOPATH == "$HOME/go"
- subnet-cli installed

# Install Avalanche

** Note: tested for MacOS only!!! **

## Install Avalanche network runner
```
# to install
git clone https://github.com/ava-labs/avalanche-network-runner.git
cd ${HOME}/go/src/github.com/ava-labs/avalanche-network-runner
go install -v ./cmd/avalanche-network-runner
```

## Start the network

### To start runner server
```
avalanche-network-runner server \
--log-level debug \
--port=":8080" \
--grpc-gateway-port=":8081"
```

### To ping the runner server:
```
curl -X POST -k http://localhost:8081/v1/ping -d ''

# or
avalanche-network-runner ping \
--log-level debug \
--endpoint="0.0.0.0:8080"
```

## To start Nodes
```
avalanche-network-runner control start \
--log-level debug \
--endpoint="0.0.0.0:8080" \
--avalanchego-path ${HOME}/go/src/github.com/ava-labs/avalanchego/build/avalanchego \
--whitelisted-subnets="24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1"

# or

# replace with your local path
curl -X POST -k http://localhost:8081/v1/control/start -d '{"execPath":"/Users/sergii/go/src/github.com/ava-labs/avalanchego/build/avalanchego","whitelistedSubnets":"24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1","logLevel":"INFO"}'

# Response 
...
node1: node ID "NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg", URI "http://127.0.0.1:42835"
node2: node ID "NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ", URI "http://127.0.0.1:30491"
node3: node ID "NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN", URI "http://127.0.0.1:44838"
node4: node ID "NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu", URI "http://127.0.0.1:45155"
node5: node ID "NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5", URI "http://127.0.0.1:53805"
```


We will use RPC calls to one of the Node endpoints from the list above.
I will use Node1: http://127.0.0.1:42835

# Create new custom subnet

## Produce & use correct subnet binary
First of all, we have to compile the binary for our new subnet nodes and move it into plugins directory of avalanchego project.

Here we go step by step.

### Build vm binary

```
cd $GOPATH/src/github.com/ava-labs/subnet-evm
# assuming that "srEXiWaHuhNyGwPUi444Tu47ZEDwxTWrbQiuD7FmgSAQ6X7Dy" will be our vm-id
./scripts/build.sh build/srEXiWaHuhNyGwPUi444Tu47ZEDwxTWrbQiuD7FmgSAQ6X7Dy
```

The output of the command was like that:

```
Using branch: master
Building Subnet EVM Version: v0.2.0; GitCommit: b60b3e0248603e3f820df6ca40fc6d98728d6abf
# gopkg.in/olebedev/go-duktape.v3
In file included from _cgo_export.c:4:
debugger.go:23:13: warning: unused function '_duk_debugger_attach' [-Wunused-function]
```

### Move vm binary into avalanchego dir

```
# assuming that subnet-vm and avalanchego are in the same dir
mv build/srEXiWaHuhNyGwPUi444Tu47ZEDwxTWrbQiuD7FmgSAQ6X7Dy ../avalanchego/build/plugins/
```

## create file with private key under path: .subnet-cli.pk (WITHOUT 0x)
The private key for the ewoq address (0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC) is 0x56289e99c94b6912bfc12adc093c9b51124f0dc54ac7a766b2bc5ccf558d8027.

```
echo "56289e99c94b6912bfc12adc093c9b51124f0dc54ac7a766b2bc5ccf558d8027" > .subnet-cli.pk
```

## Create subnet with subnet-cli wizard command
```
subnet-cli wizard \
--node-ids=NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg,NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ,NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN,NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu,NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5 \
--vm-genesis-path=my-genesis.json \
--vm-id=srEXiWaHuhNyGwPUi444Tu47ZEDwxTWrbQiuD7FmgSAQ6X7Dy \
--public-uri=http://127.0.0.1:42835 \
--chain-name=subnetevm

# Response:

waiting for validator 7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg to start validating 24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1...(could take a few minutes)
waiting for validator MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ to start validating 24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1...(could take a few minutes)
waiting for validator NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN to start validating 24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1...(could take a few minutes)
waiting for validator GWPcbFJZFfZreETSoWjPimr846mXEKCtu to start validating 24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1...(could take a few minutes)
waiting for validator P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5 to start validating 24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1...(could take a few minutes)


2022-03-26T11:04:39.639+0100	info	client/p.go:497	creating blockchain	{"subnetId": "24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1", "chainName": "sergiivm", "vmId": "speJ74oCutpKqwcovArGTR59htxo4ZATEhxuUSXcJnoZAb4RT", "createBlockchainTxFee": 100000000}
created blockchain "277HMm3iqpGx99FeGnWMkKbzQQku35aLz52mjkFJzmWDgdFuat" (took 1.953167ms)

*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| PRIMARY P-CHAIN ADDRESS | P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p                                                                                                                             |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| TOTAL P-CHAIN BALANCE   | 29,999,999.8950000 $AVAX                                                                                                                                                    |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| URI                     | http://127.0.0.1:42835                                                                                                                                                      |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| NETWORK NAME            | network-1337                                                                                                                                                                |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| SUBNET VALIDATORS       | [7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN GWPcbFJZFfZreETSoWjPimr846mXEKCtu P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5] |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| SUBNET ID               | 24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1                                                                                                                          |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| BLOCKCHAIN ID           | 277HMm3iqpGx99FeGnWMkKbzQQku35aLz52mjkFJzmWDgdFuat                                                                                                                          |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| CHAIN NAME              | sergiivm                                                                                                                                                                    |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| VM ID                   | speJ74oCutpKqwcovArGTR59htxo4ZATEhxuUSXcJnoZAb4RT                                                                                                                           |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| VM GENESIS PATH         | my-genesis.json                                                                                                                                                             |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
```

It means that subnet's RPC URL: http://localhost:42835/ext/bc/277HMm3iqpGx99FeGnWMkKbzQQku35aLz52mjkFJzmWDgdFuat/rpc 

Example: RPC URL will be used in MetaMask and Truffle.

## Whitelist our new Subnet on the validator nodes

Execute restart command for the node *with new --whitelisted-subnets value*

```
# Assuming we had SUBNET ID == 24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1
avalanche-network-runner control restart-node \
--request-timeout=3m \
--log-level debug \
--endpoint="0.0.0.0:8080" \
--node-name node1 \
--avalanchego-path /Users/sergii/go/src/github.com/ava-labs/avalanchego/build/avalanchego \
--whitelisted-subnets="24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1"
```

*Note: not 100% sure if we must restart all the nodes, but I did it for all 5 nodes from node1 ... to ... node5*

## Test:
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1642680522936}' http://localhost:42835/ext/bc/24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1/rpc ???

### Get list of Blockchains
```
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"platform.getBlockchains",
    "params" :{}
}' -H 'content-type:application/json;' 127.0.0.1:42835/ext/P

# Response:
{"jsonrpc":"2.0","result":{"blockchains":[{"id":"277HMm3iqpGx99FeGnWMkKbzQQku35aLz52mjkFJzmWDgdFuat","name":"sergiivm","subnetID":"24tZhrm8j8GCJRE9PomW8FaeqbgGS4UAQjJnqqn8pq5NwYSYV1","vmID":"speJ74oCutpKqwcovArGTR59htxo4ZATEhxuUSXcJnoZAb4RT"},{"id":"BR28ypgLATNS6PbtHMiJ7NQ61vfpT27Hj8tAcZ1AHsfU5cz88","name":"C-Chain","subnetID":"11111111111111111111111111111111LpoYY","vmID":"mgj786NP7uDwBCcq6YwThhaN8FLyybkCa4zBWTQbNgmK6k9A6"},{"id":"qzfF3A11KzpcHkkqznEyQgupQrCNS6WV6fTUTwZpEKqhj1QE7","name":"X-Chain","subnetID":"11111111111111111111111111111111LpoYY","vmID":"jvYyfQTxGMJLuGWa55kdP2p2zSUYsQ5Raupu4TW34ZAUBAbtq"}]},"id":1}
```

### Get status of our new Subnet
```
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"platform.getBlockchainStatus",
    "params" :{
        "blockchainID":"277HMm3iqpGx99FeGnWMkKbzQQku35aLz52mjkFJzmWDgdFuat"
    }
}' -H 'content-type:application/json;' 127.0.0.1:42835/ext/P

# Response:
{"jsonrpc":"2.0","result":{"status":"Created"},"id":1}
```


# Alternative way - automated and simplified
### It did work for me once. Next day i was not able to use it. Basically, you have to kill that instance of avalanche and run it again.

```
# Clone subnet env git repository
git clone https://github.com/ava-labs/subnet-evm

# All in one run script
# where 1.7.8 is the version of avalanchego
# and 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC is default admin address with plenty of tokens on the balance.
cd subnet-evm
./scripts/run.sh 1.7.8 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC
where 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC is default admin user with a lot of coins on the balance.

# In the end we must see something like 

Logs Directory: /var/folders/0z/r8b94v_56v9fc6xk_x2xr2l00000gn/T/runnerlogs884911765
PID: 25299

EVM Chain ID: 99999
Funded Address: 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC
RPC Endpoints:
- http://localhost:49832/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/rpc
- http://localhost:49834/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/rpc
- http://localhost:49836/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/rpc
- http://localhost:49838/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/rpc
- http://localhost:49840/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/rpc

WS Endpoints:
- ws://localhost:49832/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/ws
- ws://localhost:49834/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/ws
- ws://localhost:49836/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/ws
- ws://localhost:49838/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/ws
- ws://localhost:49840/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/ws

MetaMask Quick Start:
Funded Address: 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC
Network Name: Local EVM
RPC URL: http://localhost:49832/ext/bc/2cZqrPoosCsYZ4qjURSccae1nH8yHvSXB4uBHVAp8Ub4AgdDHm/rpc
Chain ID: 99999
Curreny Symbol: LEVM
```

By scrolliing back through the long output of run.sh script, we will find the following important details:
```
Node IDs
node1: node ID "NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg", URI "http://localhost:49832"
node2: node ID "NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ", URI "http://localhost:49834"
node3: node ID "NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN", URI "http://localhost:49836"
node4: node ID "NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu", URI "http://localhost:49838"
node5: node ID "NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5", URI "http://localhost:49840"

importing genesis key and funds to the user in all nodes...
funded P-chain: address "P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p", balance 30000000000000000 $AVAX in "node1"
funded P-chain: address "P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p", balance 30000000000000000 $AVAX in "node2"
funded P-chain: address "P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p", balance 30000000000000000 $AVAX in "node3"
funded P-chain: address "P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p", balance 30000000000000000 $AVAX in "node4"
funded P-chain: address "P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p", balance 30000000000000000 $AVAX in "node5"
```

### Now we have avalanche go with first subnet up and running

# Alternative 2 (step by step instead of wizzard)

After avalanche network runner server is running and validator nodes are up: instead of subnet-cli wizard, we will use subnet-cli step by step.

### Assumption:
Nodes list
- node1: node ID "NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg", URI "http://127.0.0.1:33933"
- node2: node ID "NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ", URI "http://127.0.0.1:39522"
- node3: node ID "NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN", URI "http://127.0.0.1:22974"
- node4: node ID "NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu", URI "http://127.0.0.1:37085"
- node5: node ID "NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5", URI "http://127.0.0.1:37578"

Private key (no 0x in the prefix) is stored in the file .subnet-cli.pk in current directory.


```
subnet-cli create subnet \
--private-key-path=.subnet-cli.pk \
--public-uri=http://localhost:33933
```
#### Response
```
...
created subnet "p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz" (took 1.000933676s)
(subnet must be whitelisted beforehand via --whitelisted-subnets flag!)

*-------------------------*---------------------------------------------------*
| PRIMARY P-CHAIN ADDRESS | P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p   |
*-------------------------*---------------------------------------------------*
| TOTAL P-CHAIN BALANCE   | 29,999,999.9000000 $AVAX                          |
*-------------------------*---------------------------------------------------*
| URI                     | http://localhost:33933                            |
*-------------------------*---------------------------------------------------*
| NETWORK NAME            | network-1337                                      |
*-------------------------*---------------------------------------------------*
| CREATED SUBNET ID       | p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz |
*-------------------------*---------------------------------------------------*
```

#### IMPORTANT!!! subnet ID (see above, p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz) must be whitelisted before using it.

## Adding subnet-validators to our subnet

```
subnet-cli add subnet-validator \
--private-key-path=.subnet-cli.pk \
--public-uri=http://localhost:33933 \
--node-ids="NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg" \
--subnet-id="p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz"
```
#### Response:
```
added 7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg to subnet p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz validator set (took 1.001485836s)

waiting for validator 7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg to start validating p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz...(could take a few minutes)
*-------------------------*---------------------------------------------------*
| PRIMARY P-CHAIN ADDRESS | P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p   |
*-------------------------*---------------------------------------------------*
| TOTAL P-CHAIN BALANCE   | 29,999,999.8990000 $AVAX                          |
*-------------------------*---------------------------------------------------*
| URI                     | http://localhost:33933                            |
*-------------------------*---------------------------------------------------*
| NETWORK NAME            | network-1337                                      |
*-------------------------*---------------------------------------------------*
| NODE IDs                | [7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg]               |
*-------------------------*---------------------------------------------------*
| SUBNET ID               | p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz |
*-------------------------*---------------------------------------------------*
| VALIDATE START          | 2022-03-26T15:10:24+01:00                         |
*-------------------------*---------------------------------------------------*
| VALIDATE END            | 2022-09-07T06:00:00+02:00                         |
*-------------------------*---------------------------------------------------*
| VALIDATE WEIGHT         | 1,000                                             |
*-------------------------*---------------------------------------------------*
```

### Repeating command above for the other node ids
I've added 4 validators for the subnet.

## Create blockchain on this subnet

```
subnet-cli create blockchain \
--private-key-path=.subnet-cli.pk \
--public-uri=http://localhost:33933 \
--subnet-id="p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz" \
--chain-name=serknewvm \
--vm-id=tGas3T58KzdjLHhBDMnH2TvrddhqTji5iZAMZ3RXs2NLpSnhH \
--vm-genesis-path=my-genesis.json
```
#### Response:
```
2022-03-26T15:25:59.475+0100	info	client/p.go:497	creating blockchain	{"subnetId": "p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz", "chainName": "serknewvm", "vmId": "tGas3T58KzdjLHhBDMnH2TvrddhqTji5iZAMZ3RXs2NLpSnhH", "createBlockchainTxFee": 100000000}
created blockchain "kcGi8DrMiupKtqsssMcnvnBANP8UkmB2qZ6uZe6HoJ5rpSAsh" (took 2.396061ms)

*-------------------------*---------------------------------------------------*
| PRIMARY P-CHAIN ADDRESS | P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p   |
*-------------------------*---------------------------------------------------*
| TOTAL P-CHAIN BALANCE   | 29,999,999.8960000 $AVAX                          |
*-------------------------*---------------------------------------------------*
| URI                     | http://localhost:33933                            |
*-------------------------*---------------------------------------------------*
| NETWORK NAME            | network-1337                                      |
*-------------------------*---------------------------------------------------*
| SUBNET ID               | p433wpuXyJiDhyazPYyZMJeaoPSW76CBZ2x7wrVPLgvokotXz |
*-------------------------*---------------------------------------------------*
| CREATED BLOCKCHAIN ID   | kcGi8DrMiupKtqsssMcnvnBANP8UkmB2qZ6uZe6HoJ5rpSAsh |
*-------------------------*---------------------------------------------------*
| CHAIN NAME              | serknewvm                                         |
*-------------------------*---------------------------------------------------*
| VM ID                   | tGas3T58KzdjLHhBDMnH2TvrddhqTji5iZAMZ3RXs2NLpSnhH |
*-------------------------*---------------------------------------------------*
| VM GENESIS PATH         | my-genesis.json                                   |
*-------------------------*---------------------------------------------------*
```

## Whitelist our subnet ID and restart the node

Follow the same steps as above in the Whitelist section.