# End-to-end walkthrough

This page walks one example rig from a blank machine to a trained detection model running live in `orange`. The goal is to show how `orange` and `red` fit together — not to replace the per-project docs.

The example rig is a **single host with 4 Emergent cameras**, recording a 30-second session, labeling frames in `red`, and either (A) training a YOLO detection model and running it live in `orange`, or (B) training a JARVIS-HybridNet 3D pose model and running it offline.

> If you're running a multi-host rig instead, see [Network mode](orange/network-mode.md) for the differences — the rest of this walkthrough is unchanged.

---

## 1. Prerequisites

Before starting, you need a Linux host that meets [orange's system requirements](orange/installation/index.md): NVIDIA GPU with NVENC, kernel 6.5, Ubuntu 22.04.4. Cameras connected to the Emergent NIC and reachable from `eCapture`.

You also need both apps built:

- [Build orange](orange/installation/build.md)
- [Build red](red/installation/build.md)

PTP set up:

- [PTP setup](orange/ptp.md)

---

## 2. Configure cameras and record a session in orange

### 2a. Create a camera config preset

In `~/orange_data/config/local/`, create a folder for your rig (e.g. `4cam_demo/`). Drop one JSON per camera, named after its serial. Start from the [example](https://github.com/moments-behavior/orange/blob/main/config/2002496.json.example) and adjust per-camera settings.

> *TODO: paste your serials and a representative camera JSON here.*

### 2b. Launch orange

```bash
cd ~/src/orange
./run.sh
```

In the GUI, pick the `4cam_demo` preset and open the cameras.

### 2c. Verify PTP and cameras

> *TODO: screenshot of orange GUI with 4 cameras streaming + PTP offsets near zero.*

### 2d. Record

Set a save folder (defaults to `~/orange_data/exp/unsorted/<timestamp>/`), start recording, run the trial for ~30 seconds, stop.

You should now have:

```
orange_data/exp/unsorted/2026_04_28_HHMMSS/
├── Cam<serial1>_meta.csv
├── Cam<serial1>.mp4
├── Cam<serial2>_meta.csv
├── Cam<serial2>.mp4
├── ...
```

---

## 3. Calibrate cameras (one-time)

Multi-view triangulation in `red` requires camera intrinsics + extrinsics. Run the calibration capture in `orange` once per rig.

> *TODO: brief checkerboard procedure — what board, how many poses, what to expect in `~/orange_data/calib_yaml/`.*

Calibration produces a YAML per camera plus a combined `calib.yaml` consumed by `red`.

---

## 4. Open the recording in red

```bash
cd ~/src/red
./run.sh
```

In the GUI, **File → Open project** → point at the recording folder from step 2d.

> *TODO: screenshot of red opening a 4-camera recording.*

`red` supports two labeling flows. Pick one based on what you need downstream:

| Flow            | What you label                       | Used for                                          |
| --------------- | ------------------------------------ | ------------------------------------------------- |
| **A. 2D bboxes**   | A bounding box per object, per frame | YOLO object detection — runs **live in orange** as real-time detection. Simpler, faster to label. |
| **B. 3D keypoints** | Multi-view keypoints with a skeleton | 3D pose, used **offline** with JARVIS HybridNet. More involved labeling, richer output. |

Many projects do A first (cheap to set up, useful immediately) and add B later. The remainder of this walkthrough covers both paths in parallel; sections marked **A** apply only to bboxes, **B** only to 3D keypoints.

---

## 5A. Label bounding boxes (for YOLO detection)

Pick ~30 frames spaced across the recording. For each frame, draw a bounding box around your object of interest in **each camera view that sees it**. No skeleton or keypoint setup needed.

> *TODO: brief instructions for entering bbox-labeling mode in red + screenshot.*

Save the project (`File → Save`).

## 5B. Label 3D keypoints (for JARVIS)

### 5B-i. Author the skeleton

`red` uses a `skeleton.json` to define the keypoints and edges you'll label. The repo ships an example at [`example/skeleton.json`](https://github.com/moments-behavior/red/blob/main/example/skeleton.json).

> *TODO: brief description of the skeleton authoring UI in red, or instructions to edit the JSON directly.*

### 5B-ii. Label keypoints across views

Pick ~30 frames spaced across the recording. For each frame, click each keypoint in **at least 2 camera views**. `red` triangulates the 3D position automatically.

> *TODO: screenshot of red labeling a keypoint across 4 views with the triangulated 3D point overlay.*

Save the project (`File → Save`).

### 5B-iii. Verify labels

> *TODO: how to scrub through the labeled frames and visually verify triangulation looks right.*

---

## 6. Export labels for training

Use `red`'s exporter scripts. Full reference: [Data export](red/data-export.md).

### 6A. Export YOLO detection (bboxes)

```bash
cd ~/src/red/data_exporter
conda activate red_exporter
python export_yolo_detection.py \
    -i ~/orange_data/labeled \
    -v ~/orange_data/exp/unsorted/2026_04_28_HHMMSS \
    -o ~/datasets/4cam_demo_det \
    -c class_names.txt
```

Result: a YOLO detection dataset with train/val/test splits and `data.yaml`.

### 6B. Export 3D pose for JARVIS

```bash
python red3d2jarvis.py \
    -w ~/orange_data \
    -o ~/datasets/4cam_demo_jarvis \
    -m 60
```

---

## 7. Train

This step happens outside `orange` / `red`.

### 7A. Train YOLO detection

```bash
cd ~/src/YOLOv8-TensorRT
source .venv/bin/activate
yolo task=detect mode=train model=yolov8m.pt data=~/datasets/4cam_demo_det/data.yaml epochs=100 imgsz=640
```

Result: `runs/detect/train/weights/best.pt`.

### 7B. Train a JARVIS HybridNet model

Follow the [JARVIS-HybridNet](https://github.com/JARVIS-MoCap/JARVIS-HybridNet) training docs.

---

## 8A. Compile the engine and run real-time detection in orange

This section applies to **flow A (YOLO detection)**. Convert `.pt` → `.onnx` → `.engine`. Detail: [Real-time detection](orange/realtime-detection.md).

```bash
python3 export-det.py --weights best.pt --opset 11 --sim --device cuda:0
~/nvidia/TensorRT/bin/trtexec --onnx=best.onnx --saveEngine=best.engine --device=0 --fp16
```

!!! warning "Match TensorRT versions"
    The Python `tensorrt` wheel here must match the runtime version `orange` was built against. See [TensorRT installation](orange/installation/tensorrt.md).

Drop the engine where `orange` looks for it:

```bash
cp best.engine ~/orange_data/detect/best.engine
```

Set every camera's `gpu_id` in `~/orange_data/config/local/4cam_demo/*.json` to the GPU you compiled the engine for, then relaunch:

```bash
cd ~/src/orange
./run.sh
```

In the GUI, enable detection. You should see the YOLO predictions overlaid on the live camera streams.

> *TODO: screenshot of orange with live detection overlays.*

## 8B. Run JARVIS offline

Flow B is typically used **offline** rather than fed back into `orange` real-time. Run inference with JARVIS-HybridNet on the recorded videos using its own inference scripts. To visualize JARVIS predictions back in `red`, use `jarvis2red3d.py` — see [Data export → Load JARVIS predictions back into red](red/data-export.md#load-jarvis-predictions-back-into-red).

---

## What's next

- Recording longer sessions and adding more labeled frames to either dataset.
- Multi-host capture: see [Network mode](orange/network-mode.md).
- Combining flows: label bboxes for fast real-time triage in `orange`, then re-label a subset with 3D keypoints for offline analysis.
