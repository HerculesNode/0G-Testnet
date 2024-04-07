![image](https://github.com/HerculesNode/0G-Testnet/assets/101635385/4b44e27c-d4aa-479f-8f01-42f92edfb36c)



### Linkler
 * [Hercules Telegram](https://t.me/HerculesNode)
 * [Hercules Twitter](https://twitter.com/Herculesnode)
 * [OG Discord](https://discord.gg/0glabs)
 * [Explorer](https://explorer.validatorvn.com/OG-Testnet)


## 🟢 Sistem özellikleri
- Memory: 8 GB RAM
- CPU: 4 cores
- Disk: 500 GB NVME SSD
- Bandwidth: 100mbps Gbps for Download / Upload
- Linux amd64 arm64 (The guide was tested on Ubuntu 20.04 LTS)


## 🟢 Sistemi güncelleyelim
```shell
sudo apt update && \
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

## 🟢 Go kuralım
```shell
cd $HOME && \
ver="1.21.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## 🟢 Evmosd Yapı oluşturalım
```shell
git clone https://github.com/0glabs/0g-evmos.git
cd 0g-evmos
git checkout v1.0.0-testnet
make install
evmosd version
```

## 🟢 Ayarları yapalım
```shell
echo 'export MONIKER="My_Node"' >> ~/.bash_profile
echo 'export CHAIN_ID="zgtendermint_9000-1"' >> ~/.bash_profile
echo 'export WALLET_NAME="wallet"' >> ~/.bash_profile
echo 'export RPC_PORT="26657"' >> ~/.bash_profile
source $HOME/.bash_profile
```

## 🟢 Değişkenleri Ayarlayalım
```shell
cd $HOME
evmosd init $MONIKER --chain-id $CHAIN_ID
evmosd config chain-id $CHAIN_ID
evmosd config node tcp://localhost:$RPC_PORT
evmosd config keyring-backend os 
```

## 🟢 Genesis dosyası oluşturalım
```shell
wget https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json -O $HOME/.evmosd/config/genesis.json
```
## 🟢 Seed ekleyelim
```shell
PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326" && \
SEEDS="8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27@54.176.175.48:26656,8e20e8e88d504e67c7a3a58c2ea31d965aa2a890@54.193.250.204:26656,e50ac888b35175bfd4f999697bdeb5b7b52bfc06@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml
```

## 🟢 Gas price ayarlayalım
```shell
sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00252aevmos\"/" $HOME/.evmosd/config/app.toml
```

## 🟢 Servis dosyası oluşturalım
```bash
sudo tee /etc/systemd/system/ogd.service > /dev/null <<EOF
[Unit]
Description=OG Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which evmosd) start --home $HOME/.evmosd
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## 🟢 Nodeyi çalıştırın
```shell
sudo systemctl daemon-reload && \
sudo systemctl enable ogd && \
sudo systemctl restart ogd && \
sudo journalctl -u ogd -f -o cat
```


## 🟢 Snap indirelim

```shell
sudo systemctl stop ogd
```

```shell
wget https://rpc-zero-gravity-testnet.trusted-point.com/latest_snapshot.tar.lz4
```

```shell
cp $HOME/.evmosd/data/priv_validator_state.json $HOME/.evmosd/priv_validator_state.json.backup
```

```shell
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book
```

```shell
lz4 -d -c ./latest_snapshot.tar.lz4 | tar -xf - -C $HOME/.evmosd
```

```shell
mv $HOME/.evmosd/priv_validator_state.json.backup $HOME/.evmosd/data/priv_validator_state.json
```

```shell
sudo systemctl restart ogd && sudo journalctl -u ogd -f -o cat
```

```shell
evmosd status | jq .SyncInfo
```
![image](https://github.com/HerculesNode/0G-Testnet/assets/101635385/bf3d7b46-b90d-4ce5-8b5e-06074cddb3c2)

False çıktısı alın. Kuruluma devam edelim



## 🟢 Validatör için cüzdan oluşturalım. Şifre isteyecek bir şifre belirleyin sonra size cüzdan adresi ve kelimeleri verecek onları kaydedin

```shell
evmosd keys add $WALLET_NAME
```

![image](https://github.com/HerculesNode/0G-Testnet/assets/101635385/4577b34a-42c8-4426-be8f-dc73feb60535)


Şimdi cüzdanımızı dönüştürelim faucet için bu adresi kullanacağız
```shell
echo "0x$(evmosd debug addr $(evmosd keys show $WALLET_NAME -a) | grep hex | awk '{print $3}')"
```


Faucet token alın https://faucet.0g.ai/


Faucet sonrası cüzdan adresinize token gelmişmi kontrol edin resimdeki gibi olacak
```shell
echo "0x$(evmosd debug addr $(evmosd keys show $WALLET_NAME -a) | grep hex | awk '{print $3}')"
```

![image](https://github.com/HerculesNode/0G-Testnet/assets/101635385/bfbe1e02-0c02-4a39-a71c-a9d8109fcb46)




## 🟢 Token geldikten sonra validatör oluşturalım
```shell
evmosd tx staking create-validator \
  --amount=10000000000000000aevmos \
  --pubkey=$(evmosd tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=$CHAIN_ID \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --from=$WALLET_NAME \
  --identity="" \
  --website="" \
  --details="0G to the moon!" \
  --gas=500000 --gas-prices=99999aevmos \
  -y
```


Explorer kontrol edin : https://explorer.validatorvn.com/OG-Testnet


