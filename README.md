# testnets

![c11](https://static.wixstatic.com/media/80368b_b2c7b9f0d8614798bd9df0111903155a~mv2.png/v1/fill/w_624,h_108,al_c,q_85,usm_0.66_1.00_0.01/source%20logo%20final%20hrzn.webp)

### Source Testnet Set up

**Source Testnet is now using "sourcetest-1" and v3.0.0 of the Source Blockchain. "Sourcechain-testnet" has been shutdown.**

If you are reusing your testnet box, you must first remove the old data. Validators from the previous testnet will have usource available to start a new node, use your same wallet:

```bash
sudo systemctl stop sourced && \
cd $HOME && \
rm -rf .source && \
rm -rf source && \
rm -rf $(which sourced)
```


For full Source Chain Documentation and testnet set up click: [HERE](https://docs.sourceprotocol.io/source-chain-documentation/introduction)

### Minimum hardware requirements
8GB RAM
250GB of disk space
1.4 GHz amd64 CPU

#### Install Go

**Prerequisites:** Make sure to have [Golang >=1.19](https://golang.org/).
```bash
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

### Clone Source Chain Repo

```bash
git clone https://github.com/Source-Protocol-Cosmos/source.git
```

### Compile sourced Binary

```bash
cd ~/source
git fetch
git checkout v3.0.1
make build && make install
```


### Initialize the Source directories and create the local genesis file with the correct chain-id:

```bash
sourced init <moniker-name> --chain-id=sourcetest-1
```

### Create a local key pair (or add existing key):

```sh
sourced keys add <walletName>
    or
sourced keys add <walletName> --recover
```

### Download Genesis File

```bash
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/testnets/master/sourcetest-1/genesis.json > ~/.source/config/genesis.json
```

**Genesis sha256**

```bash
sha256sum ~/.source/config/genesis.json
# c8b8e28f1cc2c6bb708d963146842da9e367874267d90ab99a13a6bd736d5682
```

### Seed nodes to add to config.toml


```bash
nano ~/.source/config/config.toml
```

```
# Comma separated list of nodes to keep persistent connections to persistent_peers = 
"ace839c852739d1ea6e3675d30380fe085c1c23a@52.26.226.21:26656,8145d4d13511e7f89dbd257f51ed5d076941f12f@164.92.98.12:26656"
```

### Set Minimum Gas Price


```bash
nano ~/.source/config/app.toml
```

```
0.025usource
```

## Setup validator node

Below are the instructions to generate & submit your genesis transaction


### Generate genesis transaction (gentx)

1. Initialize the Source directories and create the local genesis file with the correct chain-id:

```bash
sourced init <moniker-name> --chain-id=sourcetest-1
```

2. Create a local key pair (skip this step if you already have a key):

```sh
sourced keys add <key-name>
```

3. Add your account to your local genesis file with a given amount and the key you just created. Use only `10000000000usource`, other amounts will be ignored.

```bash
sourced add-genesis-account $(sourced keys show <key-name> -a) 10000000000usource
```

4. Create the gentx, use only `9000000000usource`:

```bash
sourced gentx <key-name> 9000000000usource --chain-id=sourcetest-1
```

If all goes well, you will see a message similar to the following:

```bash
Genesis transaction written to "/home/user/.source/config/gentx/gentx-******.json"
```

5. Start the chain
```bash
sourced start
```
6. Create Validator
```bash
sourced tx staking create-validator \
--amount 1000000000usource \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--details "validators write bios too" \
--pubkey=$(sourced tendermint show-validator) \
--moniker “<key-name>” \
--chain-id sourcetest-1 \
--gas-prices 0.025usource \
--from <key-name>
```



### Submit genesis transaction

- Fork [the testnets repo](https://github.com/Source-Protocol-Cosmos/testnets) into your Github account

- Clone your repo using

  ```bash
  git clone https://github.com/<your-github-username>/testnets
  ```

- Copy the generated gentx json file to `<repo_path>/testnets/gentx/`

  ```sh
  > cd testnet-genesis
  > cp ~/.source/config/gentx/gentx*.json ./sourcechain-testnet/gentx/
  ```

- Commit and push to your repo
- Create a PR onto https://github.com/Source-Protocol-Cosmos/testnets
- Only PRs from individuals / groups with a history successfully running nodes will be accepted. This is to ensure the network successfully starts on time.


### Running in production

**Note, we'll be going through some upgrades for this testnet. Consider using [Cosmovisor](https://github.com/cosmos/cosmos-sdk/tree/master/cosmovisor) to make your life easier.**

Create a systemd file for your Source service:
```bash
sudo nano /etc/systemd/system/sourced.service
```   
Copy and paste the following and update <YOUR-USERNAME>:
```bash
Description=Source daemon
After=network-online.target

[Service]
User=<YOUR_USERNAME>
ExecStart=/home/<YOUR-USERNAME>/go/bin/sourced start --home /home/<YOUR-USERNAME>/.source
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
This assumes $HOME/.source to be your directory for config and data. Your actual directory locations may vary.

Enable and start the new service:
```bash
sudo systemctl enable sourced
```
```bash   
sudo systemctl start sourced
```   
Check status:
```bash
sourced status
```
Check logs:
```bash
journalctl -u sourced -f
```   
