# 0g labs

### 0g labs node Installation Instructions.

[Official documentation](https://docs.0g.ai/0g-doc/run-a-node/validator-node)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 20.04 or 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

# Set vars
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export OG_CHAIN_ID="zgtendermint_9000-1"" >> $HOME/.bash_profile
echo "export OG_PORT="47"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Download and build binaries
```
cd $HOME
rm -rf 0g-chain
git clone -b v0.3.0 https://github.com/0glabs/0g-chain.git
cd 0g-chain
make install
```

# Config and init app
```
0gchaind config node tcp://localhost:${OG_PORT}657
0gchaind config keyring-backend os
0gchaind config chain-id zgtendermint_16600-2
0gchaind init "your name" --chain-id zgtendermint_16600-2
```

# Download genesis and addrbook
```
wget -O $HOME/.0gchain/config/genesis.json https://server-5.itrocket.net/testnet/og/genesis.json
wget -O $HOME/.0gchain/config/addrbook.json  https://server-5.itrocket.net/testnet/og/addrbook.json
```

# Set seeds and peers
```
SEEDS="8f21742ea5487da6e0697ba7d7b36961d3599567@og-testnet-seed.itrocket.net:47656"
PEERS="80fa309afab4a35323018ac70a40a446d3ae9caf@og-testnet-peer.itrocket.net:11656,928f42a91548484f35a5c98aa9dcb25fb1790a70@65.21.46.201:26656,0e44ddb20380bc06f489b8a58f6cb634c208d4e8@136.243.43.106:26656,5bb50b80f855b343ac9d8957055fd44b8c51667e@135.181.229.232:31656,593f012c2f496c3e3972e4c302e6c5d3bfef1a2a@38.242.197.125:26656,7207c781cc31324f179bf2dbbd670fa79e119d3b@37.27.131.251:18456,c85eaa1b3cbe4d7fb19138e5a5dc4111491e6e03@115.78.229.59:10156,ffdf7a8cc6dbbd22e25b1590f61da149349bdc2e@135.181.229.206:26656,7e7ca71ae2a6b97ba31af673355f0b3ba9cbbc23@213.199.43.252:26656,5e098c96e69e3e2a943b923eda791ba34543f792@116.202.210.96:26656,8bd2797c8ece0f099a1c31f98e5648d192d8cd54@38.242.146.162:26656,0ae19691f97f5797694c253bc06c79c8b58ea2a8@85.190.242.81:26656,6ed155a028ca398966a80e4daaaf86a7e0104ada@164.68.118.7:12656,9dbb76298d1625ebcc47d08fa7e7911967b63b61@45.159.221.57:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.0gchain/config/config.toml
```

# Create service file
```
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0G node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.0gchain
ExecStart=$(which 0gchaind) start --home $HOME/.0gchain --log_output_console
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

# Reset and download snapshot
```
0gchaind tendermint unsafe-reset-all --home $HOME/.0gchain
if curl -s --head curl https://server-5.itrocket.net/testnet/og/og_2024-08-09_586084_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-5.itrocket.net/testnet/og/og_2024-08-09_586084_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.0gchain
    else
  echo "no snapshot founded"
fi
```

# enable and start service
```
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind
sudo systemctl restart 0gchaind && sudo journalctl -u 0gchaind -f
```

### Becoming a Validator

# Create wallet key new
```
0gchaind keys add $WALLET
```

(OPTIONAL) RECOVER EXISTING KEY
```
0gchaind keys add $WALLET --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
0gchaind status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### We receive tokens from the tap in the [site](https://faucet.0g.ai/)

# before creating a validator, you need to fund your wallet and check balance
```
0gchaind q bank balances $(0gchaind keys show wallet -a) 
```

# Create validator
```
0gchaind tx staking create-validator \
--amount=1000000ua0gi \
--pubkey=$(0gchaind tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO❤️" \
--chain-id=zgtendermint_16600-2 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.0025ua0gi \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
No update

Current network:zgtendermint_16600-2
Current version:v0.3.0
```

### Useful commands

Check balance
```
0gchaind q bank balances $(0gchaind keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u 0gchaind -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart 0gchaind
```

GET VALIDATOR INFO
```
0gchaind status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
0gchaind tx staking delegate $(0gchaind keys show wallet --bech val -a) 1000000ua0gi --from wallet --chain-id zgtendermint_16600-2 --gas-prices 0.0025ua0gi --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop 0gchaind && sudo systemctl disable 0gchaind && sudo rm /etc/systemd/system/0gchaind.service && sudo systemctl daemon-reload && rm -rf $HOME/.0gchain && rm -rf 0g-chain && sudo rm -rf $(which 0gchaind) 
```
