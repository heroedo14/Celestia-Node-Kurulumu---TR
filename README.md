# Celestia-Node-Kurulumu---TR

Explorer: https://celestia.explorers.guru

## Sistem Gereksinimleri
- Memory: 8 GB RAM
- CPU: Quad-Core
- Disk: 250 GB SSD Depolama

## MANUEL KURULUM
Node kurulumu için aşağıdaki yönergeleri adım adım takip edin.

## Değişkenleri Ayarlama
Buraya, gezginde görünecek olan takma adınızı (doğrulayıcı) girmelisiniz.
```
NODENAME=<MONIKER_ADINIZ>
```
Değişkenleri sisteme kaydedin ve içe aktarın.
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CHAIN_ID=mamaki" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Paketleri Güncelleyin
```
sudo apt update && sudo apt upgrade -y
```

## Bağımlılıkları Yükleyin
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
```

## Go Yükleyin
```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## İkili Dosyaları İndirin ve Oluşturun
```
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app
APP_VERSION=$(curl -s https://api.github.com/repos/celestiaorg/celestia-app/releases/latest | jq -r ".tag_name")
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
```

## Ağ Araçlarını Yükleyin
```
cd $HOME
rm -rf networks
git clone https://github.com/celestiaorg/networks.git
```

## Uygulamayı Yapılandırın
```
celestia-appd config chain-id $CHAIN_ID
celestia-appd config keyring-backend test
```

## Uygulamayı Başlatın
```
celestia-appd init $NODENAME --chain-id $CHAIN_ID
```

## Genesisi Güncelleyin
```
cp $HOME/networks/$CHAIN_ID/genesis.json $HOME/.celestia-app/config
```

## Minimum Gas Fiyatını Ayarlayın
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utia\"/" $HOME/.celestia-app/config/app.toml
```

## Özel Ayarları Kullanın
```
use_legacy="false"
pex="true"
max_connections="90"
peer_gossip_sleep_duration="2ms"
sed -i.bak -e "s/^use-legacy *=.*/use-legacy = \"$use_legacy\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^pex *=.*/pex = \"$pex\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^max-connections *=.*/max-connections = \"$max_connections\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^peer-gossip-sleep-duration *=.*/peer-gossip-sleep-duration = \"$peer_gossip_sleep_duration\"/" $HOME/.celestia-app/config/config.toml
```

## Seed Peer ve Önyükleme Düğümünü Ayarlayın
```
BOOTSTRAP_PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/bootstrap-peers.txt | tr -d '\n')
MY_PEER=$(celestia-appd tendermint show-node-id)@$(curl -s ifconfig.me)$(grep -A 9 "\[p2p\]" ~/.celestia-app/config/config.toml | egrep -o ":[0-9]+")
PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/peers.txt | tr -d '\n' | head -c -1 | sed s/"$MY_PEER"// | sed "s/,,/,/g")
sed -i.bak -e "s/^bootstrap-peers *=.*/bootstrap-peers = \"$BOOTSTRAP_PEERS\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^persistent-peers *=.*/persistent-peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml
```

## Prometheus Etkinleştirin
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.celestia-app/config/config.toml
```

## Pruning Yapılandırın
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="5000"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml
```

## Zaman Aşımını Ayarla
```
sed -i.bak -e "s/^timeout-commit *=.*/timeout-commit = \"25s\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^timeout-propose *=.*/timeout-propose = \"3s\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^skip-timeout-commit *=.*/skip-timeout-commit = false/" $HOME/.celestia-app/config/config.toml
```

## Peer Bağlantısını Arttırın
```
sed -i.bak -e "s/^max-connections *=.*/max-connections = 150/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^max-num-inbound-peers *=.*/max-num-inbound-peers = 100/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^max-num-outbound-peers *=.*/max-num-outbound-peers = 50/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^max-incoming-connection-attempts *=.*/max-incoming-connection-attempts = 20/" $HOME/.celestia-app/config/config.toml
```

## Doğrulayıcı Modunu Ayarlayın
```
sed -i.bak -e "s/^mode *=.*/mode = \"validator\"/" $HOME/.celestia-app/config/config.toml
```

## Servis Oluşturun
```
tee /etc/systemd/system/celestia-appd.service > /dev/null <<EOF
[Unit]
Description=celestia-appd
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which celestia-appd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## Kaydolun ve Hizmeti Başlatın
```sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl restart celestia-appd
```

