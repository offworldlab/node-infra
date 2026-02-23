# Raspberry Pi OS Integration

**Branch:** `feature/raspios-optimizations`

## Problem

High CPI times with live SDR antennas. Root cause: USB buffer size 16MB (Debian default) vs 1000MB (RPi OS). At 2MHz sample rate, this causes buffer overflows and CPU overhead.

## Solution

Add Raspberry Pi repository to pull `raspberrypi-sys-mods` package which provides USB tuning, sysctl optimizations, and hardware-specific configurations.

## Changes

### Files Modified
- `owl-os-pi5.yml` - line 33: `debian.list` → `raspios.list`
- `plugins/playbooks/os_setup/main.yml` - Added `raspios_packages` role

### Files Created
- `plugins/playbooks/os_setup/collections/ansible_collections/get_edi_io/debian_setup/roles/apt_setup/templates/raspios.list`
- `plugins/playbooks/os_setup/roles/raspios_packages/tasks/main.yml`

## Build

```bash
edi -v project make owl-os-pi5.yml
```

No changes to build process.

## Verification

```bash
# USB buffer (should be 1000)
cat /sys/module/usbcore/parameters/usbfs_memory_mb

# Package installed
dpkg -l | grep raspberrypi-sys-mods
```

## Testing

1. Build image
2. Flash to test node
3. Deploy blah2 via Mender
4. Measure CPI time with live antennas

Expected: CPI time matches old Raspbian performance.
