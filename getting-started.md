---
title: "Getting Started"
nav_order: 2
layout: default
parent: "PolyEdge Documentation"
---

# Getting Started

PolyEdge is either provided directly on a standalone device, or can be services on compatible devices with x86 architecture. This guide provides setting up PolyEdge for both cases. 

## PolyEdge standalone

To setup PolyEdge on standalone device:

### Switching Device On

1. Ensure the device voltage is within 100-240V AC for global compatibility.
2. Enter the provided encryption password to unlock the SDD, please reach out to support for any issues with encryption issues.
3. Login to the client space, 
3. Connect via WiFi or Ethernet, requried to send IoT messages to AWS Endpoint.

### Applying updates and upgrades:

To fetch polyedge, users must have a assigned public key from their device, or polyedge standalone granting permissions to clone the repository. New users utilizing PolyEdge standalone can skip this step.

```bash
mkdir $HOME/git
cd $HOME/git
sudo apt update && sudo apt upgrade
git clone https://github.com/tiaminetworks/polyedge && cd polyedge
git checkout v25.1
```

---

## Next Steps

Continue to [Setting Up Docker](setting-up-docker.md).
