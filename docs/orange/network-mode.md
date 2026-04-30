# Network mode

For multi-host capture, the GUI host runs `orange` and each capture host runs `release/cam_server <hostname>` (or `start_server.sh`, which auto-detects the host's short hostname; override with `CAM_SERVER_NAME`). Communication is over ENet using a FlatBuffers control protocol (schema in [`schema/ctrl.fbs`](https://github.com/moments-behavior/orange/blob/main/schema/ctrl.fbs)).

## Important caveats

- **All `cam_server` processes must be running before the GUI is launched.** The GUI connects to its configured endpoints at startup, not on demand.
- The config preset folder (e.g. `config/network/8cam_rig/`) must exist on **both the GUI host and every capture host**. The contents can be identical across all hosts — each `cam_server` will only open the cameras that are physically attached to *its* host and silently ignore the rest. Copying the same complete preset folder to every machine is the simplest setup.

## Setup walkthrough

### 1. Create `endpoints.json` on the GUI host

`orange_data/config/network/endpoints.json` lists the capture hosts. A template is shipped at [`config/network/endpoints.json.example`](https://github.com/moments-behavior/orange/blob/main/config/network/endpoints.json.example):

```json
{
  "default_port": 34001,
  "servers": [
    { "name": "waffle-0", "host": "192.168.20.60" },
    { "name": "waffle-1", "host": "192.168.20.61" }
  ]
}
```

- `host` accepts a hostname or an IP.
- `port` can be set per server; otherwise `default_port` is used.
- `name` is the logical id (matches the argv `cam_server` is started with).

The file is only needed for network mode. If it's absent, `orange` starts up normally and network features are unavailable; if it's present but malformed, `orange` prints a warning and continues.

### 2. Create matching preset folders

On every host (GUI host + capture hosts), create a matching `orange_data/config/network/<preset>/` folder containing one JSON per camera in the rig (named after the camera serial). The folder can be identical across hosts.

### 3. Start `cam_server` on each capture host

```bash
sudo release/cam_server <hostname>      # or: ./start_server.sh
```

### 4. Launch `orange` on the GUI host

```bash
./run.sh
```

It reads `endpoints.json`, connects to each server, pushes the chosen preset name, and each server opens its locally-attached cameras from its copy of the preset.

## PTP

Multi-host capture requires PTP for sub-frame synchronization. See [PTP setup](ptp.md).

## Recordings

Recordings are saved under `orange_data/exp/unsorted/<timestamp>/` by default; the save location can be changed from within the app or via [user config](configuration.md#user-config-optional).
