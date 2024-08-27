## Install Dependencies

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

## Get accuser image

```
docker pull nillion/retailtoken-accuser:v1.0.0
```

## Initialize accuser

```
mkdir -p nillion/accuser
docker run -v ./nillion/accuser:/var/tmp nillion/retailtoken-accuser:v1.0.0 initialise
```

will generate accountId and pubKey for submit on website: https://verifier.nillion.com/verifier

## Fund accuser address

fund your accuser address (accountId) with: https://faucet.testnet.nillion.com/

## Run Accuser

YOU MUST WAIT 30-60 MINUTES TO CONTINUE WITH THE STEPS BELOW. The secret verification is designed wait for a period of time before fully registering the accuser.

```
docker run -v ./nillion/accuser:/var/tmp nillion/retailtoken-accuser:v1.0.0 accuse --rpc-endpoint "http://51.89.195.146:26657" --block-start 5023281
```

## CheatSheet

- Check your accountId and PubKey (your credentials)

```
cat $HOME/nillion/accuser/credentials.json
```
