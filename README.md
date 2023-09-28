
![image](https://github.com/molla202/Structs/assets/91562185/cc163ae3-c886-468c-a64a-a48699cdae4b)


### Update ve gereklilikler
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y


### Go kurulumu
```
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```



`Site` : https://www.playstructs.com/
`Explorer` : https://testnet.itrocket.net/structs/staking
## Sistem Gereksinimleri

| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| 500 GB SSD |

### Varyasyon ayarları
```
cüzdan ve monkier adınızı giriniz.
echo "export WALLET="cüzdan-adınızı-yazınız"" >> $HOME/.bash_profile
echo "export MONIKER="moniker-adınızı-yazınız"" >> $HOME/.bash_profile
echo "export STRUCTS_CHAIN_ID="structstestnet-74"" >> $HOME/.bash_profile
echo "export STRUCTS_PORT="28"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Binary çekelim
```
cd $HOME
wget -O https://testnet-files.itrocket.net/structs/structsd
chmod +X $HOME/structsd
mv $HOME/structsd $HOME/go/bin
```
### İnit işlemleri
```
init kısmında adınızı yazınız.
structsd config node tcp://localhost:${STRUCTS_PORT}657
structsd config keyring-backend os
structsd config chain-id structstestnet-74
```
```
structsd init "adınızı-yaıznız" --chain-id structstestnet-74
```
### Adressbook ve genesis
```
wget -O $HOME/.structs/config/genesis.json https://testnet-files.itrocket.net/structs/genesis.json
wget -O $HOME/.structs/config/addrbook.json https://testnet-files.itrocket.net/structs/addrbook.json
```
# seed ve peer (bulucaz :D )
```
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.structs/config/config.toml
```
### Port ayarı
```
sed -i.bak -e "s%:1317%:${STRUCTS_PORT}317%g;
s%:8080%:${STRUCTS_PORT}080%g;
s%:9090%:${STRUCTS_PORT}090%g;
s%:9091%:${STRUCTS_PORT}091%g;
s%:8545%:${STRUCTS_PORT}545%g;
s%:8546%:${STRUCTS_PORT}546%g;
s%:6065%:${STRUCTS_PORT}065%g" $HOME/.structs/config/app.toml


sed -i.bak -e "s%:26658%:${STRUCTS_PORT}658%g;
s%:26657%:${STRUCTS_PORT}657%g;
s%:6060%:${STRUCTS_PORT}060%g;
s%:26656%:${STRUCTS_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${STRUCTS_PORT}656\"%;
s%:26660%:${STRUCTS_PORT}660%g" $HOME/.structs/config/config.toml
```
# Pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.structs/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.structs/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.structs/config/app.toml
```
# Gas prometeus ve index ayarı
```
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0structs"/g' $HOME/.structs/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.structs/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.structs/config/config.toml
```
### Servis dosyası
```
sudo tee /etc/systemd/system/structsd.service > /dev/null <<EOF
[Unit]
Description=Structs node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.structs
ExecStart=$(which structsd) start --home $HOME/.structs
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
### Snap
```
structsd tendermint unsafe-reset-all --home $HOME/.structs
if curl -s --head curl https://testnet-files.itrocket.net/structs/snap_structs.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/structs/snap_structs.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.structs
    else
  echo no have snap
fi
```
### Nodu başlatalım
```
sudo systemctl daemon-reload
sudo systemctl enable structsd
sudo systemctl restart structsd && sudo journalctl -u structsd -fo cat
```
### Cüzdan
```
structsd keys add $WALLET
```
```
structsd keys add $WALLET --recover
```
### Validator
```
structsd tx staking create-validator \
--amount 1000000structs \
--from cüzdan-yazınız \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(structsd tendermint show-validator) \
--moniker "moniker-yazınız" \
--identity "" \
--details "" \
--chain-id structstestnet-74 \
--gas auto --gas-adjustment 1.5 \
-y
```
### Silme
```
sudo systemctl stop structsd
sudo systemctl disable structsd
sudo rm -rf /etc/systemd/system/structsd.service
sudo rm $(which structsd)
sudo rm -rf $HOME/.structs
sed -i "/STRUCTS_/d" $HOME/.bash_profile
```
