# Mining Guidance

- [Mining Guidance](#mining-guidance)
  - [Instructions](#instructions)
  - [SGX](#sgx)
  - [Install the Docker Environment](#install-the-docker-environment)
  - [Running the Service](#running-the-service)
    - [Preparing an Account](#preparing-an-account)
      - [Option 1](#option-1)
      - [Option 2](#option-2)
    - [Preparing Tokens](#preparing-tokens)
    - [Startup and Maintenance](#startup-and-maintenance)
      - [Create new profile](#create-new-profile)
      - [Replace owner](#replace-owner)
      - [Start server](#start-server)
      - [Update Device](#update-device)
      - [Exiting the Service (if required)](#exiting-the-service-if-required)
  - [FAQ](#faq)

## Instructions

Before starting, please cofirm on [Intel© Ark](https://ark.intel.com/content/www/us/en/ark.html#@Processors) whether your processor is compatible with [Intel© SGX](https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html).

Then clone the repository:

```bash
git clone https://github.com/deepslabs/deeps-mining-scripts.git
```

## SGX

Inspect your system's SGX support with:

```shell
sudo ./sgx-detect
```

Install the packages required to build the SGX driver:

```shell
sudo apt update
sudo apt install build-essential automake autoconf libtool wget python3 libssl-dev dkms
```

Sample output:

```text
✔  SGX instruction set
  ✔  CPU support
  ✔  CPU configuration
  ✔  Enclave attributes
  ✔  Enclave Page Cache
  SGX features
    ✔  SGX2  ✔  EXINFO  ✘  ENCLV  ✘  OVERSUB  ✔  KSS
    Total EPC size: 16.0GiB
✔  Flexible launch control
  ✔  CPU support
  ？ CPU configuration
  ✔  Able to launch production mode enclave
✔  SGX system software
  ✔  SGX kernel device (/dev/sgx_enclave)
  ✘  libsgx_enclave_common
  ✘  AESM service
  ✔  Able to launch enclaves
    ✔  Debug mode
    ✔  Production mode
    ✔  Production mode (Intel whitelisted)
```

If it displays as `✘ SGX kernel device (/dev/sgx_enclave)`, We should install SGX Environment and restart with:

```shell
sudo chmod +x sgx_enable
sudo ./sgx_enable
sudo reboot
```

## Install the Docker Environment

```shell
# Install the Docker runtime environment
sudo curl -fsSL https://get.docker.com | bash -s docker
sudo systemctl enable docker
sudo systemctl start docker
# Check if the Docker service started correctly
sudo systemctl status docker
# Press Ctrl+C to exit the status view
sudo chmod 666 /var/run/docker.sock
docker version
# Download the docker-compose program
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# Install docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## Running the Service

Once you have confirmed that your machine supports SGX2, you can launch the keyring service. The keyring service obtains events and state from a node service. In the configuration file, it is advisable to use an official node as the data source. Alternatively, you can start a local full node and use it as the data source once synchronization is complete.

### Preparing an Account

Before starting the service, you must create an account to serve as the owner responsible for holding and managing the keyring service.

#### Option 1

Generate an account using the command `docker run -it --rm deepslabs/deeps-node:pre-beta identity generate`.

You will receive an output like this:

```text
Secret seed:      0x71235e1458ce9d140c8b8ded28ccc1e32e62c340aef51a65e1350a387dbe08a6
Public key (hex): 0x0248e7f02dcc9f7061a090b67dede93d7381847e94955aee7996603d2225e9f77e
Account ID:       0x34a5572cb21d34354e3091564d5edc7b791e9d5f
```

`Secret seed` is the account's private key, which can be imported directly into wallets such as MetaMask.
`Account ID` is the account's address.

#### Option 2

Alternatively, you can create an account with MetaMask, because the DeepS account system is Ethereum-compatible.

We recommend MetaMask here, since subsequent operations require interaction with the [DHC dashboard](https://test-dhcs.deeps.fi/testnet).

### Preparing Tokens

Fund your address with some tDPS so that the device can be deployed.

### Startup and Maintenance

#### Create new profile

```shell
./dhc config --network testnet
```

Output:

```text
Info: Generating keyring.toml for testnet...
Success: Generated keyring.toml for testnet
```

#### Replace owner

```shell
./dhc owner 0x34a5572cb21d34354e3091564d5edc7b791e9d5f
```

#### Start server

To start the service and view its logs, use the following commands:

```shell
docker-compose up -d
docker-compose logs -f
```

Wait for the software to start. If any error occurs, consult the [FAQ](#faq).

If the software is running correctly, you will see logs similar to the following:

```text
register sgx: "0x13bec2ac21b038d885d49d8100d307ce7761cf890bbdf25962a0eb2f2ac18101"
```

Log in to [DeepS DHC](https://test-dhcs.deeps.fi/testnet) with your `device_owner` account. Unlisted devices initially appear in the device list.

**All subsequent actions require a MetaMask signature. Verify that the account connected in MetaMask matches the `device_owner` account in your `keyring.toml` file.**

#### Update Device

Go to [DeepS DHC](https://test-dhcs.deeps.fi/testnet) to activate the device. For the first time, you need to vote tokens for it.

![dhc-launch](./images/dhc-launch.png)

For a quick start, stake 100000 tDPS at a time, then click the `Submit` button.

![dhc-submit](./images/dhc-submit.png)

Wait for one epoch, and once the total staked amount reaches the threshold (100000 tDPS), join the service via `Join Service`.

![dhc-join](./images/dhc-join.png)

When the device status changes to `Service`, **congratulations** - the process is complete.

![dhc-joined](./images/dhc-joined.png)

> To check whether the software is running correctly, look for logs like the following:
> `HeartBeat session: 40167, challenge: [...], hash: "0xa746ff7daae0952967cc9eadb38e6627052cd073cf0a319cb8fcb65e0abdabef"`

#### Exiting the Service (if required)

Note: The system penalizes malicious nodes by deducting their staked tokens. To avoid financial losses caused by an irregular exit, follow the process below.

Exit the service by selecting `Exit Service`:

![dhc-exit](./images/dhc-exit.png)

After selecting `Exit Service`, you must wait one epoch before you can select `Remove Device`. No operations are available during this period.

Finally, stop your keyring service:

```shell
docker compose down
```

## FAQ

Refer to the [troubleshooting documentation](https://docs.deeps.fi/node-operations/troubleshooting).
