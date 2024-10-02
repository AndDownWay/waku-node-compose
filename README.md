# Run Nwaku with Docker Compose

## Prerequisites

Before you start, ensure you have the following installed:

- Docker: Download and install Docker.

- Docker Compose: Download and install Docker Compose.

You can check if Docker and Docker Compose are installed properly by running:

```
docker --version
docker-compose --version
```
## 1. Setup Project Directory

Start by creating a new directory for your nwaku project. This directory will hold all your Docker Compose configuration files.

```
mkdir nwaku-docker
cd nwaku-docker
```
## 2. Create Docker Compose File

Inside your project directory, create a docker-compose.yml file. This file defines how Docker Compose will configure and manage the services.

```
touch docker-compose.yml
```
Now open this file in a text editor and add the following configuration:

```
version: '3'
services:
  nwaku:
    image: statusteam/nim-waku:v0.13.0  # You can update to the latest version
    container_name: nwaku_node
    ports:
      - "60000:60000/tcp"    # Expose TCP port
      - "60000:60000/udp"    # Expose UDP port
    command: "nwaku --log-level=info --rpc-admin --rpc-address=0.0.0.0 --rpc-port=8545 --metrics-server --metrics-port=8008"
    volumes:
      - ./data:/data            # Persist nwaku data on the host
    networks:
      - waku-net
    restart: unless-stopped

networks:
  waku-net:
    driver: bridge
```

This configuration does the following:

- Image: Uses the official nwaku image (statusteam/nim-waku:v0.13.0).

- Container Name: Names the container nwaku_node.

- Ports: Exposes the necessary TCP/UDP ports (60000 by default) and the RPC port (8545) and metrics server (8008).

- Volumes: Mounts a directory (./data) from your host to the container for persisting data across container restarts.

- Networks: Uses a custom Docker network called waku-net for isolating and managing the container's connectivity.

## 3. Pull the Docker Image

Once you’ve configured the docker-compose.yml file, pull the required nwaku image:

```
docker-compose pull
```
This command ensures you have the latest version of the Waku node Docker image.

## 4. Run Docker Compose

Now you are ready to run nwaku with Docker Compose. Use the following command to spin up the container:

```
docker-compose up -d
```
The -d flag runs the container in the background (detached mode).

## 5. Verify nwaku is Running

Check if the container is running by listing active containers:

```
docker ps
```
You should see an entry similar to:

```
CONTAINER ID   IMAGE                    COMMAND                  STATUS            PORTS                    NAMES
123abc456def   statusteam/nim-waku:v0.13.0   "nwaku --log-level=…"   Up 2 minutes        0.0.0.0:60000->60000/tcp, 0.0.0.0:60000->60000/udp, 0.0.0.0:8545->8545/tcp, 0.0.0.0:8008->8008/tcp   nwaku_node
```
## 6. Accessing Logs

To view the logs of the running nwaku container, use:

```
docker-compose logs -f
```
This will show real-time logs of the container, which can be useful for monitoring and troubleshooting.

## 7. Connecting to the nwaku Node

a. RPC Interface

The nwaku node exposes an RPC interface on port 8545. You can interact with it using an RPC client. For example, you can check if the service is running correctly with:

```
curl -X POST http://localhost:8545 -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"get_waku_v2_debug_info","params":[],"id":1}'
```
You should get a response with information about the node.

b. Metrics Interface

nwaku also exposes metrics on port 8008. You can access the metrics using a browser or curl:

```
curl http://localhost:8008/metrics
```
This will provide Prometheus-style metrics data, useful for monitoring.

## 8. Stopping and Restarting Containers

To stop the running containers:

```
docker-compose down
```
To stop the container but keep the state:
```
docker-compose stop
```
To restart after stopping:
```
docker-compose start
```
## 9. Customizing Configuration

If you need to customize the nwaku node further (e.g., changing ports, enabling additional services), you can modify the command in the docker-compose.yml file. For example, to enable WebSocket, you can add the following options:

```
command: "nwaku --log-level=info --rpc-admin --rpc-address=0.0.0.0 --rpc-port=8545 --ws --ws-port=8546 --metrics-server --metrics-port=8008"
```
This will expose the WebSocket interface on port 8546.

## 10. Persistence and Volumes

The volumes configuration in the docker-compose.yml file ensures that data (such as node state, logs, etc.) persists even after the container is stopped or removed. The data is stored on your local machine under the ./data directory. This way, you don't lose your node's state across restarts.

## 11. Scaling nwaku

You can easily scale the nwaku service if needed. For example, if you want to run multiple instances of nwaku, you can scale the service by running:

```
docker-compose up --scale nwaku=3 -d
```
This command runs three instances of the nwaku container in parallel.

## 12. Troubleshooting

If you encounter issues, here are some tips:

- Check logs: Always check the logs with docker-compose logs -f to see any error messages.
- Port conflicts: Ensure the ports (60000, 8545, 8008) are not already in use by other services. You can change the ports in the docker-compose.yml file if necessary.
- Container status: Use docker ps -a to check if the container exited or crashed unexpectedly.
  
Example Troubleshooting

- Problem: nwaku container keeps restarting.

- Solution: Check the logs for errors:

```
docker-compose logs -f
```
If there’s an issue with binding ports or configuration errors, you can adjust the docker-compose.yml file to fix them.

## 13. Updating nwaku

To update the nwaku version, simply update the image version in the docker-compose.yml file:

```
image: statusteam/nim-waku:v0.13.1  # Update to the latest version
```
Then run the following commands to apply the update:

```
docker-compose pull
docker-compose up -d
```
This will pull the latest version of the image and recreate the container with the updated version.
