# Here's a simple automatic installer script for the Story Node setup. You can run this script in your terminal

#!/bin/bash

# Update and install required packages
sudo apt update && sudo apt install -y curl git make jq build-essential gcc unzip wget lz4 aria2

# Install Story-Geth
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzvf geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
mkdir -p $HOME/go/bin
sudo cp geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile

# Install Story
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
source $HOME/.bash_profile

# Assign MONIKER (change YOUR_MONIKER)
read -p "Enter your MONIKER: " MONIKER
story init --network iliad --moniker "$MONIKER"

# Create service files
echo -e "[Unit]\nDescription=Story Geth Client\nAfter=network.target\n\n[Service]\nUser=root\nExecStart=/root/go/bin/story-geth --iliad --syncmode full\nRestart=on-failure\nRestartSec=3\nLimitNOFILE=4096\n\n[Install]\nWantedBy=multi-user.target" | sudo tee /etc/systemd/system/story-geth.service

echo -e "[Unit]\nDescription=Story Consensus Client\nAfter=network.target\n\n[Service]\nUser=root\nExecStart=/root/go/bin/story run\nRestart=on-failure\nRestartSec=3\nLimitNOFILE=4096\n\n[Install]\nWantedBy=multi-user.target" | sudo tee /etc/systemd/system/story.service

# Start Story-Geth and Story services
sudo systemctl daemon-reload
sudo systemctl start story-geth
sudo systemctl enable story-geth
sudo systemctl start story
sudo systemctl enable story

# Check status
curl localhost:26657/status | jq
