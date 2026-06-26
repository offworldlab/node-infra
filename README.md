# Owl OS

Mender-enabled OS images for Raspberry Pi 5 radar nodes. The image comes pre-loaded with Docker, SDRplay API, Avahi mDNS, and a WiFi captive portal for easy setup.

##  Contents

- **Docker CE** with Compose plugin
- **SDRplay API v3.15** for RSPDuo hardware
- **SDRconnect v1.0.5** for standalone SDR analysis
- **Chrony** for NTP clock disciplining
- **Cloudflared** for secure tunneling
- **Avahi mDNS** for `<hostname>.local` discovery
- **WiFi Connect** captive portal for network setup
- **Mender client** for OTA updates

## Quick Start

1. **Flash the image** to an SD card (64GB+) using Raspberry Pi Imager
   - Download from [Releases](https://github.com/offworldlabs/owl-os/releases)
   - Select `owl-os-vx.x.x.img` as custom OS
   - Do not apply OS customisation settings

2. **Boot and connect to WiFi**
   - Connect to the `node-setup` WiFi network
   - Captive portal opens automatically (or go to http://192.168.42.1)
   - Enter your WiFi credentials - node reboots and connects

3. **Accept device in Mender**
   - Node appears as "pending" once online
   - Accept to enable OTA updates

4. **Deploy [retina-node](https://github.com/offworldlabs/retina-node) stack** via Mender OTA

## Mender Cloud Services

Cloud services (mender-authd, mender-updated, mender-connect) are handled differently between the two build artifacts:

- **`.img` (fresh flash)** — cloud services are **disabled** by default. `mender.conf` is backed up to `/data/mender-cloud-disabled/` and the three Mender systemd services are masked. The user must consent via the retina-gui install flow to enable them.
- **`.mender` (OTA artifact)** — cloud services are **re-enabled** via `debugfs` in the `image2mender` post-processing step. This is required because `mender-updated` must start after reboot to run `ArtifactVerifyReboot` and commit the update — without it the update hangs and rolls back.

After install, users can toggle cloud services on/off from `http://owl.local`. The preference persists across reboots and OTA updates. Toggling is blocked while any update is in progress.

### Install Lock

During a retina-node install, retina-gui holds an `install.lock` in `/data/retina-gui/` to prevent concurrent installs and block cloud service toggling mid-update. The lock is released on completion (success or failure) with a 40-minute stale timeout as a safety net. Mender state scripts write a separate `mender-update.status` file for real-time progress polling (downloading → installing → done).

## Configuration

### Node Configuration

After deploying retina-node, visit `http://owl.local` to configure capture settings, location, ADS-B truth source, and tar1090. See [retina-node](https://github.com/offworldlabs/retina-node) for details.

### Cloudflare Tunnel (Optional)

To enable Cloudflare tunnel forwarding, create a token file on the node:

```bash
sudo mkdir -p /data/cloudflared
echo "YOUR_TUNNEL_TOKEN" | sudo tee /data/cloudflared/tunnel-token
sudo chmod 600 /data/cloudflared/tunnel-token
sudo systemctl restart cloudflared
```

The token persists across OTA updates.

### SSH Access

**End users:** Add your SSH key via the web GUI at `http://owl.local` after boot. Once added, connect with:
```bash
ssh node@owl.local
# or by IP
ssh node@<ip-address>
```

Keys persist across reboots and OTA updates.

**Developers:** Public keys can be baked into the image at build time by adding them to `ssh_pub_keys/`:
```bash
cp ~/.ssh/id_ed25519.pub ssh_pub_keys/yourname.pub
```

## Creating a Release

Tag a commit with `os-vx.x.x` and push:
```bash
git tag os-v1.0.0
git push origin os-v1.0.0
```

This triggers the GHA workflow (`.github/workflows/build_os.yml`) which:
1. Builds OS image and Mender artifact
2. Uploads to GitHub Releases
3. Uploads Mender artifact to OffWorld Lab Mender server

> **Note:** Currently triggers on any `os-v*` tag. TODO: Change to only PR merges into main.

## SDRconnect

Run in server mode (headless device):
```bash
/opt/sdrconnect/SDRconnect --server
```

Connect from a SDRconnect client on another machine using the Pi's IP.

> **Warning:** Conflicts with blah2 - stop containers first:
> ```bash
> cd /data/mender-docker-compose/current/manifests && docker compose -p retina-node down
> ```

### Mender Tenant Token

**For GitHub Actions:** tenant token is added via GH secrets.

**For local builds:** Create a custom config file:
```bash
echo 'mender_tenant_token: "YOUR_TOKEN_HERE"' > configuration/mender/mender_custom.yml
```

This file is `.gitignore`d to prevent accidental commits.

## Building from Source

### Requirements

Install [EDI](https://docs.get-edi.io/en/stable/getting_started_v2.html) and dependencies:

**Ubuntu 24.04 or newer:**
```bash
sudo apt install buildah containers-storage crun curl distrobox \
  dosfstools e2fsprogs fakeroot genimage git mender-artifact \
  mmdebstrap mtools parted python3-sphinx python3-testinfra \
  podman rsync zerofree
```

### Build
```bash
edi -v project make owl-os-pi5.yml
```

**Output artifacts:**
- `owl-os-vx.x.x.img` - Flashable OS image with A/B partitioning
- `owl-os-vx.x.x.mender` - OTA update artifact

### Clean
```bash
edi -v project clean owl-os-pi5.yml
```

## Default Credentials

- **Username:** `node`
- **Password:** `raspberry`

## Credits

Built using [EDI-PI](https://github.com/lueschem/edi-pi) by [lueschem](https://github.com/lueschem).
