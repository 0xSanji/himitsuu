## Install dependecies
```console
# Install Packages
sudo apt update & sudo apt upgrade -y

sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y
```
```console
# Install Python3
sudo apt install python3
python3 --version

sudo apt install python3-pip
pip3 --version
```
```console
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
```console
# Install Go
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

## Worker Allora
- clone repo allora
```
git clone https://github.com/allora-network/allora-huggingface-walkthrough.git
```

- masuk ke folder allora nya

```cd allora-huggingface-walkthrough```

- stop allora

```docker compose down -v```

- hapus app lama

```rm -rf app.py && nano app.py```

- isi pake ini, sama ganti <API_KEY_CG> (jangan ada <>)

```
from flask import Flask, Response
import requests
import json
import pandas as pd
import torch
from chronos import ChronosPipeline

# create our Flask app
app = Flask(__name__)

# define the Hugging Face model we will use
model_name = "amazon/chronos-t5-mini"

def get_coingecko_url(token):
    base_url = "https://api.coingecko.com/api/v3/coins/"
    token_map = {
        'ETH': 'ethereum',
        'SOL': 'solana',
        'BTC': 'bitcoin',
        'BNB': 'binancecoin',
        'ARB': 'arbitrum',
        'DEGEN': 'degen-base',
        'BRETT': 'based-brett',
        'TOSHI': 'toshi',
        'BENJI': 'basenji',
        'MIGGLES': 'mister-miggles',
        'BAMBOO': 'bamboo-on-base',
        'MFER': 'mfercoin',
        'WEWE': 'upside-down-meme',
        'CHOMP': 'chompcoin',
        'MOCHI': 'mochi-thecatcoin',
        'DOGINME': 'doginme',
        'TYBG': 'base-god',
        'ANDY': 'andy-on-base',
        'KEYCAT': 'keyboard-cat-base',
        'NORMIE': 'normie-2',
        'HIGHER': 'higher',
        'BOOMER': 'boomer',
        'CRASH': 'crash-on-base',
        'BASED': 'basedmilio',
        'TOBY': 'toby-toadgod',
        'PEEZY': 'young-peezy-aka-pepe',
        'MVP': 'maga-vp',
        'CONDO': 'condo',
        'TIMI': 'this-is-my-iguana',
        'MORK': 'mork-2',
        'CARLO': 'carlo',
        'WOLF': 'landwolf-base',
        'ALF': 'alf',
        'MOBY': 'moby-2',
        'BERF': 'moby-2',
        'WGC': 'wild-goat-coin-2',
        'SKI': 'ski-mask-dog',
        'COINYE': 'coinye-west',
        'ELONRWA': 'elonrwa',
        'AYB': 'all-your-base-2',
        'AEROBUD': 'aerobud',
        'FOMO': 'fomo-bull-club',
        'BCP': 'blockchainpeople',
        'ANIME': 'anime-base',
        'PONCHO': 'poncho',
        'MOB': 'marvin-on-base',
        'ROCK': 'just-a-black-rock-on-base',
        'BOSHI': 'boshi',
        'BENI': 'beni',
        'TYBENG': 'benji-bananas',
        'BASE': 'goodle',
        'BLUE': 'blue-guy',
        'LAMBOW': 'based-lambow',
        '$WORKIE': 'workie',
        'CORGI': 'corgi',
        'GIF': 'goatwifhat',
        'NIPPY': 'cat-on-catnip',
        'ROCCO': 'just-a-rock',
        'ROXY': 'roxy-frog',
        'MOON': 'moon-inu',
        'FLOWER': 'farcaster-flower',
        'BPS': 'base-pro-shops',
        'MEWNB': 'mewnb',
        'UBPS': 'united-base-postal',
        'MOONCATS': 'mooncats-on-base',
        'OKAYEG': 'okayeg',
        'APED': 'aped-3',
        'MOEW': 'moew',
        'CATS': 'lolcat',
        'POV': 'degen-pov-2',
        'BOE': 'boe',
        'USA': 'based-usa',
        'LOA': 'law-of-attraction',
        'DINO': 'coding-dino',
        'TREE': 'treeplanting',
        'POOP': 'poopcoin-poop',
        'ELEPEPE': 'elephantpepe',
        'SKOP': 'skull-of-pepe-token',
        'DADDY': 'crypto-journey',
        'BIRDDOG': 'bird-dog-on-base',
        'ROOST': 'roost',
        'BUTT': 'buttman',
        'BROGE': 'broge',
        'BALDO': 'bald-dog',
        'CHAD': 'based-chad',
        'BRIUN': 'briun-armstrung',
        '$BOSHI': 'based-boshi',
        'LOUDER': 'louder',
        'BSHIB': 'based-shiba-inu',
        'BORED': 'edison-bored',
        'DOG': 'basic-dog-meme',
        'BRETT2.0': 'brett-2-0',
        'BANANAS': 'monkey-peepo',
        'LONG': 'betterbelong',
        'NOGS': 'noggles',
        'POLA': 'pola',
        'KEREN': 'keren',
        'SACA': 'sandwich-cat',
        '$ROCKY': 'rocky-on-base',
        'CHUCK': 'chuck'
    }

    token = token.upper()
    if token in token_map:
        url = f"{base_url}{token_map[token]}/market_chart?vs_currency=usd&days=30&interval=daily"
        return url
    else:
        raise ValueError("Unsupported token")

# define our endpoint
@app.route("/inference/<string:token>")
def get_inference(token):
    """Generate inference for given token."""
    try:
        # use a pipeline as a high-level helper
        pipeline = ChronosPipeline.from_pretrained(
            model_name,
            device_map="auto",
            torch_dtype=torch.bfloat16,
        )
    except Exception as e:
        return Response(json.dumps({"pipeline error": str(e)}), status=500, mimetype='application/json')

    try:
        # get the data from Coingecko
        url = get_coingecko_url(token)
    except ValueError as e:
        return Response(json.dumps({"error": str(e)}), status=400, mimetype='application/json')

    headers = {
        "accept": "application/json",
        "x-cg-demo-api-key": "<API_KEY_CG>" # replace with your API key
    }

    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        data = response.json()
        df = pd.DataFrame(data["prices"])
        df.columns = ["date", "price"]
        df["date"] = pd.to_datetime(df["date"], unit='ms')
        df = df[:-1] # removing today's price
        print(df.tail(5))
    else:
        return Response(json.dumps({"Failed to retrieve data from the API": str(response.text)}),
                        status=response.status_code,
                        mimetype='application/json')

    # define the context and the prediction length
    context = torch.tensor(df["price"])
    prediction_length = 1

    try:
        forecast = pipeline.predict(context, prediction_length)  # shape [num_series, num_samples, prediction_length]
        print(forecast[0].mean().item()) # taking the mean of the forecasted prediction
        return Response(str(forecast[0].mean().item()), status=200)
    except Exception as e:
        return Response(json.dumps({"error": str(e)}), status=500, mimetype='application/json')

# run our Flask app
if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000, debug=True)

```

