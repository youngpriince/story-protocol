# story-protocol
![image](https://github.com/user-attachments/assets/6b73e2c3-495a-470c-9de7-fef8f75c6ddc)

### Hardware requirements

| **Hardware**        | **Requirements**              |
|------------------------|--------------------------|
| **Operating System**   | Linux (linux/amd64, linux/arm64) |
| **vCPU**               | 4                        |
| **RAM**                | 8GB                       |
| **Disk Space**         | 200GB                     |
| **Bandwith**         | 10 MBit/s                |

### Install dependencies
```console
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 -y
```
### Download story-geth binary
```console
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzvf geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
source $HOME/.bash_profile
story-geth version
```
### Download story binary
```console
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
source $HOME/.bash_profile
story version
```
### Init iliad node
```console
story init --network iliad --moniker "Your_moniker_name"
```
### Create story-geth service file
```console
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
### Create story service file
```console
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
### Reload and start story-geth service
```console
sudo systemctl daemon-reload && \
sudo systemctl start story-geth && \
sudo systemctl enable story-geth && \
sudo systemctl status story-geth
```
### Reload and start story service
```console
sudo systemctl daemon-reload && \
sudo systemctl start story && \
sudo systemctl enable story && \
sudo systemctl status story
```
### Check logs
```console
# for story-geth
sudo journalctl -u story-geth -f -o cat
```
```console
# for story
sudo journalctl -u story -f -o cat
```
### Check sync status
```console
curl localhost:26657/status | jq
```
![image](https://github.com/user-attachments/assets/223ee412-3a9c-469e-95b7-fa8c3d660e60)

* Wait for `catching_up` to turn `false` before you create validator
## Create Validator
Export private key
By default, when you run `story init` a validator key is created for you. To view your validator key, run the following command:
```console
story validator export
```
In addition, if you want to export the derived EVM private key of your validator into the default data config directory, please run the following
```console
story validator export --export-evm-key
```
* Please keep your private key in a safe place. Your EVM Private Key saved to `/root/.story/story/config/private_key.txt`
* Paste the private key in metamask and request faucet
* Note that to participate in consensus, at least 1 IP must be staked (equivalent to 1000000000000000000 wei)!
* Faucet link: https://faucet.story.foundation/
```console
story validator create --stake 1000000000000000000 --private-key "your_private_key"
```
Backup the validator key `/root/.story/story/config/priv_validator_key.json`
Copy this file and store it carefully; this is the most crucial key for your validator.
### Validator staking
```console
story validator stake \
   --validator-pubkey "VALIDATOR_PUB_KEY_IN_BASE64" \
   --stake 1000000000000000000 \
   --private-key xxxxxxxxxxxxxx
```
* Replace `VALIDATOR_PUB_KEY_IN_BASE64` from what you copied earlier

Check your Validator on Explorer: https://testnet.story.explorers.guru/
Paste HEX Validator Address to search
![image](https://github.com/user-attachments/assets/ac896c00-6091-4cb9-9f66-c4071017ce0e)

