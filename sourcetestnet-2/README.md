# sourcetestnet-2

![c11](https://static.wixstatic.com/media/80368b_b2c7b9f0d8614798bd9df0111903155a~mv2.png/v1/fill/w_624,h_108,al_c,q_85,usm_0.66_1.00_0.01/source%20logo%20final%20hrzn.webp)

## If you are reusing a testnet box, do this first

1. Stop your node
2. Build `v2.0.0` tag of `sourced`
3. Reset using `sourced unsafe-reset-all` or `sourced tendermint unsafe-reset-all --home ~/.source` (the `--home` flag is required)
4. Remove genesis `rm $HOME/.source/config/genesis.json`
5. Remove gentxs `rm -r $HOME/.source/config/gentx/`
6. If you are using cosmovisor, remove symlink: `rm $HOME/.source/cosmovisor/current`
7. Then remove upgrades dir `rm -r $HOME/.source/cosmovisor/upgrades && mkdir $HOME/.source/cosmovisor/upgrades`
8. Move `sourced` to genesis bin: `cp $HOME/go/bin/sourced $DAEMON_HOME/cosmovisor/genesis/bin`
9. Remove any `upgrade-info` file in the `data` dir: `rm $HOME/.source/data/upgrade-info.json`
10. Check genesis bin is `v2.0.0`: `$DAEMON_HOME/cosmovisor/genesis/bin/sourced version`
11. Follow generate gentx as normal below

### Source Testnet Set up

For full Source Chain Documentation and testnet set up click: [HERE](https://docs.sourceprotocol.io/source-chain-documentation/introduction)

### Minimum hardware requirements
4GB RAM
250GB of disk space
1.4 GHz amd64 CPU

#### Install Go

**Prerequisites:** Make sure to have [Golang >=1.18](https://golang.org/).
```bash
wget https://golang.org/dl/go1.18.2.linux-amd64.tar.gz
```
```bash
sudo tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz
```
```bash
sudo chown -R <YOUR_USERNAME>:<YOUR_USERNAME> /usr/local/go
```

You need to ensure your gopath configuration is correct. If the following **'make'** step does not work then you might have to add these lines to your .profile or .zshrc in the users home folder:

```sh
nano ~/.profile
```

```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

Source update .profile

```sh
source .profile
```


### Install Ignite CLI/Starport

[IGNITE CLI Install Docs](https://docs.ignite.com/guide/install.html)

```bash
sudo curl https://get.ignite.com/cli! | sudo bash
```

### Clone Source Chain Repo

```bash
git clone https://github.com/Source-Protocol-Cosmos/source.git
```
```bash
cd source
```
```bash
git checkout v2.0.0
```

### Compile sourced Binary

```bash
cd ~/source
ignite chain build
```


### Initialize the Source directories and create the local genesis file with the correct chain-id:

```bash
sourced init <moniker-name> --chain-id=sourcetestnet-2
```

### Create a local key pair (or add existing key):

```sh
sourced keys add <key-name>
```

### Download Genesis File

```bash
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/testnets/master/sourceestnet-2/genesis.json > ~/.source/config/genesis.json
```

**Genesis sha256**

```bash
sha256sum ~/.source/config/genesis.json
# 6b3ae7a2b7d652d44a404ab462ed258b786b20fc57e0d2e6d02c41e8d2ec8d68
```

### Seed nodes to add to config.toml


```bash
nano ~/.source/config/config.toml
```

```
# Comma separated list of nodes to keep persistent connections to persistent_peers = 
"b322e27f7d28f025a3ddecf80bde32799814c7b3@44.234.121.117:26656"
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
sourced init <moniker-name> --chain-id=sourcetestnet-2
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
sourced gentx <key-name> 9000000000usource --chain-id=sourcetestnet-2
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
--chain-id sourcetestnet-2 \
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
  > cp ~/.source/config/gentx/gentx*.json ./sourcetestnet-2/gentx/
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
