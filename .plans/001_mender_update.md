# 001 — Mender Client 6.0.0 Upgrade (Phase 1)

## Context
Upgrading owl-os to the Mender Client 6.0.0 suite. Phase 1 covers the client and
addon version pins only. Container update module migration is Phase 2.

Key facts:
- `mender-client4` meta-package name unchanged in 6.0
- Service names unchanged: `mender-authd` and `mender-updated`
- APT repo URL already correct (device-components) — comment update only
- `mender-connect` major version bump 2.x → 3.0.0 — was unpinned, now pinned
- `mender-configure` now pinned (was unpinned)
- `mender-monitor` 1.5.0 added — will fail cleanly at build if not in public pool

## Chronological Steps

### Step 1 — versions.yml
File: `plugins/playbooks/os_setup/versions.yml`

- Replace held packages comment (line 9):
  `# - mender-client4, mender-auth, mender-update, mender-connect`

- Replace Mender block (lines 18-19):
  ```yaml
  # Mender Client 6.0.0 suite
  mender_client_version: "5.1.0-1+debian+bookworm"
  mender_connect_version: "3.0.0-1+debian+bookworm"
  mender_configure_version: "1.1.4-1+debian+bookworm"
  mender_monitor_version: "1.5.0-1+debian+bookworm"
  ```

- Update app-update-module TODO comment (line 46) → note Phase 2

### Step 2 — tasks/main.yml (hard-linked, edit once)
File: `plugins/playbooks/os_setup/roles/mender/tasks/main.yml`

- Add version pins to "Install mender addons" task (lines 75-81):
  ```yaml
  - name: Install mender addons.
    apt:
      name:
      - "mender-connect={{ mender_connect_version }}"
      - "mender-configure={{ mender_configure_version }}"
      - "mender-monitor={{ mender_monitor_version }}"
      state: present
      install_recommends: no
  ```

- Fix held packages (lines 164-171):
  ```yaml
  - name: Hold Mender packages
    dpkg_selections:
      name: "{{ item }}"
      selection: hold
    loop:
      - mender-client4
      - mender-auth
      - mender-update
      - mender-connect
      - mender-monitor
  ```
  (Remove stale `mender-client` entry)

### Step 3 — mender-io.list comment
File: `plugins/playbooks/board_support/roles/mender/templates/mender-io.list`

- Line 1: `# Mender device-components repository for Mender Client 6.0.0 suite`

## Verification
1. EDI build: `edi -v image create owl-os-pi5.yml`
2. Check pins: `apt-cache policy mender-client4 mender-connect mender-configure mender-monitor`
3. Check holds: `dpkg --get-selections | grep mender`
4. Flash to Pi 5, enable cloud services, verify connect to hosted.mender.io
5. Smoke test mender-connect remote terminal (3.0.0)

## Notes
- If mender-monitor build fails (package not found) → remove from addons task and holds, it's gated
- mender-connect.conf: verify no breaking config changes in 3.0.0 before flashing

## Phase 2 (separate)
File: `plugins/playbooks/os_setup/roles/radar_packages/tasks/main.yml` lines 232-269
Replace app-update-module → `mender-container-modules` APT package.
Artifact type: `app` → `docker-compose`.
retina-node already bundles images — pipeline change is feasible.
