# 0g labs

### 0g labs node Installation Instructions.

[Official documentation](https://0glabs.gitbook.io/0g-doc/run-a-node/validator-node)

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
rm -rf 0g-evmos
git clone https://github.com/0glabs/0g-evmos.git
cd 0g-evmos
git checkout v1.0.0-testnet
make build
mv $HOME/0g-evmos/build/evmosd $HOME/go/bin/
```

# Config and init app
```
evmosd config node tcp://localhost:${OG_PORT}657
evmosd config keyring-backend os
evmosd config chain-id zgtendermint_9000-1
evmosd init "YOUR NODE NAME" --chain-id zgtendermint_9000-1
```

# Download genesis and addrbook
```
wget -O $HOME/.evmosd/config/genesis.json https://testnet-files.itrocket.net/og/genesis.json
wget -O $HOME/.evmosd/config/addrbook.json https://testnet-files.itrocket.net/og/addrbook.json
```

# Set seeds and peers
```
SEEDS="c9b8e7e220178817c84c7268e186b231bc943671@og-testnet-seed.itrocket.net:47656"
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml
```

# Create service file
```
sudo tee /etc/systemd/system/evmosd.service > /dev/null <<EOF
[Unit]
Description=Og node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.evmosd
ExecStart=$(which evmosd) start --home $HOME/.evmosd
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

# Reset and download snapshot
```
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd
if curl -s --head curl https://testnet-files.itrocket.net/og/snap_og.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/og/snap_og.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.evmosd
    else
  echo no have snap
fi
```

# enable and start service
```
sudo systemctl daemon-reload
sudo systemctl enable evmosd
sudo systemctl restart evmosd && sudo journalctl -u evmosd -f
```

### Becoming a Validator

# Create wallet key new
```
evmosd keys add $WALLET
```

(OPTIONAL) RECOVER EXISTING KEY
```
evmosd keys add $WALLET --recover
```

# save wallet and validator address
```
WALLET_ADDRESS=$(evmosd keys show $WALLET -a)
VALOPER_ADDRESS=$(evmosd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
evmosd status 2>&1 | jq .SyncInfo
```

### We receive tokens from the tap in the [discord](https://discord.gg/0glabs)
```
The faucet is not working yet, so we are waiting or asking for tokens in the chat.
```

# before creating a validator, you need to fund your wallet and check balance
```
evmosd query bank balances $WALLET_ADDRESS
```
# Create validator
```
evmosd tx staking create-validator \
--amount 1000000aevmos \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(evmosd tendermint show-validator) \
--moniker "YOUR MONIKER" \
--identity "FFB0AA51A2DF5955" \
--details "I love YTWO❤️" \
--chain-id zgtendermint_9000-1 \
--gas 500000 --gas-prices 99999aevmos \
-y
```

### Update
```
No update

Current network:zgtendermint_9000-1
Current version:v1.0.0-testnet
```

### Useful commands

Check balance
```
evmosd q bank balances $(evmosd keys show wallet -a) 
```

CHECK SERVICE LOGS
```
sudo journalctl -u evmosd -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart evmosd
```

GET VALIDATOR INFO
```
evmosd status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
evmosd tx staking delegate $(evmosd keys show wallet --bech val -a) 1000000aevmos --from wallet --chain-id zgtendermint_9000-1 --gas-prices 250000000aevmos --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop evmosd
sudo systemctl disable evmosd
sudo rm -rf /etc/systemd/system/evmosd.service
sudo rm $(which evmosd)
sudo rm -rf $HOME/.evmosd
sed -i "/OG_/d" $HOME/.bash_profile
```
