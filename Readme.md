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

- Fund the Ethereum address that your Chainlink node uses. You can find the address in the node Operator GUI under the **Key Management** configuration. The address of the node is the `Regular` type. You can obtain test ETH from several [faucets](/resources/link-token-contracts). For this tutorial to work, you will have to fund the node's Ethereum address with Sepolia ETH. Here is an example:

![](/public/images/key-management.jpg)

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
