Dependencies Installation

**Install dependencies for building from source**
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential
```

**Install Go**
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
```

**Node Installation**

Clone project repository
```
cd && rm -rf archway
git clone https://github.com/archway-network/archway
cd archway
git checkout v7.0.1
``` 
**Build binary**
```
make install
```

**Set node CLI configuration**
```
archwayd config chain-id archway-1
archwayd config keyring-backend file
archwayd config node tcp://localhost:11557
```

**Initialize the node**
```
archwayd init "Your Node Name" --chain-id archway-1
```

**Download genesis and addrbook files**
```
curl -L https://snapshots.nodejumper.io/archway/genesis.json > $HOME/.archway/config/genesis.json
curl -L https://snapshots.nodejumper.io/archway/addrbook.json > $HOME/.archway/config/addrbook.json
```

**Set seeds**
```
sed -i -e 's|^seeds *=.*|seeds = "3ba7bf08f00e228026177e9cdc027f6ef6eb2b39@35.232.234.58:26656,b308dda41e4db2ee00852d91846f981c49943d46@161.97.96.91:46656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@seeds.whispernode.com:11556,ebc272824924ea1a27ea3183dd0b9ba713494f83@archway-mainnet-seed.autostake.com:26946,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:11556,b6c1198fa025ce24d26d90527c5d2b71f9399756@seed-node.mms.team:34656,6471ac9ff8474373e8055d45b6246fd8c5204890@archway.seed.mzonder.com:10756,261acb73f483d1cace653cb54f7b8815f63b7e56@archway.lgns.net:26656,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@archway.rpc.kjnodes.com:15659,bd9332cd0a99f5830ea457a32a56b32790f68716@135.181.58.28:27456"|' $HOME/.archway/config/config.toml
```

**Set minimum gas price**
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "196000000000aarch"|' $HOME/.archway/config/app.toml
```

**Set pruning**
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.archway/config/app.toml
```

**Change ports**
```
sed -i -e "s%:1317%:11517%; s%:8080%:11580%; s%:9090%:11590%; s%:9091%:11591%; s%:8545%:11545%; s%:8546%:11546%; s%:6065%:11565%" $HOME/.archway/config/app.toml
sed -i -e "s%:26658%:11558%; s%:26657%:11557%; s%:6060%:11560%; s%:26656%:11556%; s%:26660%:11561%" $HOME/.archway/config/config.toml
```

**Download latest chain data snapshot**
```
curl "https://snapshots.nodejumper.io/archway/archway_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.archway"
```

**Create a service**
```
sudo tee /etc/systemd/system/archwayd.service > /dev/null << EOF
[Unit]
Description=Archway node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which archwayd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable archwayd.service
```

**Start the service and check the logs**
```
sudo systemctl start archwayd.service
sudo journalctl -u archwayd.service -f --no-hostname -o cat
Secure Server Setup (Optional)
```

**generate ssh keys, if you don't have them already, DO IT ON YOUR LOCAL MACHINE**
```
ssh-keygen -t rsa
```

**save the output, we'll use it later on instead of YOUR_PUBLIC_SSH_KEY**
```
cat ~/.ssh/id_rsa.pub
# upgrade system packages
sudo apt update
sudo apt upgrade -y
```

**add new admin user**
```
sudo adduser admin --disabled-password -q
```

**upload public ssh key, replace YOUR_PUBLIC_SSH_KEY with the key above**
```
mkdir /home/admin/.ssh
echo "YOUR_PUBLIC_SSH_KEY" >> /home/admin/.ssh/authorized_keys
sudo chown admin: /home/admin/.ssh
sudo chown admin: /home/admin/.ssh/authorized_keys

echo "admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

**disable root login, disable password authentication, use ssh keys only**
```
sudo sed -i 's|^PermitRootLogin .*|PermitRootLogin no|' /etc/ssh/sshd_config
sudo sed -i 's|^ChallengeResponseAuthentication .*|ChallengeResponseAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PasswordAuthentication .*|PasswordAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PermitEmptyPasswords .*|PermitEmptyPasswords no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PubkeyAuthentication .*|PubkeyAuthentication yes|' /etc/ssh/sshd_config

sudo systemctl restart sshd
```

**install fail2ban**
```
sudo apt install -y fail2ban
```

**install and configure firewall**
```
sudo apt install -y ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9100
sudo ufw allow 26656
```

**make sure you expose ALL necessary ports, only after that enable firewall**
```
sudo ufw enable
```

**make terminal colorful**
sudo su - admin
source <(curl -s https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/enable_colorful_bash.sh)

# update servername, if needed, replace YOUR_SERVERNAME with wanted server name
sudo hostnamectl set-hostname YOUR_SERVERNAME

# now you can logout (exit) and login again using ssh admin@YOUR_SERVER_IP
