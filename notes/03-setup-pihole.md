The setup was based on the official Pi-hole GitHub repository.
https://github.com/pi-hole/docker-pi-hole

# Pi-hole in Docker
The goal of this project is not only to deploy Pi-hole, but also to understand how Docker containers, Linux processes, networking and DNS work together.

## 1. Docker Compose configuration
```yml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # which network to connect the service pihole
    networks:
      - "pihole_net"
    # externally accessible ports
    ports:
      # pihole works as DNS server
      - "53:53/tcp"
      - "53:53/udp"
      # HTTP / HTTPs to use the web control interface
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: "Europe/Berlin"
      # password for logging into the pihole web control panel
      FTLCONF_webserver_api_password: "SuperSecretPassword"
      # allow pihole to respond to all clients on the network
      FTLCONF_dns_listeningMode: "ALL"
    # volumes for storing data (databases & configuration file) between container upgrades (host:container)
    volumes:
      - "./etc-pihole:/etc/pihole"
    # automatic container restart
    restart: unless-stopped

# creating networks for the project
networks:
  pihole_net:
    name: pihole_net
    driver: bridge
```

The Docker Compose file is written in YAML format, so it uses a dictionary-like structure and relies on indentation to define relationships between onfiguration sections.

The project currently contains a single service:
```
services:
  pihole:
```
The container name ```pihole``` is set so that Docker doesn't generate a random name. The service uses the lastes Pi-hole image.

### Network
The container is connected to a custom Docker bridge network. 
I chose the bridge driver because I'm currently using Pi-hole only as a DNS server. A bridge network provides network isolation while still allowing access through published ports.

In the future, if Pi-hole is also configured as a DHCP server, using host networking may be more appropriate because DHCP relies on broadcast traffic within the local network.

### Ports
Ports are published to make services inside the container accessible from the host system.

DNS:
```
53:53/tcp
53:53/udp
```
Web interface:
```
80:80/tcp
443:443/tcp
```
These mappings allow devices on the local network to access Pi-hole through Raspberry Pi IP address.

### Environment Variables
Environment variables are used to configure the container. They allow Pi-hole to be configured without modifying files inside the container.
Examples: 
- timezone
- web interface password
- DNS listening behavior

### Volumes
Bind mounts are used to persist data.
```
./etc-pihole:/etc/pihole
```
The directory on the Raspberry Pi and the directory inside the container refer to the same data.
This ensures that configuration files and databases survive container recreation and image updates.

### Restart Policy
The restart policy tells Docker to automatically restart the container if it stops unexpectedly.
This improves service availability without requiring manual intervention.

### Custom Network
A custom Docker network is defined explicitly.
Docker could create a network automatically, but defining it manually gives me full control over the network name and driver configuration.

## 2. Validating the Compose File
Before starting the container, I verified the Compose configuration.
From the directory where the compose.yml is located, command:
```
docker compose config
```
- Docker Compose reads the YAAML file
- it validates the syntax and configuration
- the command displays the final configuration that Docker Compose will use, including the default value

This is useful for detecting configuration mistakes before deployment.

## 3. Starting the Container
Command:
```
docker compose up -d
```
- Docker Compose creates the required resources
- the image is downloaded if it does not exist locally
- Docker creates the container
- Docker creates the network
- The container starts its main process
- the ```-d``` flag runs the conteiner in detached mode, allowing the terminal to be used for other commands

## 4. Inspecting the Container
Docker stores detailed metadata about containers. 
```
docker inspect pihole
```
The inspect command displays the container configuration and runtime stste.
Information includes:
- container state
- network configuration
- mounted volumes
- environment variables
- health checks
- port mappings
- process information

### Key Information revealed by docker inspect
The ```State``` section showed:
```
Status: running
Running: true
Health: healthy
```
- the container is running
- the main process exists
- the health check succeeds

The container received its own Process ID (PID) from the Linux kernel. A container may run multiple processes, but from the host system's perspective it is represented by its main process. 
```
Pid: 2222
```
- the PID shown by Docker is the PID of the main process
- restarting the container creates a new process and therefore a new PID

To see the processes and the process hierarchy (PID / PPID) inside the container, command:
```
docker top pihole
```
```
start.sh 
 ├── crond
 ├── bash
 │   └── pihole-FTL
 └── tail
```
Parent Process ID (PPID) identifies the process that started another process. 
for example, ```start.sh``` is the main process of the Pi-hole container and had its own PID. It starts and manages other processes required by the application. While processes such as ```crond, bash, tail``` used that PID as their PPID.
This makes it possible to view the process hierarchy inside the container and understand which processes were started by the main process.

The ```NetworkSettings``` section contains information about networking, port mappings, IP appresses and Docker networks.
#### Published ports
Docker showed port mapping similar to:
```
HostIp: 0.0.0.0
HostPort: 53/tcp
HostIp: ::
HostPort: 53/udp
```
- 0.0.0.0 - means all IPv4 interfaces on the host
- :: - all IPv6 interfaces in the host

Devices on the local network can access the container through the Raspberry Pi IP.
Docker creates forwarding rules between host ports and container ports.

#### Gateway
Docker assigned the gateway for the container network. It is the exit point from the Docker network.
The container uses the gateway when it needs to reach other networks.
For example, when Pi-hole needs to download updates or block lists from the Internet

#### Address
The container received its own IP inside the Docker bridge network. Devices on the local network do not access this address directly. Instead, they connect to the Raspberry Pi IP and Docker forwards the traffic to the container through published ports.

The container IP is mainly used inside the Docker network. External devices usually connect through the host IP, while containers can communicate directly using container IP or Docker DNS names.

## 5. Testing the Web Interface
Before configuring the router, I tested the Pi-hole web interface from a mobile phone.
I opened the following address in a web browser:
```
http://Raspberry_Pi_IP/admin
```
- the login page loaded successfully
- the Pi-hole web interface was running correctly inside the container
- port forwarding for HTTP was working as expected
- devices on the local network could access the service through the Raspberry Pi IP
- the web interface can be tested independently from DNS configuration

## 6. Testing DNS Functionality
Before changing any router settings, Iwanted to verify that Pi-hole was already working as a DNS server. The DNS server could be tested independently from the local network.
To test this, I used the dig command:
```
dig google.com @127.0.0.1
```
- ```dig```(Domain Information Groper) is a DNS query tool that can send DNS requests directly to a DNS server and display the response. The command is provided by the ```dnsutils``` package on Debian-based systems
- ```127.0.0.1``` is a special IPv4 address known as localhost. It aiways refers to the current machine. On Linux systems, this address belongs to the loopback interface ```lo```

Traffic sent to ```127.0.0.1``` never leaves the computer. Instead, it is routed back to the local system.
In this test, the DNS query was sent to the DNS service running on the Raspberry Pi itself.

The command asks the DNS server for the IP address of google.com.
The DNS server listens for the request on:
```
127.0.0.1:53
```
The response returned valid IPv4 addresses and the status ```NOERROR```, which confirmed that Pi-hole was functioning correctly as a DNS server.










