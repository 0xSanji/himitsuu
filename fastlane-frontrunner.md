# FASTLANE FRONTRUNNER BOT

```
sudo apt update & sudo apt upgrade -y

sudo apt install screen

sudo apt install python3

sudo apt install python3

sudo ln -sf $(which python3) /usr/bin/python
sudo ln -sf $(which pip3) /usr/bin/pip
```

```
git clone https://github.com/FastLane-Labs/break-monad-frontrunner-bot-py.git
cd break-monad-frontrunner-bot-py
pip install -r requirements.txt

nano settings.toml
```

Isi pake ini
```
[api_settings]
rpc_url = 'https://testnet-rpc.monad.xyz'

[game_settings]
frontrunner_contract_address = '0x9EaBA701a49adE7525dFfE338f0C7E06Eca7Cf07'
abi_string = '[{"type":"function","name":"frontrun","inputs":[],"outputs":[],"stateMutability":"nonpayable"},{"type":"function","name":"getScore","inputs":[{"name":"_address","type":"address","internalType":"address"}],"outputs":[{"name":"","type":"tuple","internalType":"struct Frontrunner.ParticipantData","components":[{"name":"Address","type":"address","internalType":"address"},{"name":"Wins","type":"uint256","internalType":"uint256"},{"name":"Losses","type":"uint256","internalType":"uint256"}]}],"stateMutability":"view"},{"type":"function","name":"getScores","inputs":[],"outputs":[{"name":"","type":"tuple[]","internalType":"struct Frontrunner.ParticipantData[]","components":[{"name":"Address","type":"address","internalType":"address"},{"name":"Wins","type":"uint256","internalType":"uint256"},{"name":"Losses","type":"uint256","internalType":"uint256"}]}],"stateMutability":"view"}]'

[eoa]
private_key = 'your_private_key_here'
```


Run
```
screen -S fastlane
python play.py
```
