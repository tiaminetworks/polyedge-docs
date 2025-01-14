---
title: "Setting Up Docker"
nav_order: 3
layout: default
parent: "PolyEdge Documentation"
---

# Setting Up Docker

PolyEdge uses Docker services for deployment. Follow these steps to install and configure Docker.

---

## Step 1: Update the System

Run the following commands to update your system:
```bash
sudo apt update && sudo apt upgrade
```
## Step 2: Install Docker

### 1. Supported Ubuntu Versions:
Docker supports 64-bit versions of these distributions:
    Ubuntu Oracular 24.10
    Ubuntu Noble 24.04 (LTS)
    Ubuntu Jammy 22.04 (LTS)
    Ubuntu Focal 20.04 (LTS)

### 2. Add Docker's Official GPG Key

Run the following commands to add Docker’s official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 3. Add the Docker Repository to Apt Sources
Run the following commands to add the Docker repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```


### 4. Install Docker Engine: Run the following commands to install Docker Engine:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 5. Install Buildx
#### 1. Download the latest Buildx release from Docker Buildx Releases.
#### 2. Move the Buildx distribution with the following commands:
```bash
mkdir -p ~/.docker/cli-plugins/
mv <downloaded-buildx-file> ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
```

### 6. Add Your User to the Docker Group: Avoid needing sudo for Docker commands:
```bash
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
```

### 7. Docker CLI and usage:

PolyEdge leverages docker services to expose relevant entrypoints for both WiFi and NR. To service these ports such as for terminal interactivity in docker-cli the `service-ports` arg is utilized.
```bash
docker compose run --service-ports polyedge
```
Where `polyedge` is the container name servicing PolyEdge, by default this invokes the entrypoint for WiFi.

## Next Steps

Go to [Setting Up PolyEdge](setting-up-polyedge.md).

## Previous Steps

Return to [Getting Started](getting-started.md).