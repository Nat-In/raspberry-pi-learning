I only need Docker CLI because I work with Raspberry Pi via SSH.
Therefore, I decided to install Docker Engine using the APT repository from the official documentation:
https://docs.docker.com/engine/install/raspberry-pi-os/#install-using-the-repository

# Docker APT repository installation
Setting up the official Docker APT repository ensures that I install the latest stable version with security updates instead of outdated version from the default repositories.
It also allows automatic updates and guarantees that verified and compatible packages are installed.

## 1. Add Docker’s official GPG key
First, I update the system and install required tools:
  ```
  sudo apt update
  sudo apt install ca-certificates curl gnupg
```
- ```ca-certificates``` - allows secure HTTPS connections
- ```curl``` - downloads files from the internet
- ```gnupg``` - manages cryptographic keys

Create a directory for storing security keys:
  ```
  sudo install -m 0755 -d /etc/apt/keyrings
```
This creates a folder and sets permissions so:
- everyone can read and execute
- only root can modify

Download Docker’s GPG key:
```
  sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
```
This key is used to verify that packages come from Docker and are not tampered with.

Set correct permissions:
```
  sudo chmod a+r /etc/apt/keyrings/docker.asc
```
This allows the system (APT) to read the key during installation.

## 2. Add the repository to APT sources
This step tells the system where to download Docker packages from.
```
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Explanation:
- ```dpkg --print-architecture``` - detects CPU architecture (e.g. arm64)
- ```$VERSION_CODENAME``` - detects OS version (e.g. trixie)
- ```signed-by``` - tells APT which key to use
- ```tee``` - writes this configuration into a system file

Update package list:
```
  sudo apt update
```
The system now reads the new repository and downloads available package versions.

### Error
I encountered an error:
```
  https://download.docker.com/linux/raspbian trixie Release → 404 Not Found
```
Raspberry Pi OS is based on Debian, but Docker expects the repository path to use ```debian```, not ```raspbian```.

### Solution
Fix the repository configuration:
```
  sudo sed -i 's/raspbian/debian/g' /etc/apt/sources.list.d/docker.list
  sudo apt update
```
This command replaces ```raspbian``` with ```debian``` in the configuration file.
Alternatively, I could have used ```debian``` from the beginning.

## 3. Install Docker packages
```
  sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Packages:
- ```docker-ce``` - Docker Engine
- ```docker-ce-cli``` - command-line interface
- ```containerd.io``` - container runtime
- ```docker-buildx-plugin``` - advanced image building
- ```docker-compose-plugin``` - tool for running multi-container applications
Docker Compose uses a YAML file ```(docker-compose.yml)``` to define services, networks, and volumes.

## 4. Post-install step
Add current user to Docker group:
```
  sudo usermod -aG docker $USER
```
This allows running Docker commands without ```sudo```.
Reconnect to apply changes.

## 5. Check Docker
```
  docker --version
  docker ps
```

# Result
Docker is installed, running, and ready to use.
