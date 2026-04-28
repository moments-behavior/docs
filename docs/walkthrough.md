# End-to-end walkthrough

This page walks one example rig from a blank machine to a trained 3D pose model. The goal is to show how `orange` and `red` fit together.

The example rig is **17 Emergent cameras spread across 1 GUI host and 2 capture servers**, recording at **180 fps**, labeling rat poses in `red`, and training a **JARVIS-HybridNet** 3D pose model on the result.

---

## 1. Prerequisites

Before starting, every host needs to meet [orange's system requirements](orange/installation/index.md): NVIDIA GPU with NVENC, kernel 6.5, Ubuntu 22.04.4. Cameras connected to the Emergent NIC and reachable from `eCapture`.

You also need both apps built (on the GUI host at minimum):

- [Build orange](orange/installation/build.md)
- [Build red](red/installation/build.md)

Each capture server needs `cam_server` built (see `server_build.sh` in the orange repo).

PTP set up across all hosts and switches:

- [PTP setup](orange/ptp.md)

---

## 2. Configure cameras and record a session in orange

This rig is multi-host, so it runs `orange` in **network mode**. Full details: [Network mode](orange/network-mode.md).

### 2a. Create a config preset on every host

On the GUI host **and** each capture server, create the matching preset folder:

```
~/orange_data/config/network/arena/
├── <serial-1>.json
├── <serial-2>.json
├── ...
└── <serial-17>.json
```

One JSON per camera, named after its serial. Start from the [example](https://github.com/moments-behavior/orange/blob/main/config/2002496.json.example) and adjust per-camera settings (resolution, exposure, `gpu_id`, etc.).

> *TODO: paste a representative camera JSON for the arena rig (180 fps settings, etc.).*

The contents can be identical across all hosts — each `cam_server` will only open the cameras physically attached to its host and silently ignore the rest.

### 2b. Create `endpoints.json` on the GUI host

`~/orange_data/config/network/endpoints.json` lists the two capture servers:

```json
{
  "default_port": 34001,
  "servers": [
    { "name": "waffle-0", "host": "192.168.20.60" },
    { "name": "waffle-1", "host": "192.168.20.61" }
  ]
}
```

> *TODO: replace with the actual hostnames / IPs of the arena servers.*

### 2c. Start `cam_server` on each capture host

```bash
sudo release/cam_server <hostname>      # or: ./start_server.sh
```

Both servers must be running **before** the GUI launches.

### 2d. Launch orange on the GUI host

```bash
cd ~/src/orange
./run.sh
```

In the GUI, pick the `arena` preset and open the cameras. The GUI connects to both servers, pushes the preset name, and each server opens its locally-attached cameras.

### 2e. Verify PTP and cameras

> *TODO: screenshot of orange GUI with all 17 cameras streaming + PTP offsets near zero.*

### 2f. Record

Set a save folder (defaults to `~/orange_data/exp/unsorted/<timestamp>/`), start recording, run the trial, stop.

> *TODO: confirm typical session length for the arena rig at 180 fps.*

You should now have:

```
orange_data/exp/unsorted/2026_04_28_HHMMSS/
├── Cam<serial-1>_meta.csv
├── Cam<serial-1>.mp4
├── Cam<serial-2>_meta.csv
├── Cam<serial-2>.mp4
├── ...
```

per host (each capture server writes its own cameras to its local disk).

---

## 3. Calibrate cameras (one-time)

Multi-view triangulation in `red` requires camera intrinsics + extrinsics. Run the calibration capture in `orange` once per rig.

> *TODO: brief checkerboard procedure — board size, how many poses, what to expect in `~/orange_data/calib_yaml/`.*

Calibration produces a YAML per camera plus a combined `calib.yaml` consumed by `red`.

---

## 4. Open the recording in red

Bring all 17 cameras' recordings together (e.g. via NFS or by copying off each capture server) so `red` can open them as one project.

```bash
cd ~/src/red
./run.sh
```

In the GUI, **File → Open project** → point at the recording folder.

> *TODO: screenshot of red opening a 17-camera recording.*

---

## 5. Label 3D rat poses

### 5a. Author the skeleton

`red` uses a `skeleton.json` to define the keypoints and edges. The repo ships an example at [`example/skeleton.json`](https://github.com/moments-behavior/red/blob/main/example/skeleton.json) — adapt it for the rat keypoints you want to track.

> *TODO: brief description of the skeleton authoring UI in red, or instructions to edit the JSON directly. Mention typical rat keypoint set used in the arena rig.*

### 5b. Label keypoints across views

Pick frames spaced across the recording. For each frame, click each keypoint in **at least 2 camera views**. `red` triangulates the 3D position automatically and projects it back into all 17 views.

With 17 cameras you don't need to label more than 2–3 views per keypoint — pick the views where the rat is least occluded for that frame.

> *TODO: target frame count for an initial JARVIS dataset (~50? ~100?).*
>
> *TODO: screenshot of red labeling a keypoint across the 17 views with the triangulated 3D point overlay.*

Save the project (`File → Save`).

### 5c. Verify labels

> *TODO: how to scrub through the labeled frames and visually verify triangulation looks right across all 17 views.*

---

## 6. Export for JARVIS

Use the JARVIS exporter from `red`'s `data_exporter/`. Full reference: [Data export](red/data-export.md).

```bash
cd ~/src/red/data_exporter
conda activate red_exporter
python red3d2jarvis.py \
    -w ~/orange_data \
    -o ~/datasets/arena_jarvis \
    -m 60
```

`-m` is the bounding-box margin (in calibration units, typically mm). Adjust for the rat size.

---

## 7. Train a JARVIS HybridNet model

Follow the [JARVIS-HybridNet](https://github.com/JARVIS-MoCap/JARVIS-HybridNet) training docs.

> *TODO: any project-specific training tips — typical epochs, GPU memory, batch size for 17-camera setup.*

---

## 8. Run JARVIS inference and view results back in red

Run JARVIS-HybridNet inference on the recorded videos with its own inference scripts.

To visualize the predictions in `red`, convert them back with `jarvis2red3d.py`:

```bash
python jarvis2red3d.py \
    -i /path/to/predictions_3D_folder/ \
    -s arena \
    -o /path/to/output_folder
```

Then open the output folder in `red` and scrub through the predicted poses across all 17 views — useful for spot-checking and finding error frames to relabel.

A confidence filter is on by default (drops predictions below 0.7 confidence and z > 500 mm). Disable with `--filter=0`.

---

## What's next

- Adding the **YOLO real-time detection** path so an animal-bbox model runs live during capture in `orange`. See [Real-time detection](orange/realtime-detection.md).
- Different downstream pipelines: see the YOLO pose option in [Data export](red/data-export.md).
