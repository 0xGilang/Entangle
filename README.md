# Installation
# **Auto Install:**
```
curl https://revonode.xyz/sc/entangle | bash
```
# Manual Install:
# **Server preparation**
- Updating packages
```
sudo apt update && sudo apt upgrade -y
```
- Install developer tools and necessary packages
```
sudo apt install curl build-essential pkg-config libssl-dev git wget jq make gcc tmux chrony -y
```
- Installing GO
```
ver="1.20"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
# Node installation
- Clone the project repository with the node, go to the project folder and collect the binary files
```
cd $HOME
rm -rf entangle-blockchain/
git clone https://github.com/Entangle-Protocol/entangle-blockchain
cd entangle-blockchain/
make install
```
- Checking the version
```
entangled version
#1.0.1
```
- Creating Variables
```
MONIKER_ENTANGLE=type in your name
CHAIN_ID_ENTANGLE=entangle_33133-1
PORT_ENTANGLE=37
```
- Save variables, reload .bash_profile and check variable values
```
echo "export MONIKER_ENTANGLE="${MONIKER_ENTANGLE}"" >> $HOME/.bash_profile
echo "export CHAIN_ID_ENTANGLE="${CHAIN_ID_ENTANGLE}"" >> $HOME/.bash_profile
echo "export PORT_ENTANGLE="${PORT_ENTANGLE}"" >> $HOME/.bash_profile
source $HOME/.bash_profile

echo -e "\nmoniker_ENTANGLE > ${MONIKER_ENTANGLE}.\n"
echo -e "\nchain_id_ENTANGLE > ${CHAIN_ID_ENTANGLE}.\n"
echo -e "\nport_ENTANGLE > ${PORT_ENTANGLE}.\n"
```
- Setting up the config
```
entangled config chain-id $CHAIN_ID_ENTANGLE
entangled config keyring-backend test
entangled config node tcp://localhost:${PORT_ENTANGLE}657
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025aNGL\"/" $HOME/.entangled/config/app.toml
```
- Initialize the node
```
entangled init $MONIKER_ENTANGLE --chain-id $CHAIN_ID_ENTANGLE
```
Loading the genesis file and address book
```
sudo cp config/genesis.json $HOME/.entangled/config
curl -Ls https://snapshots.indonode.net/entangle/addrbook.json > $HOME/.entangled/config/addrbook.json
```
- Adding seeds and peers
```
sed -i -e "s|^seeds *=.*|seeds = \"76492a1356c14304bdd7ec946a6df0b57ba51fe2@3.92.0.61:26656\"|" $HOME/.entangled/config/config.toml
```
- Setting up pruning
```
PRUNING="custom"
PRUNING_KEEP_RECENT="100"
PRUNING_INTERVAL="19"

sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.entangled/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$PRUNING_KEEP_RECENT\"/" $HOME/.entangled/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$PRUNING_INTERVAL\"/" $HOME/.entangled/config/app.toml
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.entangled/config/config.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.entangled/config/config.toml
```
- Create a service file
```
printf "[Unit]
Description=ENTANGLE Service
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=$(which entangled) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/entangled.service
```
- Start the service and check the logs
```
sudo systemctl daemon-reload && \
sudo systemctl enable entangled && \
sudo systemctl restart entangled && \
sudo journalctl -u entangled -f -o cat
```
- We are waiting for the end of synchronization, you can check the synchronization with the command
```
entangled status 2>&1 | jq .SyncInfo
```
If the output shows false, the sync is complete.
# Creating a wallet and validator
- Create a wallet
entangled keys add $MONIKER_ENTANGLE
Save the mnemonic phrase in a safe place!

If you participated in previous testnets, restore the wallet with the command and enter the mnemonic phrase
```
entangled keys add $MONIKER_ENTANGLE --recover
```
- Create a variable with the address of the wallet and validator
```
WALLET_ENTANGLE=$(entangled keys show $MONIKER_ENTANGLE -a)
VALOPER_ENTANGLE=$(entangled keys show $MONIKER_ENTANGLE --bech val -a)

echo "export WALLET_ENTANGLE="${WALLET_ENTANGLE}"" >> $HOME/.bash_profile
echo "export VALOPER_ENTANGLE="${VALOPER_ENTANGLE}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
echo -e "\nwallet_ENTANGLE > ${WALLET_ENTANGLE}.\n"
echo -e "\nvaloper_ENTANGLE > ${VALOPER_ENTANGLE}.\n"
```
- Checking your balance
```
entangled q bank balances $WALLET_ENTANGLE
```
- After the synchronization is completed and the wallet is replenished, we create a validator
```
entangled tx staking create-validator \
  --amount "1000000000000000000aNGL" \
  --pubkey $(entangled tendermint show-validator) \
  --moniker "MONIKER" \
  --identity "KEYBASE_ID" \
  --details "YOUR DETAILS" \
  --website "YOUR WEBSITE" \
  --chain-id entangle_33133-1 \
  --commission-rate "0.05" \
  --commission-max-rate "0.20" \
  --commission-max-change-rate "0.01" \
  --min-self-delegation "1" \
  --gas-prices "10aNGL" \
  --gas "500000" \
  --gas-adjustment "1.5" \
  --from wallet \
  -y
```
# Deleting a node
- Before deleting a node, make sure that the files from the ~/.entangled/config directory are saved To remove a node, use the following commands
```
sudo systemctl stop entangled
sudo systemctl disable entangled
sudo rm -rf $HOME/.entangled
sudo rm -rf $HOME/ENTANGLE
sudo rm -rf /etc/systemd/system/entangled.service
sudo rm -rf /usr/local/bin/entangled
sudo systemctl daemon-reload
```

<img src="https://miro.medium.com/v2/resize:fit:561/1*oXy3gLuaF2CK9y289TZDYQ.png">

# **Contact Me🔥☕**
<p align="left">
<a href="https://www.github.com/0xGilang"><img height="40" width="40" align="center" src="https://avatars.githubusercontent.com/u/94946818?v=4"></a>
<a href="https://www.facebook.com/Gilangbuanasultoni" target="blank"><img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/facebook.svg" alt="AryaTrickers2020" height="30" width="40" /></a>
<a href="https://wa.me/6282131561458?text=Halo+Bang+Gilang" target="blank"><img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/whatsapp.svg" alt="6289694295787" height="30" width="40" /></a>
<a href="https://twitter.com/Gilangsultoni"><img height="40" width="40" align="center" src="https://static.dezeen.com/uploads/2023/07/x-logo-twitter-elon-musk_dezeen_2364_col_0.jpg"></a>

------
------
