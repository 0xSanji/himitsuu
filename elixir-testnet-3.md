## Install dokcer kalo belum ada
```
# Install Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version

# Install Docker-Compose
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version

# Docker Permission to user
sudo groupadd docker
sudo usermod -aG docker $USER
```

## Download images docker
```
docker pull elixirprotocol/validator:v3
```

## setup env
```
mkdir elixir
cd elixir
nano validator.env
```

isi file nya pake ini
```
ENV=testnet-3

STRATEGY_EXECUTOR_IP_ADDRESS=<IP_ADDRESS>
STRATEGY_EXECUTOR_DISPLAY_NAME=<Nama Node>
STRATEGY_EXECUTOR_BENEFICIARY=<ADDRESS_WALLET_REWARD>
SIGNER_PRIVATE_KEY=<PK_WALLET_VALIDATOR>
```

## Run
Jangan lupa isi wallet validatornya pake eth sepolia 
```
docker run -d --env-file validator.env --name elixir -p 17690:17690 elixirprotocol/validator:v3
```

## stake ke validator kita
Web : https://testnet-3.elixir.xyz/
- Mint Mock
- Stake
- Delegate ke validator kita cari pake address reward

wallet reward sama validator boleh beda, sama juga gpp sih buat keamanan aja dibedain di docs nya

Dono cik

## Note untuk yang sudah run kemarin
- remove container lama
```
docker stop elixir && docker rm elixir
```
- Download images docker

```
docker pull elixirprotocol/validator:v3
```

- run ulang
```
cd $HOME/elixir
docker run -d --env-file validator.env --name elixir -p 17690:17690 elixirprotocol/validator:v3
```
