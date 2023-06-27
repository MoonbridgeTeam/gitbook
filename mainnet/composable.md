# Composable

![](https://github.com/MrHoodd/TestnetNodes/assets/105497450/ab0fb624-942f-4c0f-b436-832376007d49)

#### Official Links

[Website](https://www.composable.finance/) | [Discord](https://discord.gg/composable) | [Twitter](https://twitter.com/ComposableFin)

#### :satellite:[EXPLORER](https://explorer.moonbridge.team/composable-mainnet)

**Chain ID:** centauri-1 | **Latest Version:** v2.3.5 | **Custom Port:** 35

#### Install Node Guide Composable centauri-1

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
git clone https://github.com/notional-labs/composable-centauri.git
cd composable-centauri
git checkout v2.3.5
make install
banksyd version --long | grep -e commit -e version
```

#### Config and Init node

```bash
# Set node configuration
banksyd config chain-id centauri-1
banksyd config keyring-backend test
banksyd init $MONIKER --chain-id centauri-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Composable/genesis.json > $HOME/.banksy/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Composable/addrbook.json > $HOME/.banksy/config/addrbook.json

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0upica\"|" $HOME/.banksy/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.banksy/config/app.toml
```

#### Disable indexer (optional)

```bash
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.banksy/config/config.toml
```

#### Enable Prometheus (optional)

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.banksy/config/config.toml
```

#### Setting custom ports (optional)

```bash
BANKSY_PORT=35
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BANKSY_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${BANKSY_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BANKSY_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BANKSY_PORT}656\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${BANKSY_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BANKSY_PORT}660\"%" $HOME/.banksy/config/config.toml
sed -i.bak -e "s%:1317%:${BANKSY_PORT}317%g; s%:8080%:${BANKSY_PORT}080%g; s%:9090%:${BANKSY_PORT}090%g; s%:9091%:${BANKSY_PORT}091%g; s%:8545%:${BANKSY_PORT}545%g; s%:8546%:${BANKSY_PORT}546%g; s%:6065%:${BANKSY_PORT}065%g" $HOME/.banksy/config/app.toml
```

#### Create service

```bash
sudo tee /etc/systemd/system/banksyd.service > /dev/null <<EOF
[Unit]
Description=Composable Node
After=network-online.target
[Service]
User=$USER
Type=simple
ExecStart=$(which banksyd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable banksyd
```

#### Start service and check the logs

```bash
sudo systemctl start banksyd && sudo journalctl -u banksyd -f --no-hostname -o cat
```

#### Delete node

```bash
sudo systemctl stop banksyd
sudo systemctl disable banksyd
sudo rm -rf /etc/systemd/system/banksyd*
sudo systemctl daemon-reload
sudo rm -f $(which banksyd) 
sudo rm -rf $HOME/.banksy 
sudo rm -rf $HOME/composable-centauri
```
