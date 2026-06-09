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
```
- timezone
- web interface password
- DNS listening behavior
```

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



