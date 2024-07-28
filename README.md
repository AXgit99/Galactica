**install go, if needed**
```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

**set vars**
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export GALACTICA_CHAIN_ID="galactica_9302-1"" >> $HOME/.bash_profile
echo "export GALACTICA_PORT="46"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**download binary**
```
cd $HOME
rm -rf galactica
git clone https://github.com/Galactica-corp/galactica
cd galactica
git checkout v0.1.2
make build
mv $HOME/galactica/build/galacticad $HOME/go/bin
```

**config and init app**
```
galacticad config node tcp://localhost:${GALACTICA_PORT}657
galacticad config keyring-backend os
galacticad config chain-id galactica_9302-1
galacticad init "test" --chain-id galactica_9302-1
```

**download genesis and addrbook**
```
wget -O $HOME/.galactica/config/genesis.json https://server-4.itrocket.net/testnet/galactica/genesis.json
wget -O $HOME/.galactica/config/addrbook.json  https://server-4.itrocket.net/testnet/galactica/addrbook.json
```

**set seeds and peers**
```
SEEDS="52ccf467673f93561c9d5dd4434def32ef2cd7f3@galactica-testnet-seed.itrocket.net:46656"
PEERS="c9993c738bec6a10cfb8bb30ac4e9ae6f8286a5b@galactica-testnet-peer.itrocket.net:11656,12c0c18b03527ae9babbb76538551f3191a89676@144.91.95.178:26656,6f5ea6dbdd258ab7ae6b30c76b5053993beb068f@65.109.52.156:46656,7b0c8f2dfca134ae2d4939cdb6f57353be77113c@135.181.216.54:3460,03d3c773e52c46274e35f3a36a7644e7f2ff37be@167.235.14.83:35656,f3cd6b6ebf8376e17e630266348672517aca006a@46.4.5.45:27456,7185be6d31b30299cac8f7b2ddab59c24e5dfb35@62.169.19.141:26656,232050092a90b94b002f34a79de8a3859ed13ee7@5.104.108.184:26656,b352df9a2ed42fd741120598fbecb188dc01a48e@84.247.191.148:26656,6b846b316d704d78f3f9ca46d86cebc5a22de8ae@161.97.111.249:26656,378dcf86fb8748ce9b1ffff9d8310c6c59af09f3@5.189.175.174:26656,23b708f655464ce585814d25c6b546175e7c55ef@91.227.33.18:46656,e104cf08023696afa2c364ca386b67c5db0982b5@151.115.60.105:46656,64309c18c7bb536f79d566766873ec3dc3dfdd1b@198.7.121.177:22656,1dfafc2ebc9dc5cc06cb6040132b19c70fa48a9a@144.126.139.139:46655,76e3c79cb8ba7b42238dba9867036497757e7502@5.78.90.75:46656,998472dbf0a6c8a162f9ddd201565ad6f9eabf2c@95.217.89.100:46656,9990ab130eac92a2ed1c3d668e9a1c6e811e8f35@148.251.177.108:27456,d6d9badeefbc0bc80d263f3bfc196ab91a1049dc@[2402:1f00:8300:92c::]:13156,a028446e34e3c5bd198a60bf6e799a05e8db16a1@116.202.162.188:15656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.galactica/config/config.toml
```

# set custom ports in app.toml
sed -i.bak -e "s%:1317%:${GALACTICA_PORT}317%g;
s%:8080%:${GALACTICA_PORT}080%g;
s%:9090%:${GALACTICA_PORT}090%g;
s%:9091%:${GALACTICA_PORT}091%g;
s%:8545%:${GALACTICA_PORT}545%g;
s%:8546%:${GALACTICA_PORT}546%g;
s%:6065%:${GALACTICA_PORT}065%g" $HOME/.galactica/config/app.toml

# set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${GALACTICA_PORT}658%g;
s%:26657%:${GALACTICA_PORT}657%g;
s%:6060%:${GALACTICA_PORT}060%g;
s%:26656%:${GALACTICA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${GALACTICA_PORT}656\"%;
s%:26660%:${GALACTICA_PORT}660%g" $HOME/.galactica/config/config.toml

# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.galactica/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.galactica/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.galactica/config/app.toml

# set minimum gas price, enable prometheus and disable indexing
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10agnet"|g' $HOME/.galactica/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.galactica/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.galactica/config/config.toml

# create service file
sudo tee /etc/systemd/system/galacticad.service > /dev/null <<EOF
[Unit]
Description=Galactica node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.galactica
ExecStart=$(which galacticad) start --home $HOME/.galactica --chain-id galactica_9302-1
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# reset and download snapshot
galacticad tendermint unsafe-reset-all --home $HOME/.galactica
if curl -s --head curl https://server-4.itrocket.net/testnet/galactica/galactica_2024-07-26_1609629_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-4.itrocket.net/testnet/galactica/galactica_2024-07-26_1609629_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.galactica
    else
  echo "no snapshot founded"
fi

# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable galacticad
sudo systemctl restart galacticad && sudo journalctl -u galacticad -f
