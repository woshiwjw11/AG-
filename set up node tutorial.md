## set up node tutorial 

`## Server environment Ubuntu20.04  step by step `

```sh
##Install Node.js
curl https://deb.nodesource.com/setup_12.x | sudo bash
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -

echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt update

sudo apt upgrade -y

sudo apt install nodejs=12.* yarn build-essential jq -y
##Install Go
sudo rm -rf /usr/local/go

The curl https://dl.google.com/go/go1.15.7.linux-amd64.tar.gz | sudo tar - C/usr/local - ZXVF -

cat <<'EOF' >>$HOME/.profile

export GOROOT=/usr/local/go

export GOPATH=$HOME/go

export GO111MODULE=on

export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin

EOF

source $HOME/.profile
##after that you should take a look 
go version
## if shows 
##go version go1.15.7 linux/amd64 is all right
##Install Agoric SDK
Git clone https://github.com/Agoric/agoric-sdk - b @ agoric/sdk@2.15.1

cd agoric-sdk

yarn install

yarn build

cd packages/cosmic-swingset && make
##check 
ag-chain-cosmos version --long
##Configuring Your Node
curl https://testnet.agoric.net/network-config > chain.json

chainName=jq -r .chainName < chain.json

echo $chainName
## initialized your validator 
ag-chain-cosmos init --chain-id $chainName <your_moniker>
curl https://devnet.agoric.net/genesis.json > $HOME/.ag-chain-cosmos/config/genesis.json 
ag-chain-cosmos unsafe-reset-all
##Adjust configuration
peers=$(jq '.peers | join(",")' < chain.json)
seeds=$(jq '.seeds | join(",")' < chain.json)
echo $peers
echo $seeds
sed -i.bak 's/^log_level/# log_level/' $HOME/.ag-chain-cosmos/config/config.toml
sed -i.bak -e "s/^seeds *=.*/seeds = $seeds/; s/^persistent_peers *=.*/persistent_peers = $peers/" $HOME/.ag-chain-cosmos/config/config.toml
## start your node 
sudo tee <<EOF >/dev/null /etc/systemd/system/ag-chain-cosmos.service
[Unit]
Description=Agoric Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/ag-chain-cosmos start --log_level=warn
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
##if you wanna run by cli you can just run this command 
ag-chain-cosmos start --log_level=warn
## if you wanna use sys  try this flowing  
sudo systemctl enable ag-chain-cosmos
sudo systemctl daemon-reload
sudo systemctl start ag-chain-cosmos
##To check on the status of syncing
ag-cosmos-helper status 2>&1 | jq .SyncInfo


```



```shell
##Connect your validator to prometheus analytics  n
nano ~/.ag-chain-cosmos/config/app.toml
## change your app.toml just like this 
[telemetry]
enabled = true
prometheus-retention-time = 60

[api]
# Note: this key is "enable" (without a "d", not "enabled")
enable = true
address = "tcp://your host ip address:1317"
## save your change by ctrl+x 
nano ~/.ag-chain-cosmos/config/config.toml
## change your config.toml just like this


[instrumentation]
prometheus = true
prometheus_listen_addr = ":26660"
## save your change by ctrl+x 
##Agoric VM (SwingSet) metrics
##first set your environment variable
export OTEL_EXPORTER_PROMETHEUS_PORT=9464
export OTEL_EXPORTER_PROMETHEUS_HOST=your host ip address
## to see you data from browser
## stop you agoric node 
sudo systemctl stop ag-chain-cosmos
## open a new screen 
screen -S ag
## start 
OTEL_EXPORTER_PROMETHEUS_PORT=9464 ag-chain-cosmos start --log_level=warn
## see your data 
yourhostip:9464/metrics 
yourhostip:26660/metrics

## that's all

```

