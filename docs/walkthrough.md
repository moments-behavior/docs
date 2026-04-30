# Tutorial

This page walks the post-recording side of the pipeline — calibration, labeling, training, and inference — using a 10-second sample from our 17-camera arena rig, so you can practice with real data without setting up your own cameras first. The output is a JARVIS-HybridNet 3D pose model for a freely-moving rat.

---

## 1. Prerequisites

- [`red`](red/installation/build.md) built and runnable.
- A Python environment for the JARVIS exporter and trainer (see [JARVIS-HybridNet](https://github.com/JARVIS-MoCap/JARVIS-HybridNet) and [Data export](red/data-export.md) for setup).
- The sample dataset downloaded (next step).

---

## 2. Get the sample data

> *TODO: download URL for the 10-second 17-camera arena sample.*

The sample bundle contains:

- 17 short `.mp4` recordings (one per camera) plus per-camera meta CSVs
- Camera calibration files (`calibration/`) — intrinsics + extrinsics
- A starter `skeleton.json` for the rat keypoint set used in this walkthrough

Layout after unpacking:

```
red_labeling_example/
├── calibration/
│   ├── <serial-1>.yaml
│   ├── ...
├── movies/
│   ├── Cam<serial-1>.mp4
│   └── ...
```

---

## 3. About the calibration

The sample ships with calibration already done — `calibration/` contains intrinsics and extrinsics for all 17 cameras. We use ChArUco board to calibrate the rig. Please refer to this [repo](https://github.com/moments-behavior/multiview_calib) for details. 

---

## 4. Open the recording in red

```bash
cd ~/src/red
./run.sh
```

In the GUI, **File → Open Video(s)** → navigate to the `red_labeling_example/movies` folder, select all cameras, and click **OK**.

![load videos dialog](assets/tutorial_labeling/load_videos.png)

Once loaded, you can watch the synchronized recorded videos in `red`. Note: it might take some time to initialize multiple large videos.

To start labeling, in the GUI choose **File → Create Project**. In this dialog, name the labeling project, select a save path, and pick the **Rat24** skeleton preset. (`red` also has a skeleton creator under **Tools → Skeleton Creator** that you can use to define your own `skeleton.json`; load it by switching the dropdown from **Preset** to **File**.)

Click **Browse** to navigate to the calibration folder and select it. For instance:

![create project dialog](assets/tutorial_labeling/create_project.png)

Click **Create Project**. It creates a folder with the Project Name inside Full Path — in this example, `/mnt/data/red_labeling_example/demo/` — with the following layout:

```
red_labeling_example/demo/
├── labeled_data
└── project.redproj
```

`project.redproj` is a JSON file containing the project's metadata (calibration folder, skeleton, etc.). Double-clicking it opens the project and reloads the videos and most recently labeled frames — provided `red` was installed via the [desktop launcher](red/installation/build.md#optional-install-a-desktop-launcher).

---

## 5. Label 3D rat poses

Pick frames spaced across the 10-second clip. For each frame, click each keypoint in **at least 2 camera views**. `red` triangulates the 3D position automatically and projects it back into all 17 views.

With 17 cameras you don't need to label more than 2–3 views per keypoint — pick the views where the rat is least occluded for that frame.

For a detailed walkthrough of labeling 24 keypoints on a rat, watch the video demo:

<div style="position: relative; padding-bottom: 56.25%; height: 0;">
  <iframe src="https://www.youtube.com/embed/9eOJaadE1Nc"
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
    frameborder="0" allowfullscreen></iframe>
</div>


---

## 6. Export for JARVIS

Use the JARVIS exporter from `red`'s `data_exporter/`. Full reference: [Data export](red/data-export.md).

```bash
cd ~/src/red/data_exporter
conda activate red_exporter
python red3d2jarvis.py \
    -w ~/red_labeling_example/demo \
    -o ~/datasets/red_labeling_example_jarvis \
    -m 60
```

`-m` is the bounding-box margin (in calibration units, typically mm). Adjust for the rat size.

---

## 7. Train a JARVIS HybridNet model

Follow the [JARVIS-HybridNet](https://github.com/JARVIS-MoCap/JARVIS-HybridNet) training docs.

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

- **Iterating:** relabel error frames found in step 8, retrain, repeat.
- **Recording your own data:** see the [orange docs](orange/index.md) for setting up cameras, PTP, and multi-host capture.
- **Different downstream pipelines:** see the YOLO pose option in [Data export](red/data-export.md).
- **Adding YOLO real-time detection in orange:** see [Real-time detection](orange/realtime-detection.md).
