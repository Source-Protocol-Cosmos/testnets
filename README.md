# testnet-genesis


Source Chain Testnet Set up

### Clone Source Chain Repo

```bash
   git clone -b testnet https://github.com/Source-Protocol-Cosmos/source.git
```

### Download Genesis File

```bash
   curl -s  https://raw.githubusercontent.com/SourceNexxus/testnet-genesis/genesis.json > ~/.source/config/genesis.json
```

### Seed nodes


```bash
nano ~/.source/config/config.toml
```

```
# Comma separated list of nodes to keep persistent connections to persistent_peers = 
"6ca675f9d949d5c9afc8849adf7b39bc7fccf74f@164.92.98.17:26656"
```

## Setup validator node

Below are the instructions to generate & submit your genesis transaction

### Generate genesis transaction (gentx)

1. Initialize the Source directories and create the local genesis file with the correct chain-id:

```bash
sourced init <moniker-name> --chain-id=sourcechain-testnet
```

2. Create a local key pair (skip this step if you already have a key):

```sh
> sourced keys add <key-name>
```

3. Add your account to your local genesis file with a given amount and the key you just created. Use only `10000000000usource`, other amounts will be ignored.

```bash
sourced add-genesis-account $(sourced keys show <key-name> -a) 10000000000usource
```

4. Create the gentx, use only `9000000000usource`:

```bash
sourced gentx <key-name> 9000000000usource --chain-id=sourcechain-testnet
```

If all goes well, you will see a message similar to the following:

```bash
Genesis transaction written to "/home/user/.source/config/gentx/gentx-******.json"
```

5. Change minimum gas prices in `app.toml` to `0.025usource`.



### Submit genesis transaction

- Fork [the testnets repo](https://github.com/SourceNexxus/testnet-genesis) into your Github account

- Clone your repo using

  ```bash
  git clone https://github.com/<your-github-username>/testnet-genesis
  ```

- Copy the generated gentx json file to `<repo_path>/testnet-genesis/gentx/`

  ```sh
  > cd testnet-genesis
  > cp ~/.source/config/gentx/gentx*.json ./testnet-genesis/gentx/
  ```

- Commit and push to your repo
- Create a PR onto https://github.com/SourceNexxus/testnet-genesis
- Only PRs from individuals / groups with a history successfully running nodes will be accepted. This is to ensure the network successfully starts on time.
