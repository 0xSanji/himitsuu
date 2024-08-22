Download pingpong
```
wget https://pingpong-build.s3.ap-southeast-1.amazonaws.com/linux/latest/PINGPONG
```

Ambil key di https://mining.pingpong.build/devices
Ambil yg usage priority aja jgn mining
```
screen -S pingpong
```

```
chmod +x ./PINGPONG && ./PINGPONG --key your_device_key
```

Keluar screen dulu, tekan `ctrl + a + d`

Set Dawn Akun : isi email sama password
```
./PINGPONG config set --dawn.email=***  --dawn.pwd=***
./PINGPONG stop --depins=dawn
./PINGPONG start --depins=dawn
```

Note : pasti nanti depin lain ikut ke run kalo mau stop yg lain cek pake `docker ps` terus stop sesuai namanya, misal
```
./PINGPONG stop --depins=grass
```
