# Relay interchain-queries using the new GO v2 Relayer

## 1. Download and build binaries
```
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
sudo mv interchain-queries /usr/local/bin/icq
```

## 2. Make home dir for icq and create configurations file
```
cd $HOME && mkdir .icq
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride-testnet
chains:
  gaia-testnet:
    key: wallet
    chain-id: GAIA
    rpc-addr: http://127.0.0.1:23657      # use your own GAIA RPC endpoint here
    grpc-addr: http://127.0.0.1:23090     # use your own GAIA GRPC endpoint here
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
    chain-id: STRIDE-TESTNET-4
    rpc-addr: http://127.0.0.1:16657      # use your own Stride GRPC endpoint here
    grpc-addr: http://127.0.0.1:16090     # use your own Stride GRPC endpoint here
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
> NOTE: Please use the same wallet you have used for relayer task, as it is the only way to prove that icq runs on your behalf!
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

You will have to wait some time until you see some logs (5-15min)

## If you get "icq not found" error try this
### These commands permanently delete all files of existing icq installation

```
cd $HOME
rm -rf interchain-queries
rm -rf /usr/local/bin/icq
rm -rf .icq
rm -rf /etc/systemd/system/icqd.service
```

## Install go

```
wget -c https://go.dev/dl/go1.18.3.linux-amd64.tar.gz && rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz && rm -rf go1.18.3.linux-amd64.tar.gz
```
```
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
```

## Install icq again



***************************

## Error parsing chain config: default chain (stride-testnet) configuration not found error

Make sure your .icq/config.yaml file is valid.
The spaces at the beginning of the lines are important. Set it like this.

![image](https://user-images.githubusercontent.com/101174090/185809736-b8b88ea0-7347-457e-96dd-59358da687d1.png)

