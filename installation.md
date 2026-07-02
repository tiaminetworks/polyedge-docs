---
title: "Installation"
nav_order: 2
layout: default
parent: "PolyEdge Documentation"
---

# Installation

PolyEdge ships as four versioned Debian packages. Install them in this order, since later packages depend on earlier ones:

| Package | Installs | Depends on |
|---|---|---|
| `polyedge-runtime` | Config contract (`/etc/polyedge`), state (`/var/lib/polyedge`), license placeholder, profile selector | None |
| `polyedge-radio` | 5G NR UE stack (`nr-uesoftmodem`), `tiami-ringd`, radio libraries, to `/opt/polyedge/RADIO/` | Independent build chain |
| `polyedge-streamer` | The ISAC Engine binary and helpers, to `/opt/polyedge/STREAMER/` | `polyedge-runtime` (same major version) |
| `polyedge-interact` | The GUI web app (Node.js server + React client), port 3000 | `polyedge-runtime` |

```bash
sudo apt install -y ./polyedge-runtime_<version>_amd64.deb
sudo apt install -y ./polyedge-radio_<version>_amd64.deb
sudo apt install -y ./polyedge-streamer_<version>_amd64.deb
sudo apt install -y ./polyedge-interact_<version>_amd64.deb

# Wire up symlinks and the systemd service for the GUI
sudo bash /opt/polyedge-interact/install-polyedge-interact-service.sh
```

Install `polyedge-radio` even if you plan to operate everything through the GUI, since the GUI orchestrates these same binaries underneath. Without it, `init_nrUE.sh` falls back to a legacy runtime path instead of the packaged radio stack.

## After installing

Verify the GUI is up:

```bash
polyedge-interact-status
```

This prints the accessible URLs (local and network) and confirms the systemd service is running. Continue to [Interacting with PolyEdge](interacting-with-polyedge.md).

## License required

PolyEdge won't start the detection pipeline without a valid, hardware-bound license issued by Tiami Networks. See [Licensing](licensing.md) to check status or request one before your first run.
