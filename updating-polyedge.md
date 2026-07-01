---
title: "Updating PolyEdge"
nav_order: 4
layout: default
parent: "PolyEdge Documentation"
---

# Updating PolyEdge

PolyEdge deployments update themselves by fetching a version manifest and reinstalling the four packages — no manual `.deb` download required once configured.

## How it works

1. The GUI's orchestrator exposes `POST /api/system/update`.
2. That endpoint runs `/opt/polyedge/bin/polyedge-update.sh`, which:
   - Fetches a JSON manifest from a configured URL
   - Downloads each `.deb` package listed in the manifest and verifies its SHA-256 checksum (if provided)
   - Installs the packages via `apt`, in dependency order (`polyedge-runtime` → `polyedge-streamer` → `polyedge-interact`, plus any others listed)
   - Updates Python dependencies in the streamer's virtual environment
   - Restarts the `polyedge-interact` service

The manifest format:

```json
{
  "version": "1.2.3",
  "packages": [
    {"name": "polyedge-runtime", "url": "https://.../polyedge-runtime_1.2.3_amd64.deb", "sha256": "..."},
    {"name": "polyedge-streamer", "url": "https://.../polyedge-streamer_1.2.3_amd64.deb", "sha256": "..."},
    {"name": "polyedge-interact", "url": "https://.../polyedge-interact_1.2.3_amd64.deb", "sha256": "..."}
  ]
}
```

## Enabling it

Self-update is **disabled by default**. To enable it, set both of these in `/etc/polyedge-interact/.env`:

```bash
POLYEDGE_ALLOW_UPDATE=true
POLYEDGE_UPDATE_MANIFEST_URL=https://your-hosted-manifest-url/manifest.json
```

Then restart the service: `sudo systemctl restart polyedge-interact`.

## Triggering an update

```bash
curl -X POST http://<host>:3000/api/system/update
```

This restarts the GUI service as part of the update, so any active browser session or API client connection will drop momentarily — expected, not an error. Progress is logged to `/var/log/polyedge-update.log` and streamed over the GUI's Socket.IO connection (`log_update` events) while it runs.

## Notes

- `polyedge-radio` isn't part of the default update rotation (it has an independent build chain) — update it separately if a new radio-stack version is needed.
- Without `POLYEDGE_ALLOW_UPDATE=true`, `POST /api/system/update` returns `403`.
