# Ojo

### Ojo node Installation Instructions.

[Official documentation](https://github.com/ojo-network/ojo)

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

### Download and build binaries
```
cd && rm -rf ojo
git clone https://github.com/ojo-network/ojo
cd ojo
git checkout v0.1.2
make install
```

# Config and init app
```
ojod config chain-id ojo-devnet
ojod config keyring-backend test
ojod config node tcp://localhost:21657
ojod init "your moniker" --chain-id ojo-devnet
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/ojo-testnet/genesis.json > $HOME/.ojo/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/ojo-testnet/addrbook.json > $HOME/.ojo/config/addrbook.json
```

# Set seeds and peers
```
sed -i -e 's|^seeds *=.*|seeds = "7186f24ace7f4f2606f56f750c2684d387dc39ac@ojo-testnet-seed.itrocket.net:12656"|' $HOME/.ojo/config/config.toml
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uojo"|' $HOME/.ojo/config/app.toml
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.ojo/config/app.toml
```

# Create service file
```
sudo tee /etc/systemd/system/ojod.service > /dev/null << EOF
[Unit]
Description=Ojo node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which ojod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable ojod.service
```

# Reset and download snapshot
```
curl "https://snapshots-testnet.nodejumper.io/ojo-testnet/ojo-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.ojo"
```

# enable and start service
```
sudo systemctl start ojod.service
sudo journalctl -u ojod.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
ojod keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
ojod keys add wallet --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
ojod status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### We receive tokens from the tap in the [discord](https://discord.gg/CenmR5B8bX)
```
The faucet is not working yet, so we are waiting or asking for tokens in the chat.
```

# before creating a validator, you need to fund your wallet and check balance
```
ojod q bank balances $(ojod keys show wallet -a) 
```
# Create validator
```
ojod tx staking create-validator \
--amount=1000000uojo \
--pubkey=$(ojod tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO❤️" \
--chain-id=ojo-devnet \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.01uojo \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
No update

Current network:ojo-devnet
Current version:v0.1.2
```

### Useful commands

Check balance
```
ojod q bank balances $(ojod keys show wallet -a) 
```

CHECK SERVICE LOGS
```
sudo journalctl -u ojod -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart ojod
```

GET VALIDATOR INFO
```
ojod status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
ojod tx staking delegate $(ojod keys show wallet --bech val -a) 1000000uojo --from wallet --chain-id ojo-devnet --gas-prices 0.01uojo --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop ojod && sudo systemctl disable ojod && sudo rm /etc/systemd/system/ojod.service && sudo systemctl daemon-reload && rm -rf $HOME/.ojo && rm -rf ojo && sudo rm -rf $(which ojod) 
```
