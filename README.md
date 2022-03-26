# avalanche-subnet-setup
Setting up Avalanche network with custom subnet

# Install Avalanche

## Install Avalanche network runner
```
# to install
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

# Create new custom subnet

## generate VM ID
```
subnet-cli create VMID sergiivm
# Response:
# created a new VMID speJ74oCutpKqwcovArGTR59htxo4ZATEhxuUSXcJnoZAb4RT from sergiisvm
```

## create file with private key under path: .subnet-cli.pk (WITHOUT 0x)

```
subnet-cli wizard \
--node-ids=NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg,NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ,NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN,NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu,NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5 \
--vm-genesis-path=my-genesis.json \
--vm-id=speJ74oCutpKqwcovArGTR59htxo4ZATEhxuUSXcJnoZAb4RT \
--public-uri=http://localhost:8080 \
--chain-name=sergiivm

# Response:

waiting for validator 7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg to start validating NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8...(could take a few minutes)
waiting for validator MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ to start validating NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8...(could take a few minutes)
waiting for validator NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN to start validating NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8...(could take a few minutes)
waiting for validator GWPcbFJZFfZreETSoWjPimr846mXEKCtu to start validating NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8...(could take a few minutes)
waiting for validator P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5 to start validating NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8...(could take a few minutes)
```

2022-03-25T20:19:10.942+0100	info	client/p.go:497	creating blockchain	{"subnetId": "NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8", "chainName": "sergiisvm", "vmId": "speJ74oCuF4s9jxavSYsYtCfodpDTY2kMKbxbxbur1FrwZ3v3", "createBlockchainTxFee": 100000000}
created blockchain "2GxiHHt5RnU7TkS2SVK8X9RtWpKUGWidMaX3ibk7RsD3NDzuit" (took 2.092678ms)

*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| PRIMARY P-CHAIN ADDRESS | P-custom18jma8ppw3nhx5r4ap8clazz0dps7rv5u9xde7p                                                                                                                             |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| TOTAL P-CHAIN BALANCE   | 29,999,999.5850000 $AVAX                                                                                                                                                    |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| URI                     | http://localhost:49832                                                                                                                                                      |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| NETWORK NAME            | network-1337                                                                                                                                                                |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| SUBNET VALIDATORS       | [7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN GWPcbFJZFfZreETSoWjPimr846mXEKCtu P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5] |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| SUBNET ID               | NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8                                                                                                                           |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| BLOCKCHAIN ID           | 2GxiHHt5RnU7TkS2SVK8X9RtWpKUGWidMaX3ibk7RsD3NDzuit                                                                                                                          |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| CHAIN NAME              | sergiisvm                                                                                                                                                                   |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| VM ID                   | speJ74oCuF4s9jxavSYsYtCfodpDTY2kMKbxbxbur1FrwZ3v3                                                                                                                           |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*
| VM GENESIS PATH         | my-genesis.json                                                                                                                                                             |
*-------------------------*-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------*

It means that subnet's RPC: RPC URL: http://localhost:49832/ext/bc/NhTW2m1dPjc4qKFgvCr56xpSyTmVvBXUFBdT21XSmRhw589e8/rpc


# Alternative way - automated and simplified
### It did work for me once. Next day i was not able to 

- git clone https://github.com/ava-labs/subnet-evm
- cd subnet-evm
- ./scripts/run.sh 1.7.8 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC
where 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC is default admin user with a lot of coins on the balance.

In the end we must see something like 

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

### Now we have avalanche go with first subnet up and running


