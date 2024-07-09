# ZeroGravity (0G) is the first infinitely scalable and decentralized data availability layer with a built-in general-purpose storage layer.




# Hardware requirements

```
- Memory: 64 GB RAM
- CPU: 8 cores
- Disk: 1 TB NVME SSD
- Bandwidth: 100mbps Gbps for Download / Upload
```

---

## Server Timezone Configuration

Make sure your server timezone configuration is UTC. Check your current timezone by running `timedatectl`

---


# Installation guide

## 1. Install required packages

```bash
sudo apt update && \
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

## 2. Install Go

```bash
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

## 3. Install 0gchaind via CLI

```bash
git clone -b v0.2.3 https://github.com/0glabs/0g-chain.git
./0g-chain/networks/testnet/install.sh
source ~/.profile
```

## 4. Set Chain ID

You can find the current Chain ID [here](https://docs.0g.ai/0g-doc/run-a-node/testnet-configuration)

```bash
0gchaind config chain-id zgtendermint_16600-2
```

## 5. Initialize Node

We need to initialize the node to create all the necessary validator and node configuration files:
```bash
0gchaind init <your_validator_name> --chain-id zgtendermint_16600-2
```

__Note: the validator name can only contain ASCII characters.__

By default, the init command creates config and data folder under `~/.0gchain(i.e $HOME)`. In the config directory, the most important files for configuration are `app.toml` and `config.toml`.

## 6. Genesis & Seeds

### Copy the Genesis File

Check the `genesis.json` file and copy it over to the config directory `$HOME/.0gchain/config/genesis.json`. This is a genesis file with the chain-id and genesis accounts balances.

```bash
sudo apt install -y unzip wget
rm ~/.0gchain/config/genesis.json
wget -P ~/.0gchain/config https://github.com/0glabs/0g-chain/releases/download/v0.2.3/genesis.json
```

Then verify the correctness of the genesis configuration file:

```bash
0gchaind validate-genesis
```
### Add Seed Nodes

Your node needs to know how to find [peers](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#peers). You’ll need to add healthy [seed nodes](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#seed) to `$HOME/.0gchain/config/config.toml`.
The format of the `config.toml` file is as follows:

```toml
#######################################################
###           P2P Configuration Options             ###
#######################################################
[p2p]

# ...

# Comma separated list of seed nodes to connect to
seeds = "<node-id>@<ip>:<p2p port>"
```

Seeds
`81987895a11f6689ada254c6b57932ab7ed909b6@54.241.167.190:26656,010fb4de28667725a4fef26cdc7f9452cc34b16d@54.176.175.48:26656,e9b4bc203197b62cc7e6a80a64742e752f4210d5@54.193.250.204:26656,68b9145889e7576b652ca68d985826abd46ad660@18.166.164.232:26656`

### Add Persistent Peers
You can set the `persistent_peers` field in `$HOME/.0gchain/config/config.toml` to specify peers that your node will maintain persistent connections with.

# Start Testnet

Start the node and sync up to the latest block height. Note that the first time you start the sync up, it may take longer time to run.

```0gchaind start```

### Garbage Collection Optimization
To maximize sync speed for validators and other network providers that are running pruning nodes, the following settings are recommended:
- Start 0gchaind process with environment variable and value `GOGC=900`; this instructs the golang garbage collector to wait until the heap has grown to 9x it's initial allocated size before running garbage collection
- Start 0gchaind process with environment variable `GOMEMLIMIT` set to 66% of the total memory available to the 0gchaind process (e.g. `GOMEMLIMIT=40GB` for a node with 64 GB of memory) to ensure garbage collection runs whenever 66% of the total memory is used
---
# Make sure you've synced your node to the latest block height before running the following steps.
---
# Create Validator

You could either create a new account or import from an existing key. To create a new account:
```
0gchaind keys add <key_name> --eth
```

Here if you want to get the public address which starts with 0x, you could first run the following command to get your key’s private key.
```
0gchaind keys unsafe-export-eth-key <key_name>
```

Then import the returned private key to a wallet (Metamask for example) to get the public address.

As a next step, you must acquire some testnet tokens either by wallet transfer or requesting on the [faucet](https://faucet.0g.ai/) before submitting your validator account address.

```bash
0gchaind tx staking create-validator \
  --amount=<staking_amount>ua0gi \
  --pubkey=$(0gchaind tendermint show-validator) \
  --moniker="<your_validator_name>" \
  --chain-id=zgtendermint_16600-2 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<key_name> \
  --gas=auto \
  --gas-adjustment=1.4
```
Check that it is in the validator set:
```
0gchaind q staking validators -o json --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '.tokens + " - " + .description.moniker' | sort -gr | nl
```
>Only top 125 staked validators will be selected as active validators.

By any chance your validator is put in jail, use this command to unjail it
```
0gchaind tx slashing unjail --from <key_name> --gas=500000 --gas-prices=99999neuron -y
```






















