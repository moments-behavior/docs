# Data export

Labeled keypoints in `red` are saved as plain CSV. The `data_exporter/` folder in the [red repo](https://github.com/moments-behavior/red/tree/main/data_exporter) provides Python scripts to convert them into the formats consumed by common training pipelines:

- **YOLO detection** — bounding-box dataset for YOLO object detection.
- **YOLO pose** — bounding box + keypoints, for YOLOv8 pose estimation.
- **JARVIS** — [JARVIS-MoCap/JARVIS-HybridNet](https://github.com/JARVIS-MoCap/JARVIS-HybridNet) format.

## Set up the Python environment

```bash
conda create -n red_exporter python=3.10
conda activate red_exporter
conda install numpy
conda install -c conda-forge opencv
pip install pyyaml
pip install PyNvVideoCodec
```

## YOLO detection

Export bounding-box data for YOLO object detection training:

```bash
python export_yolo_detection.py \
    -i /path/to/labeled/data \
    -v /path/to/videos \
    -o /path/to/output \
    -c class_names.txt
```

| Arg                | Description                                                                      |
| ------------------ | -------------------------------------------------------------------------------- |
| `-i, --label_dir`  | Directory containing timestamped label folders from `red`                        |
| `-v, --video_dir`  | Directory containing video files (filenames must match camera names)             |
| `-o, --output_dir` | Output directory for the YOLO dataset                                            |
| `-c, --class_file` | Optional text file with class names (one per line)                               |
| `--train_ratio`    | Fraction of data for training (default 0.7)                                      |
| `--val_ratio`      | Fraction for validation (default 0.2)                                            |
| `--test_ratio`     | Fraction for testing (default 0.1)                                               |
| `--seed`           | Random seed for reproducible splits (default 42)                                 |

Result: a YOLO-format dataset with train/val/test splits, a `data.yaml`, and normalized bbox coordinates.

## YOLO pose

Export bounding boxes + keypoints for YOLOv8 pose training:

```bash
python export_yolo_pose.py \
    -i /path/to/labeled/data \
    -v /path/to/videos \
    -o /path/to/output \
    -s skeleton.json
```

Same args as YOLO detection, plus:

- `-s, --skeleton_file` — JSON file defining skeleton structure (export from `red`'s skeleton creator).

Result: pose dataset with normalized bboxes + keypoint coords, a `data.yaml` for pose training, the skeleton JSON, and a README documenting the keypoint names.

## JARVIS

### Generate training data

```bash
python red3d2jarvis.py \
    -p project_path \
    -o output_folder \
    -m margin_for_bbox \
    [-s subset_of_keypoint_indices] \
    [-e new_skeleton_edges]
```

`project_path` is the `red` project folder — it should contain `labeled_data` and `project.redproj`.

Verify the dataset:

```bash
python check_jarvis_dataset.py -i jarvis_dataset [-s train/valid]
```

If your skeleton isn't in `keypoints.py`, add it manually for now.

### Load JARVIS predictions back into red

To visualize JARVIS predictions in `red`:

```bash
python jarvis2red3d.py \
    -i /path/to/predictions_3D_folder/ \
    -p /path/to/red_project
```

The converted predictions land in `<project>/predictions/`. Open the project in `red`, use **Load From Selected** to load the predictions, then scrub through the predicted poses across all views.

A confidence filter is applied by default (drops predictions below 0.7 confidence and z > 500 mm). Disable with `--filter=0`.

### Merge multiple JARVIS datasets (WIP)

Merge several JARVIS projects into one. Assumes all projects share the same camera set and image resolutions.

1. Convert each annotated `red` project into a JARVIS project with `red3d2jarvis.py`.
2. Collate the projects into one parent directory:

   ```
   jarvis_merge
   ├── dataset_11_06
   │   ├── annotations
   │   ├── calib_params
   │   ├── train
   │   └── val
   └── dataset_11_25
       ├── annotations
       ├── calib_params
       ├── train
       └── val
   ```

3. Merge:

   ```bash
   python merge_jarvis_datasets.py -i ~/data/jarvis_merge -o ~/data/test_merge
   ```

## YOLO 3D-to-2D

Generate 2D YOLO training data from 3D `red` labels:

```bash
python red3d2yolo.py -i path/to/labels -o output_dir -d 40
```

`-d` is the diameter of the labeled object in the same unit as your camera calibration (e.g. mm). The script scales the bounding box based on each label's depth from the camera.

Visualize the resulting dataset:

```bash
python check_yolo_dataset.py -y path/to/config.yaml [-s train/val]
```
