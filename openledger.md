### setup xrdp

```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

```
sudo apt install xfce4 xfce4-goodies
sudo apt install xrdp
sudo systemctl enable xrdp
sudo systemctl start xrdp
sudo adduser root ssl-cert
sudo apt install gdebi
```

### Open pake RDP sesuai IP VPS masing2, lalu buka terminal

```
wget https://cdn.openledger.xyz/openledger-node-1.0.0-linux.zip
```

```
unzip openledger-node-1.0.0-linux.zip
```

```
sudo dpkg -i openledger-node-1.0.0.deb
```

```
openledger-node --no-sandbox
```
