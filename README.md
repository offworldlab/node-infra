# Radar Node infrastructure

Radar Node infrastructure for building Mender-enabled OS images and artifacts pre-loaded with system requirements (Docker, SDRplay API, Graphviz) for blah2 radar deployment. Currently in development.

# Requirements

Install [edi](https://github.com/lueschem/edi) according to these [instructions](https://docs.get-edi.io/en/stable/getting_started_v2.html).

On Ubuntu 24.04 or newer install the following tools:

```
sudo apt install buildah containers-storage crun curl distrobox dosfstools e2fsprogs fakeroot genimage git mender-artifact mmdebstrap mtools parted python3-sphinx python3-testinfra podman rsync zerofree
```

# Usage

## SSH

Public SSH keys are stored in the `ssh_pub_keys` folder. To add a key, copy your public key file:
```bash
cp ~/.ssh/id_ed25519.pub ssh_pub_keys/sol.pub
```

## Mender Tenant Token

Tenant tokens are currently untracked and will be added via GitHub secrets. To add a tenant token on your local machine:
```bash
echo 'mender_tenant_token: "TOKEN_HERE"' > configuration/mender/mender_custom.yml
```

## Build

To build the project, run:
```bash
edi -v project make pi5-radar.yml
```

This will output a series of artifacts, including:
- An mender-enabled `.img` file with A/B partitioning loaded with Docker, SDRplay API & Graphviz that can be flashed to an SD card
- A corresponding `.mender` artifact for OTA OS updates

## Running blah2

> **Note:** This process is temporary and will change soon.

**Default Credentials:**
- Username: `pi`
- Password: `raspberry`

### Installation Steps

1. Flash the image to an SD card and insert it into the Raspberry Pi
2. SSH into the Pi and run the following commands:
```bash
sudo rm -rf /opt/blah2 
sudo git clone https://github.com/offworldlab/blah2-arm /opt/blah2
cd /opt/blah2
sudo chown -R $USER .
sudo docker network create blah2
sudo docker compose up -d --build
```
## Clean

To clean the project:
```bash
edi -v project clean pi5-radar.yml
```

## Credits
Built using [EDI-PI](https://github.com/lueschem/edi-pi) by [lueschem](https://github.com/lueschem)