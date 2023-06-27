# Nois



![](https://github.com/MrHoodd/MainnetNodes/assets/105497450/0eb53292-5507-4579-8f62-8d100e987f6c)

#### Official Links

[Website](https://nois.network/) | [Discord](https://discord.gg/dHdpwtEb6F) | [Twitter](https://twitter.com/NoisRNG)

#### :satellite:[EXPLORER](https://explorer.moonbridge.team/nois-mainnet)

**Chain ID:** nois-1 | **Latest Version:** v1.0.3 | **Custom Port:** 37

#### Install Node Guide Nois

:red\_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

#### Update system and install tools

```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop -y
```

#### Install GO

```bash
cd $HOME
version="1.20.4"
wget "https://golang.org/dl/go$version.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$version.linux-amd64.tar.gz"
rm "go$version.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### Download and install

```bash
cd $HOME
git clone https://github.com/noislabs/noisd.git
cd noisd
git checkout v1.0.3
make install
noisd version --long | grep -e commit -e version
```

#### Config and Init node

```bash
# Set node configuration
noisd config chain-id nois-1
noisd config keyring-backend test
noisd init $MONIKER --chain-id nois-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.noisd/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.noisd/config/addrbook.json

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.05unois\"|" $HOME/.noisd/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.noisd/config/app.toml
```

#### Disable indexer (optional)

```bash
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.noisd/config/config.toml
```

#### Enable Prometheus (optional)

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.noisd/config/config.toml
```

#### Setting custom ports (optional)

```bash
CUSTOM_PORT=37
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.noisd/config/config.toml
sed -i.bak -e "s%:1317%:${CUSTOM_PORT}317%g; s%:8080%:${CUSTOM_PORT}080%g; s%:9090%:${CUSTOM_PORT}090%g; s%:9091%:${CUSTOM_PORT}091%g; s%:8545%:${CUSTOM_PORT}545%g; s%:8546%:${CUSTOM_PORT}546%g; s%:6065%:${CUSTOM_PORT}065%g" $HOME/.noisd/config/app.toml
```

#### Create service

```bash
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=Nois Node
After=network-online.target
[Service]
User=$USER
Type=simple
ExecStart=$(which noisd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable noisd
```

#### Start service and check the logs

```bash
sudo systemctl start noisd && sudo journalctl -u noisd -f --no-hostname -o cat
```

#### Delete node

```bash
sudo systemctl stop noisd
sudo systemctl disable noisd
sudo rm -rf /etc/systemd/system/noisd*
sudo systemctl daemon-reload
sudo rm -f $(which noisd) 
sudo rm -rf $HOME/.noisd 
sudo rm -rf $HOME/noisd
```
