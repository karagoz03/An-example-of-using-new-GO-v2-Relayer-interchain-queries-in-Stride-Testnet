## 1. Download and build binaries
```
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
mv interchain-queries /usr/local/bin/icq
```
## To learn rpc and grpc, run:

`echo "$(curl -s ifconfig.me)$(grep -A 3 "[rpc]" ~/.stride/config/config.toml | egrep -o ":[0-9]+")"` in stride,

`echo "$(curl -s ifconfig.me)$(grep -A 6 "\[grpc\]" ~/.stride/config/app.toml | egrep -o ":[0-9]+")"`

`echo "$(curl -s ifconfig.me)$(grep -A 3 "[rpc]" ~/.gaia/config/config.toml | egrep -o ":[0-9]+")"` in gaia

`echo "$(curl -s ifconfig.me)$(grep -A 6 "\[grpc\]" ~/.stride/config/app.toml | egrep -o ":[0-9]+")"`


## 2. Make home dir for icq and create configurations file
```
cd $HOME && mkdir .icq
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride-testnet
chains:
  gaia-testnet:
    key: wallet
    chain-id: GAIA
    rpc-addr: http://217.160.207.56:26657
    grpc-addr: http://217.160.207.56:9090
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride-testnet:
    key: wallet
    chain-id: STRIDE-TESTNET-2
    rpc-addr: http://149.102.147.205:16657
    grpc-addr: http://149.102.147.205:9090
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```

## 3. Import wallets
```
icq keys restore --chain stride-testnet wallet
icq keys restore --chain gaia-testnet wallet
```

## 4. Create icq service
```
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=Interchain Query Service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## 5. Start icq service
```
sudo systemctl daemon-reload
sudo systemctl enable icqd
sudo systemctl restart icqd
```

## 6. Check icq logs
```
journalctl -u icqd -f -o cat
```

You will have to wait some time until you see some logs (5-15min):
![image](https://user-images.githubusercontent.com/38834586/183267537-f2b8af75-3204-49e7-9c2f-c788976d19ad.png)



After that you can check you transaction in the explorer
![image](https://user-images.githubusercontent.com/38834586/183267561-7bf34c69-7eb8-4f39-b737-c2a4a8794bcc.png)
https://poolparty.stride.zone/STRIDE/tx/405CEA3A01798D456E1A7D8B2D431E1325366846E6D7004A2B58F0AA70497550

## Special thanks to kjnodes#8455 for his awesome guides and helpfulness
