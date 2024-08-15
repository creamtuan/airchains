# Download binary
cd $HOME && mkdir -p go/bin/
wget https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond
chmod +x junctiond
mv junctiond $HOME/go/bin/


# Set node CLI configuration
junctiond config set client chain-id junction
junctiond config set client keyring-backend test
junctiond config set client node tcp://localhost:26657

# Initialize the node
junctiond init "Your Node Name" --chain-id junction

# Download genesis and addrbook files
curl -L https://snapshots-testnet.nodejumper.io/airchains-testnet/genesis.json > $HOME/.junction/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/airchains-testnet/addrbook.json > $HOME/.junction/config/addrbook.json

# Set seeds
sed -i -e 's|^seeds *=.*|seeds = "575e98598e9813a26576759c7ef70fd38d2516a4@junction-testnet-rpc.synergynodes.com:15656,04e2fdd6ec8f23729f24245171eaceae5219aa91@airchains-testnet-seed.itrocket.net:19656,aeaf101d54d47f6c99b4755983b64e8504f6132d@airchain-testnet-peer.dashnode.org:28656,bb26fc8cef05cee75d4cae3f25e17d74c7913967@airchains-t.seed.stavr.tech:4476,df949a46ae6529ae1e09b034b49716468d5cc7e9@testnet-seeds.stakerhouse.com:13756,48887cbb310bb854d7f9da8d5687cbfca02b9968@35.200.245.190:26656,60133849b4c83531eb2d835970035a0f08868658@65.109.93.124:28156,df2a56a208821492bd3d04dd2e91672657c79325@airchain-testnet-peer.cryptonode.id:27656,04e2fdd6ec8f23729f24245171eaceae5219aa91@airchains-testnet-seed.itrocket.net:19656,3dc2f101876e1a26730f99c06a5a2eb6e2cc2349@65.21.69.53:33656"|' $HOME/.junction/config/config.toml

# Set minimum gas price
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.00025amf"|' $HOME/.junction/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.junction/config/app.toml

# Download latest chain data snapshot
curl "https://snapshots-testnet.nodejumper.io/airchains-testnet/airchains-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.junction"

# Create a service
sudo tee /etc/systemd/system/junctiond.service > /dev/null << EOF
[Unit]
Description=Airchains node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which junctiond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable junctiond.service

# Start the service and check the logs
sudo systemctl start junctiond.service
sudo journalctl -u junctiond.service -f --no-hostname -o cat
