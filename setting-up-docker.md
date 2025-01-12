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

### 2. Install Docker Engine: Run the following commands to install Docker Engine:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 3. Add Your User to the Docker Group: Avoid needing sudo for Docker commands:
```bash
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
```

## Next Steps

Go to [Setting Up PolyEdge](setting-up-polyedge.md).

## Previous Steps

Return to [Getting Started](getting-started.md).