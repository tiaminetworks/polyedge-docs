---
title: "Setting Up PolyEdge"
nav_order: 4
layout: default
parent: "PolyEdge Documentation"
---

# Setting Up PolyEdge

After installing Docker, follow these steps to configure PolyEdge:

---

## Step 1: Prepare Docker Containers

Run the following commands to start the necessary Docker containers:
```bash
docker compose up
```
### Step 2: Custom Arguments for WiFi and NR
## WiFi
```bash
./stream_wifi
./recv_wifi
./visualize_wifi
```
## NR
```bash
./stream_NR
./nr-uesoftmodem
./visualize_NR
```

Next Steps

Continue to [Occupancy Tracking](occupancy-tracking.md).