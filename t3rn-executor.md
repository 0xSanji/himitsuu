# TERN EXECUTOR WITH DOCKER

## Buat Folder
```
mkdir t3rn-executor
```

## Pindah ke folder
```
cd t3rn-executor
```

## Buat Dockerfile
```
nano Dockerfile
```

- isi file dengan ini
```
# Use a specific version of Debian Slim
FROM debian:bookworm-slim

# Set the working directory
WORKDIR /app

# Copy the entire executor directory into the container
COPY executor /app/executor

# Copy the entrypoint script into the container
COPY entrypoint.sh /app/entrypoint.sh

# Ensure the binary and the script have executable permissions
RUN chmod +x /app/executor/executor/bin/executor && chmod +x /app/entrypoint.sh

# Set the entry point for the container
ENTRYPOINT ["/app/entrypoint.sh"]
```

## Buat entrypoint
```
nano entrypoint.sh
```

- Isi file dengan ini, ganti pk
```
#!/bin/sh

# Export environment variables
export NODE_ENV=${NODE_ENV:-testnet}
export LOG_LEVEL=${LOG_LEVEL:-debug}
export LOG_PRETTY=${LOG_PRETTY:-false}
export PRIVATE_KEY_LOCAL=${PRIVATE_KEY_LOCAL:-PRIVATE_KEY}
export ENABLED_NETWORKS=${ENABLED_NETWORKS:-'arbitrum-sepolia,base-sepolia,blast-sepolia,optimism-sepolia,l1rn'}

# Execute the binary
exec /app/executor/executor/bin/executor
```

- Note: Jangan lupa isi tiap jaringan minimal 0.1 ETH, l1rn itu brn, isi juga 0.1, ada tanda "-" depan PK itu jgn dihapus

## Download binary
```
wget https://github.com/t3rn/executor-release/releases/download/v0.21.0/executor-linux-v0.21.0.tar.gz && tar -xzvf executor-linux-v0.21.0.tar.gz
```

## Build Images dan Run
- build images
```
docker build -t executor-t3rn .
```

- run docker
```
docker run -d --env-file .env --restart always --name executor executor-t3rn
```

- cek logs
```
docker logs -f executor
```
