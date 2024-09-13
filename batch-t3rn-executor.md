# BATCH EXECUTOR TERN WITH DOCKER

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
DEFAULT_ENABLED_NETWORKS="arbitrum-sepolia,base-sepolia,blast-sepolia,optimism-sepolia,l1rn"
DEFAULT_CONTAINER_NAME="t3rn-executor-container"
DEFAULT_VERSION="v0.21.0"  # Default version to download

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

# Prompt for container names and private keys
read -p "Enter container names (comma-separated, e.g., container1,container2): " CONTAINER_NAMES
IFS=',' read -r -a CONTAINER_NAME_ARRAY <<< "$CONTAINER_NAMES"

read -p "Enter PRIVATE_KEY_LOCAL values (comma-separated, e.g., key1,key2): " PRIVATE_KEYS
IFS=',' read -r -a PRIVATE_KEY_ARRAY <<< "$PRIVATE_KEYS"

if [ ${#CONTAINER_NAME_ARRAY[@]} -ne ${#PRIVATE_KEY_ARRAY[@]} ]; then
  echo "Number of container names must match the number of PRIVATE_KEY_LOCAL values."
  exit 1
fi

# Create .env file template
ENV_FILE_TEMPLATE="env_template"

# Create Docker images
IMAGE_NAME="t3rn-executor"
echo "Building Docker image..."
docker build -t $IMAGE_NAME .

# Run containers
for i in "${!CONTAINER_NAME_ARRAY[@]}"; do
  CONTAINER_NAME=${CONTAINER_NAME_ARRAY[$i]}
  PRIVATE_KEY_LOCAL=${PRIVATE_KEY_ARRAY[$i]}
  
  # Create .env file with default values and user input
  cat <<EOF > $ENV_FILE_TEMPLATE
NODE_ENV=$DEFAULT_NODE_ENV
LOG_LEVEL=$DEFAULT_LOG_LEVEL
LOG_PRETTY=$DEFAULT_LOG_PRETTY
PRIVATE_KEY_LOCAL=$PRIVATE_KEY_LOCAL
ENABLED_NETWORKS=$DEFAULT_ENABLED_NETWORKS
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
