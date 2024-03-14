# Chainlink
#### Prerequisite
* Docker
* Docker-compose
* Ubuntu 22.04 

## Docker
#### Uninstall old versions

Before you can install Docker Engine, you need to uninstall any conflicting packages.

Distro maintainers provide unofficial distributions of Docker packages in
APT. You must uninstall these packages before you can install the official
version of Docker Engine.

The unofficial packages to uninstall are:

- `docker.io`
- `docker-compose`
- `docker-compose-v2`
- `docker-doc`
- `podman-docker`

```console
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
#### Install using the `apt` repository
Before you install Docker Engine for the first time on a new host machine, you
need to set up the Docker repository. Afterward, you can install and update
Docker from the repository.

1. Set up Docker's `apt` repository.

   ```bash
   # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add the repository to Apt sources:
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
   ```
2. Install the Docker packages.

   To install the latest version, run:

   ```console
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
## Install the Compose plugin

2. Update the package index, and install the latest version of Docker Compose:

    * For Ubuntu run:

        ```console
        sudo apt-get update
        sudo apt install docker-compose
        ```

#### Add your user to the `docker` group.
Append the your user to the docker group to be able to interact with the Docker daemon without needing to use sudo for every Docker command

   ```console
   sudo usermod -aG docker $USER
   ```

####3. Log out and log back in so that your group membership is re-evaluated

## Run the Docker-compose
Be sure you are in .chainlink-sepolia directory
```bash
docker-compose up -d
```
You can now connect to your Chainlink node's UI interface by navigating to [http://localhost:6688](http://localhost:6688). Use the API
credentials you set up earlier to log in.

If you are using a VPS, you can create an [SSH tunnel](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/) to your node for `6688:localhost:6688` to enable connectivity to the GUI. Typically this is done with `ssh -i $KEY $USER@$REMOTE-IP -L 6688:localhost:6688 -N`.

# ETH/USD Price Feed
## Requirements

- Fund the Ethereum address that your Chainlink node uses. You can find the address in the node Operator GUI under the **Key Management** configuration. 

## Setup your Operator contract

### Deploy your own Operator contract

