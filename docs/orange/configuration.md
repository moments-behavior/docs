# Configuration

When first opened, `orange` creates an `orange_data/` folder next to the binary with the following structure:

```
orange_data
├── calib_yaml
├── config
│   ├── local
│   └── network
├── detect       # TensorRT engine files for real-time detection
├── exp
│   ├── calibration
│   └── unsorted # default save location for recordings
└── pictures     # camera-frame snapshots
```

There are two modes of using the application: **local** vs **network**. See [Local mode](local-mode.md) and [Network mode](network-mode.md).

## User config (optional)

A few defaults can be overridden by creating `~/.config/orange/config.json`:

```json
{
  "recording_folder": "/mnt/data/orange_recordings"
}
```

Recognized keys:

- `recording_folder` — moves the recordings root (`exp/unsorted`, `exp/calibration`) out of `orange_data/` and onto a different disk. The rest of `orange_data/` (configs, calibration files, snapshots) stays where it is.
- `codec` — default encoder. Either `h264` (the default) or `hevc`.
