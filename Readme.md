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

(public/images/key-management.jpg)