1. Go to Remix and open the [`Operator.sol` smart contract](https://remix.ethereum.org/#url=https://docs.chain.link/samples/ChainlinkNodes/Operator.sol).

2. On the **Compile** tab, click the **Compile** button for `Operator.sol`. Remix automatically selects the compiler version and language from the `pragma` line unless you select a specific version manually.

3. On the **Deploy and Run** tab, configure the following settings:

   - Select "Injected Provider" as your **Environment**. The Javascript VM environment cannot access your oracle node. Make sure your Metamask is connected to Sepolia testnet.
   - Select the "Operator" contract from the **Contract** menu.
   - Copy the LINK token contract address for the network you are using and paste it into the `LINK` field next to the **Deploy** button. For Sepolia, you can use this address:

     ```text Sepolia
     0x779877A7B0D9E8603169DdbD7836e478b4624789
     ```

   - Copy the _Admin wallet address_ into the `OWNER` field.

   ![](https://github.com/smartcontractkit/documentation/blob/main/public/images/chainlink-nodes/node-operators/deploy-operator.jpg?raw=true)

4. Click **transact**. MetaMask prompts you to confirm the transaction.

5. If the transaction is successful, a new address displays in the **Deployed Contracts** section.

   ![Screenshot showing the newly deployed contract.](https://github.com/smartcontractkit/documentation/blob/main/public/images/chainlink-nodes/node-operators/operator-deployed.jpg?raw=true)

5. Keep note of the Operator contract address. You need it later for your consuming contract.

### Whitelist your node address in the Operator contract

1. In the Chainlink node GUI, find and copy the address of your chainlink node.

1. In Remix, call the `setAuthorizedSenders` function with the address of your node. Note the function expects an array.

   ![A screenshot showing all of the fields for the deployed contract in Remix.](https://github.com/smartcontractkit/documentation/blob/main/public/images/chainlink-nodes/node-operators/operator-authorizedsenders.jpg?raw=true)

1. Click the `transact` function to run it. Approve the transaction in MetaMask and wait for it to confirm on the blockchain.

1. Call `isAuthorizedSender` function with the address of your node to verify that your chainlink node address can call the operator contract. The function must return `true`.

   ![A screenshot showing Chainlink node whitelisted in Remix.](https://github.com/smartcontractkit/documentation/blob/main/public/images/chainlink-nodes/node-operators/operator-isauthorizedsender.jpg?raw=true)

   ## Add a job to the node

1. In the Chainlink Operator UI on the **Jobs** tab, click **New Job**.
1. Paste the job specification from above into the text field.

```console
# THIS IS EXAMPLE CODE THAT USES HARDCODED VALUES FOR CLARITY.
# THIS IS EXAMPLE CODE THAT USES UN-AUDITED CODE.
# DO NOT USE THIS CODE IN PRODUCTION.

name = "Get > Uint256 - (TOML)"
schemaVersion = 1
type = "directrequest"
# evmChainID for Sepolia Testnet
evmChainID = "11155111"
# Optional External Job ID: Automatically generated if unspecified
# externalJobID = "b1d42cd5-4a3a-4200-b1f7-25a68e48aad8"
contractAddress = "YOUR_OPERATOR_CONTRACT_ADDRESS"
maxTaskDuration = "0s"
minIncomingConfirmations = 0
observationSource = """
    decode_log   [type="ethabidecodelog"
                  abi="OracleRequest(bytes32 indexed specId, address requester, bytes32 requestId, uint256 payment, address callbackAddr, bytes4 callbackFunctionId, uint256 cancelExpiration, uint256 dataVersion, bytes data)"
                  data="$(jobRun.logData)"
                  topics="$(jobRun.logTopics)"]

    decode_cbor  [type="cborparse" data="$(decode_log.data)"]
    fetch        [type="http" method=GET url="$(decode_cbor.get)" allowUnrestrictedNetworkAccess="true"]
    parse        [type="jsonparse" path="$(decode_cbor.path)" data="$(fetch)"]

    multiply     [type="multiply" input="$(parse)" times="$(decode_cbor.times)"]

    encode_data  [type="ethabiencode" abi="(bytes32 requestId, uint256 value)" data="{ \\"requestId\\": $(decode_log.requestId), \\"value\\": $(multiply) }"]
    encode_tx    [type="ethabiencode"
                  abi="fulfillOracleRequest2(bytes32 requestId, uint256 payment, address callbackAddress, bytes4 callbackFunctionId, uint256 expiration, bytes calldata data)"
                  data="{\\"requestId\\": $(decode_log.requestId), \\"payment\\":   $(decode_log.payment), \\"callbackAddress\\": $(decode_log.callbackAddr), \\"callbackFunctionId\\": $(decode_log.callbackFunctionId), \\"expiration\\": $(decode_log.cancelExpiration), \\"data\\": $(encode_data)}"
                  ]
    submit_tx    [type="ethtx" to="YOUR_OPERATOR_CONTRACT_ADDRESS" data="$(encode_tx)"]

    decode_log -> decode_cbor -> fetch -> parse -> multiply -> encode_data -> encode_tx -> submit_tx
"""

```
1. Replace `YOUR_OPERATOR_CONTRACT_ADDRESS` with the address of your deployed operator contract address from the previous steps.

1. Click **Create Job**. If the node creates the job successfully, a notice with the job number appears.

1. Click the job number to view the job details. You can also find the job listed on the **Jobs** tab in the Node Operators UI. Save the `externalJobID` value because you will need it later to tell your consumer contract what job ID to request from your node.

   ![A screenshot showing the External Job ID.](https://github.com/smartcontractkit/documentation/blob/main/public/images/chainlink-nodes/node-operators/job-id.jpg?raw=true)

## Create a request to your node

After you add jobs to your node, you can use the node to fulfill requests. This section shows what a requester does when they send requests to your node. It is also a way to test and make sure that your node is functioning correctly.

1. Open [ATestnetConsumer.sol in Remix](https://remix.ethereum.org/#url=https://docs.chain.link/samples/APIRequests/ATestnetConsumer.sol).

1. Note that `setChainlinkToken(0x779877A7B0D9E8603169DdbD7836e478b4624789)` is configured for _Sepolia_.

1. On the **Compiler** tab, click the **Compile** button for `ATestnetConsumer.sol`.

1. On the **Deploy and Run** tab, configure the following settings:

   - Select _Injected Provider_ as your environment. Make sure your metamask is connected to Sepolia.
   - Select _ATestnetConsumer_ from the **Contract** menu.

1. Click **Deploy**. MetaMask prompts you to confirm the transaction.

1. Fund the contract by sending LINK to the contract's address. See the [Fund your contract](/resources/fund-your-contract) page for instructions. The address for the `ATestnetConsumer` contract is on the list of your deployed contracts in Remix. You can fund your contract with 1 LINK.

1. After you fund the contract, create a request. Input your operator contract address and the job ID for the `Get > Uint256` job into the `requestEthereumPrice` request method **without dashes**. The job ID is the `externalJobID` parameter, which you can find on your job's definition page in the Node Operators UI.

   ![Screenshot of the requestEthereumPrice function with the oracle address and job ID specified.](https://github.com/smartcontractkit/documentation/blob/main/public/images/chainlink-nodes/node-operators/requestEthereumPrice.jpg?raw=true)

1. Click the **transact** button for the `requestEthereumPrice` function and approve the transaction in Metamask.

1. After the transaction processes, you can see the details for the complete the job run the **Runs** page in the Node Operators UI.

   ![A screenshot of the task link](https://github.com/smartcontractkit/documentation/blob/main/public/images/chainlink-nodes/node-operators/taskList.jpg?raw=true)

1. In Remix, click the `currentPrice` variable to see the current price updated on your consumer contract.

# Credentials and Links
## Grafana
[localhost:3000](http://localhost:3000)

Username: admin

password: grafana

## Prometheus
[localhost:9090](http://localhost:9090)
## Alertmanager
[localhost:9093](http://localhost:9093)

## Explorer Links

[Oracle smart contract address](https://sepolia.etherscan.io/address/0x912a9151F233Ab0c54fF3b59b328318Bd16193FF)
[Chainlink node account address](https://sepolia.etherscan.io/address/0x147F4d88676466F00c45d35D52baea91b68dc07C)