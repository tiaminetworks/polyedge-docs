# PolyEdge Documentation

## Occupancy Tracking - Getting Started Tutorial

### Using an Edge Device Offered by Tiami Networks

#### Switching Device On
Power the device. Ensure the device voltage is within the 100-240V AC range for global compatibility.

- **For Tiami Networks Devices**: Follow the voltage range mentioned above.
- **For PolyEdge on User Devices**: Verify power compliance with **B2x0 devices from Ettus Research**.

---

### Configuring the Device

#### Enter Device Encryption Password
Use the provided encryption password to unlock the SDD.

#### Connect to WiFi or Ethernet
- Use the Ethernet port to connect to the AWS endpoint.
- **Important**: If utilizing WiFi sensing paradigms, WiFi ports will be disabled for data transfer. Ethernet is required to access AWS endpoint benefits.

---

## Running PolyEdge

PolyEdge utilizes Docker services for deployment. For edge-specific devices, the necessary services should already be pre-installed.

### Updating the System
Run the following commands to ensure your system is updated:
```bash
sudo apt update && sudo apt upgrade
```
### 1. Installing Docker

## Supported Ubuntu Versions

To install Docker Engine, you need the 64-bit version of one of the following Ubuntu distributions:

- Ubuntu Oracular 24.10
- Ubuntu Noble 24.04 (LTS)
- Ubuntu Jammy 22.04 (LTS)
- Ubuntu Focal 20.04 (LTS)

---

## Step-by-Step Installation

### 1. Remove Conflicting Packages
Run the following command to remove any conflicting packages:
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do 
    sudo apt-get remove $pkg; 
done
```
### 2. Add Docker's Official GPG Key
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
### 3. Add the Docker Repository to Apt Sources

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
### 4. Install Docker

Install Docker Engine and related components:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

### 5. Install Buildx

#### 1. Download the latest Buildx release from Docker Buildx Releases.
#### 2. Move the Buildx distribution:

```bash
mkdir -p ~/.docker/cli-plugins/
mv <downloaded-buildx-file> ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
```
### 6. Add Your User to the Docker Group

To avoid needing sudo for Docker commands, run:

```bash
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
```
## Starting Docker Containers
Default Commands

Update and launch the default Docker configuration:

docker compose up   

Here's the rewritten content in proper GitHub Markdown format:

# Installing Docker

## Supported Ubuntu Versions

To install Docker Engine, you need the 64-bit version of one of the following Ubuntu distributions:

- Ubuntu Oracular 24.10
- Ubuntu Noble 24.04 (LTS)
- Ubuntu Jammy 22.04 (LTS)
- Ubuntu Focal 20.04 (LTS)

---

## Step-by-Step Installation

### 1. Remove Conflicting Packages
Run the following command to remove any conflicting packages:
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do 
    sudo apt-get remove $pkg; 
done
```
2. Add Docker's Official GPG Key
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
3. Add the Docker Repository to Apt Sources
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
4. Install Docker

Install Docker Engine and related components:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
5. Install Buildx

    Download the latest Buildx release from Docker Buildx Releases.
    Move the Buildx distribution:
```bash
    mkdir -p ~/.docker/cli-plugins/
    mv <downloaded-buildx-file> ~/.docker/cli-plugins/docker-buildx
    chmod +x ~/.docker/cli-plugins/docker-buildx
```
6. Add Your User to the Docker Group

To avoid needing sudo for Docker commands, run:
```bash
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
```
Starting Docker Containers
Default Commands

Update and launch the default Docker configuration:

docker compose up

Custom Arguments

Modify docker-compose.yml or use utility scripts:
WiFi:
```bash
./stream_wifi
./recv_wifi
./visualize_wifi
```
NR:
```bash
./stream_NR
./nr-uesoftmodem
./visualize_NR
```

