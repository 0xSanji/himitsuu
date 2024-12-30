# BATCH EXECUTOR TERN WITH DOCKER

## Install Docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version

VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)
curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version

sudo groupadd docker
sudo usermod -aG docker $USER
```

## buat folder
```
mkdir t3rn-batch-executor
cd t3rn-batch-executor
```

## buat script
```
nano batch.sh
```

- isi pake ini
```
#!/bin/bash

# Default values
DEFAULT_NODE_ENV="testnet"
DEFAULT_LOG_LEVEL="debug"
DEFAULT_LOG_PRETTY="false"
DEFAULT_ENABLED_NETWORKS="base-sepolia,optimism-sepolia,l1rn"
DEFAULT_CONTAINER_NAME="t3rn-executor-container"
DEFAULT_VERSION="v0.31.0"  # Default version to download

# Define the Dockerfile content
DOCKERFILE_CONTENT=$(cat <<EOF
# Use a specific version of Debian Slim
FROM debian:bookworm-slim

# Set the working directory
WORKDIR /app

# Copy the entire executor directory into the container
COPY executor /app/executor

# Ensure the binary has executable permissions
RUN chmod +x /app/executor/executor/bin/executor

# Default command to run the binary (can be overridden)
CMD ["/app/executor/executor/bin/executor"]
EOF
)

# Define the entrypoint.sh content
ENTRYPOINT_CONTENT=$(cat <<EOF
#!/bin/sh

# Export environment variables
export NODE_ENV=\${NODE_ENV}
export LOG_LEVEL=\${LOG_LEVEL}
export LOG_PRETTY=\${LOG_PRETTY}
export PRIVATE_KEY_LOCAL=\${PRIVATE_KEY_LOCAL}
export ENABLED_NETWORKS=\${ENABLED_NETWORKS}
export EXECUTOR_PROCESS_ORDERS=true
export EXECUTOR_PROCESS_CLAIMS=true
export EXECUTOR_MAX_L3_GAS_PRICE=500
export RPC_ENDPOINTS_ARBT='https://sepolia-rollup.arbitrum.io/rpc,https://endpoints.omniatech.io/v1/arbitrum/sepolia/public'
export RPC_ENDPOINTS_BSSP='https://sepolia.base.org,https://base-sepolia-rpc.publicnode.com'
export RPC_ENDPOINTS_BLSS='https://sepolia.blast.io,https://endpoints.omniatech.io/v1/blast/sepolia/public'
export RPC_ENDPOINTS_OPSP='https://sepolia.optimism.io,https://endpoints.omniatech.io/v1/op/sepolia/public'

# Execute the binary
exec /app/executor/executor/bin/executor
EOF
)

# Prompt for executor version
read -p "Enter executor version to download (default: $DEFAULT_VERSION): " VERSION
VERSION=${VERSION:-$DEFAULT_VERSION}

# Define download URL
EXECUTOR_URL="https://github.com/t3rn/executor-release/releases/download/${VERSION}/executor-linux-${VERSION}.tar.gz"

# Download the executor binary
echo "Downloading executor version $VERSION from $EXECUTOR_URL..."
wget "$EXECUTOR_URL" -O executor-linux.tar.gz

# Extract the executor binary
echo "Extracting the binary..."
tar -xzvf executor-linux.tar.gz

# Check if the extraction was successful
if [ ! -d "executor" ]; then
  echo "Failed to download or extract the executor binary. Please check the version or URL."
  exit 1
fi

# Make sure the executor binary has the correct permissions
chmod +x executor/executor/bin/executor

# Create Dockerfile
echo "$DOCKERFILE_CONTENT" > Dockerfile

# Create entrypoint.sh
echo "$ENTRYPOINT_CONTENT" > entrypoint.sh

# Make entrypoint.sh executable
chmod +x entrypoint.sh

# Prompt for private keys
read -p "Enter PRIVATE_KEY_LOCAL values (comma-separated, e.g., key1,key2): " PRIVATE_KEYS
IFS=',' read -r -a PRIVATE_KEY_ARRAY <<< "$PRIVATE_KEYS"

# Create .env file template
ENV_FILE_TEMPLATE="env_template"

# Create Docker images
IMAGE_NAME="t3rn-executor"
echo "Building Docker image..."
docker build -t $IMAGE_NAME .

# Run containers
for i in "${!PRIVATE_KEY_ARRAY[@]}"; do
  CONTAINER_NAME=executor-$((i + 1))
  PRIVATE_KEY_LOCAL=${PRIVATE_KEY_ARRAY[$i]}
  
  # Create .env file with default values and user input
  cat <<EOF > $ENV_FILE_TEMPLATE
NODE_ENV=$DEFAULT_NODE_ENV
LOG_LEVEL=$DEFAULT_LOG_LEVEL
LOG_PRETTY=$DEFAULT_LOG_PRETTY
PRIVATE_KEY_LOCAL=$PRIVATE_KEY_LOCAL
ENABLED_NETWORKS=$DEFAULT_ENABLED_NETWORKS
EXECUTOR_PROCESS_ORDERS=true
EXECUTOR_PROCESS_CLAIMS=true
EXECUTOR_MAX_L3_GAS_PRICE=500
RPC_ENDPOINTS_ARBT='https://sepolia-rollup.arbitrum.io/rpc,https://endpoints.omniatech.io/v1/arbitrum/sepolia/public'
RPC_ENDPOINTS_BSSP='https://sepolia.base.org,https://base-sepolia-rpc.publicnode.com'
RPC_ENDPOINTS_BLSS='https://sepolia.blast.io,https://endpoints.omniatech.io/v1/blast/sepolia/public'
RPC_ENDPOINTS_OPSP='https://sepolia.optimism.io,https://endpoints.omniatech.io/v1/op/sepolia/public'
EOF

  # Run the Docker container with the .env file
  echo "Running Docker container '$CONTAINER_NAME' with .env file: $ENV_FILE_TEMPLATE"
  docker run -d \
    --restart always \
    --name $CONTAINER_NAME \
    --env-file $ENV_FILE_TEMPLATE \
    -v "$(pwd)/entrypoint.sh:/app/entrypoint.sh" \
    --entrypoint /app/entrypoint.sh \
    $IMAGE_NAME

  # Clean up the .env file
  rm $ENV_FILE_TEMPLATE
done

# Clean up the .tar.gz file
rm executor-linux.tar.gz

echo "All Docker containers are running."
```

- permission file
```
chmod +x batch.sh
```

- run
```
./batch.sh
```
