---
title: "Installation"
nav_order: 2
layout: default
parent: "PolyEdge Documentation"
---

# Installation

PolyEdge ships as four versioned Debian packages. Install them in this order — later packages depend on earlier ones:

| Package | Installs | Depends on |
|---|---|---|
| `polyedge-runtime` | Config contract (`/etc/polyedge`), state (`/var/lib/polyedge`), license placeholder, profile selector | — |
| `polyedge-radio` | 5G NR UE stack (`nr-uesoftmodem`), `tiami-ringd`, radio libraries — to `/opt/polyedge/RADIO/` | Independent build chain |
| `polyedge-streamer` | The passive radar streamer binary and helpers, to `/opt/polyedge/STREAMER/` | `polyedge-runtime` (same major version) |
| `polyedge-interact` | The GUI web app (Node.js server + React client), port 3000 | `polyedge-runtime` |

```bash
sudo apt install -y ./polyedge-runtime_<version>_amd64.deb
sudo apt install -y ./polyedge-radio_<version>_amd64.deb
sudo apt install -y ./polyedge-streamer_<version>_amd64.deb
sudo apt install -y ./polyedge-interact_<version>_amd64.deb

# Wire up symlinks and the systemd service for the GUI
sudo bash /opt/polyedge-interact/install-polyedge-interact-service.sh
```

Without `polyedge-radio` installed, `init_nrUE.sh` falls back to a legacy runtime path instead of the packaged radio stack — install it even if you plan to operate everything through the GUI, since the GUI orchestrates these same binaries underneath.

## Full build/packaging detail

Each package has its own README with build prerequisites and exact install layout — read these if you're building from source rather than installing a pre-built `.deb`:

- `deploy/polyedge_runtime_deb/README.md`
- `deploy/polyedge_radio_deb/README.md`
- `deploy/polyedge_streamer_deb/README.md` (needs the streamer binary built first via `passive_radar/py_spec/build.sh`)

## After installing

Verify the GUI is up:

```bash
polyedge-interact-status
```

This prints the accessible URLs (local + network) and confirms the systemd service is running. Continue to [Interacting with PolyEdge](interacting-with-polyedge.md).

## License required

PolyEdge won't start the detection pipeline without a valid, hardware-bound license issued by Tiami Networks. See [Licensing](licensing.md) to check status or request one before your first run.
