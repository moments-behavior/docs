# Local mode

In local mode, all cameras are connected to a single host running the GUI. Camera settings are saved as **config presets** — named subfolders under `orange_data/config/local/`, each containing the cameras you want to load together.

## Folder layout

For instance:

```
orange_data
├── config
│   ├── local
│   │   ├── 5cam
│   │   │   ├── 2002488.json
│   │   │   ├── 2002489.json
│   │   │   ├── 2002490.json
│   │   │   ├── 2002496.json
│   │   │   └── 710038.json
│   │   └── center_ceiling
│   │       └── 710038.json
│   └── network
├── exp
│   └── unsorted
│       └── 2024_10_31_13_09_41
│           ├── Cam710038_meta.csv
│           └── Cam710038.mp4
└── pictures
    └── 710038_0.tiff
```

Above, `5cam` and `center_ceiling` are two config presets. The GUI lets you pick a preset to load that set of cameras.

## Camera config JSON

Each preset contains one JSON file per camera, named after the camera serial number (e.g. `2002488.json`). See the [example config](https://github.com/moments-behavior/orange/blob/main/config/2002496.json.example) in the orange repo.

Key fields:

| Field          | Type    | Notes                                                                |
| -------------- | ------- | -------------------------------------------------------------------- |
| `name`         | string  | Display name in the GUI (e.g. `"Cam0"`)                              |
| `width`        | int     | Image width in pixels                                                |
| `height`       | int     | Image height in pixels                                               |
| `frame_rate`   | int     | Capture frame rate                                                   |
| `gain`         | int     | Sensor gain                                                          |
| `iris`         | int     | Lens iris setting (0 if not motorized)                               |
| `focus`        | int     | Lens focus setting                                                   |
| `exposure`     | int     | Exposure time                                                        |
| `pixel_format` | string  | e.g. `"BayerRG8"`, `"Mono8"`                                         |
| `gpu_id`       | int     | Which GPU handles image processing for this camera                   |
| `color_temp`   | string  | One of `CT_Off`, `CT_2800K`, `CT_3000K`, `CT_4000K`, `CT_5000K`, `CT_6500K`, `CT_Custom` |
| `gpu_direct`   | bool    | `true` to enable GPUDirect RDMA from the NIC (zero-copy)             |
| `color`        | bool    | `true` for color cameras, `false` for mono                           |