- save, lanjut

```rm -rf config.json && nano config.json```

- isi ini, sama ganti <SEED_PHRASE> (jangan ada <>) sama di topic 10 kan ada Token: DEGEN itu bisa ganti token meme base nya sama yg ada di file app.py sebelumnya, pilih salah satu, DEGEN udah jarang dpt pointnya ganti aja

```
{
  "wallet": {
    "addressKeyName": "testkey",
    "addressRestoreMnemonic": "<SEED_PHRASE>",
    "gas": "1000000",
    "gasAdjustment": 1.5,
    "nodeRpc": "https://rpc.ankr.com/allora_testnet",
    "maxRetries": 5,
    "delay": 1,
    "submitTx": true
  },
  "worker": [
    {
      "topicId": 1,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "ETH"
      }
    },
    {
      "topicId": 2,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "ETH"
      }
    },
    {
      "topicId": 3,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "BTC"
      }
    },
    {
      "topicId": 4,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "BTC"
      }
    },
    {
      "topicId": 5,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "SOL"
      }
    },
    {
      "topicId": 6,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "SOL"
      }
    },
    {
      "topicId": 7,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "ETH"
      }
    },
    {
      "topicId": 8,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "BNB"
      }
    },
    {
      "topicId": 9,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "ARB"
      }
    },
    {
      "topicId": 10,
      "inferenceEntrypointName": "api-worker-reputer",
      "loopSeconds": 5,
      "parameters": {
        "InferenceEndpoint": "http://inference:8000/inference/{Token}",
        "Token": "DEGEN"
      }
    }
  ]
}
```

- save terus jgn lupa run

```./init.config```

- biar confignya diperbarui

- terus run lagi

```docker compose up -d --build```

# dono

# Intinya semakin harga token yg di predict sepi yg nebak harusnya bisa gacor pointnya, makanya topic 10 dpt banyak point, kalo mulai sepi ganti lagi aja token di topic 10 sama jgn lupa run ./init.config

