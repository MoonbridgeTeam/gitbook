# Gitopia

![](https://github.com/MrHoodd/MainnetNodes/assets/105497450/756476dd-24ed-4797-a0e0-69ea355cc022)

#### Official Links

[Website](https://gitopia.com/) | [Discord](https://discord.com/invite/aqsKW3hUHD) | [Twitter](https://twitter.com/gitopiaDAO)

#### :satellite:[EXPLORER](https://explorer.moonbridge.team/gitopia-mainnet)

**Chain ID:** gitopia | **Latest Version:** v2.1.1 | **Custom Port:** 36

#### Install Node Guide Gitopia

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
git clone https://github.com/gitopia/gitopia.git
cd gitopia
git checkout v2.1.1
make install
gitopiad version --long | grep -e commit -e version
```

#### Config and Init node

```bash
# Set node configuration
gitopiad config chain-id gitopia
gitopiad config keyring-backend test
gitopiad init $MONIKER --chain-id gitopia

# Download genesis and addrbook
curl -Ls https://github.com/gitopia/mainnet/raw/master/genesis.tar.gz > $HOME/genesis.tar.gz
tar -xzf $HOME/genesis.tar.gz
mv genesis.json $HOME/.gitopia/config/genesis.json
rm $HOME/genesis.tar.gz
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Gitopia/addrbook.json > $HOME/.gitopia/config/addrbook.json

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001ulore\"|" $HOME/.gitopia/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.gitopia/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.gitopia/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.gitopia/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.gitopia/config/app.toml
```

#### Disable indexer (optional)

```bash
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.gitopia/config/config.toml
```

#### Enable Prometheus (optional)

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gitopia/config/config.toml
```

#### Setting custom ports (optional)

```bash
CUSTOM_PORT=36
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.gitopia/config/config.toml
sed -i.bak -e "s%:1317%:${CUSTOM_PORT}317%g; s%:8080%:${CUSTOM_PORT}080%g; s%:9090%:${CUSTOM_PORT}090%g; s%:9091%:${CUSTOM_PORT}091%g; s%:8545%:${CUSTOM_PORT}545%g; s%:8546%:${CUSTOM_PORT}546%g; s%:6065%:${CUSTOM_PORT}065%g" $HOME/.gitopia/config/app.toml
```

#### Create service

```bash
sudo tee /etc/systemd/system/gitopiad.service > /dev/null <<EOF
[Unit]
Description=Gitopia Node
After=network-online.target
[Service]
User=$USER
Type=simple
ExecStart=$(which gitopiad) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable gitopiad
```

#### Start service and check the logs

```bash
sudo systemctl start gitopiad && sudo journalctl -u gitopiad -f --no-hostname -o cat
```

#### Delete node

```bash
sudo systemctl stop gitopiad
sudo systemctl disable gitopiad
sudo rm -rf /etc/systemd/system/gitopiad*
sudo systemctl daemon-reload
sudo rm -f $(which gitopiad) 
sudo rm -rf $HOME/.gitopia 
sudo rm -rf $HOME/gitopia
```
