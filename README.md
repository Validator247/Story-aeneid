# Story-aeneid

🌐RPC cosmos:  https://Story-eaneid-testnet-rpc.validator247.com

🔧API :   https://story-eaneid-testnet-api.validator247.com

⚙️RPC EVM : https://story-eaneid-evm-rpc.validator247.com

    📡 WebSocket Secure: [wss://story-eaneid-evm-ws.validator247.com](wss://story-eaneid-evm-ws.validator247.com)



  # install go
    cd $HOME
    VER="1.22.5"
    wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
    rm "go$VER.linux-amd64.tar.gz"
    [ ! -f ~/.bash_profile ] && touch ~/.bash_profile
    echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
    source $HOME/.bash_profile
    [ ! -d ~/go/bin ] && mkdir -p ~/go/bin

# set vars
    echo "export MONIKER="Node_name"" >> $HOME/.bash_profile
    echo "export STORY_CHAIN_ID="aeneid"" >> $HOME/.bash_profile
    echo "export STORY_PORT="26"" >> $HOME/.bash_profile
    source $HOME/.bash_profile

# download binaries
    cd $HOME
    rm -rf story-geth
    git clone https://github.com/piplabs/story-geth.git
    cd story-geth
    git checkout v1.1.0
    make geth
    mv build/bin/geth  $HOME/go/bin/
    [ ! -d "$HOME/.story/story" ] && mkdir -p "$HOME/.story/story"
    [ ! -d "$HOME/.story/geth" ] && mkdir -p "$HOME/.story/geth"

# install Story
    cd $HOME
    rm -rf story
    git clone https://github.com/piplabs/story
    cd story
    git checkout v1.3.0
    go build -o story ./client 
    mkdir -p $HOME/go/bin/
    mv $HOME/story/story $HOME/go/bin/

# init story app
    story init --moniker $MONIKER --network $STORY_CHAIN_ID

# set peers
    PEERS="83818af9c35d3543b2c8b91c14b33bc7332617cd@57.129.88.113:26656,b4b3f2d3bf950c3f6819183471c22e425d7213b4@148.72.167.173:26646,7160dec63da82b56e1ce59a93c057c05e361cf85@135.181.117.37:64656,311cd3903e25ab85e5a26c44510fbc747ab61760@152.53.87.97:36656,a8d01e154197d799637eca4f0f369dc215db6b70@144.76.111.9:26656,a7c38e322fff3f264b9163db475ba42b1b48e765@65.21.97.155:26656,9d34ab3819aa8baa75589f99138318acfa0045f5@95.217.119.251:30900,8b8f8d6fb17b86499c491751975fe27234e60809@65.108.75.52:62656,dfb96be7e47cd76762c1dd45a5f76e536be47faa@65.108.45.34:32655,ae49103a54f77effa438978ad8a7ba09b6f20da0@144.76.202.120:35656,72c83daf61042571e1a98f6e474509157d9bdfe7@178.63.79.214:26656"
    sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml

# set custom ports in story.toml file
    sed -i.bak -e "s%:1317%:${STORY_PORT}317%g;
    s%:8551%:${STORY_PORT}551%g" $HOME/.story/story/config/story.toml

# set custom ports in config.toml file
    sed -i.bak -e "s%:26658%:${STORY_PORT}658%g;
    s%:26657%:${STORY_PORT}657%g;
    s%:26656%:${STORY_PORT}656%g;
    s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${STORY_PORT}656\"%;
    s%:26660%:${STORY_PORT}660%g" $HOME/.story/story/config/config.toml

# enable prometheus and disable indexing
    sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.story/story/config/config.toml
    sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.story/story/config/config.toml

 # Download Addrbook & Genesis

     wget -O ~/.story/config/addrbook.json https://raw.githubusercontent.com/Validator247/Story-aeneid/main/addrbook.json
     wget -O ~/.story/config/genesis.json https://raw.githubusercontent.com/Validator247/Story-aeneid/main/genesis.json

# create geth servie file
    sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
    [Unit]
    Description=Story Geth daemon
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$HOME/go/bin/geth --aeneid --syncmode full --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 0.0.0.0 --http.port ${STORY_PORT}545 --authrpc.port ${STORY_PORT}551 --ws --ws.api eth,web3,net,txpool --ws.addr 0.0.0.0 --ws.port ${STORY_PORT}546
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

# create story service file
    sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
    [Unit]
    Description=Story Service
    After=network.target

    [Service]
    User=$USER
    WorkingDirectory=$HOME/.story/story
    ExecStart=$(which story) run

    Restart=on-failure
    RestartSec=5
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

 # enable and start geth, story
    sudo systemctl daemon-reload
    sudo systemctl enable story story-geth
    sudo systemctl restart story-geth && sudo systemctl restart story

# check logs
    journalctl -u story -u story-geth -f -o cat

# Create validator

View your validator key

    story validator export

Export EVM private key

    story validator export --export-evm-key

Create validator, locked

    story validator create --stake 1024000000000000000000 --moniker $MONIKER --chain-id 1315 --unlocked=false

Create validator, unlocked

    story validator create --stake 1024000000000000000000 --moniker $MONIKER --chain-id 1315 --unlocked=true

Remember to backup your validator priv_key from here:

    cat $HOME/.story/story/config/priv_validator_key.json

View EVM private key and make a key backup

    cat $HOME/.story/story/config/private_key.txt
