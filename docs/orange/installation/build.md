# Build orange

With all [dependencies](index.md#software-dependencies) in place, the build is a single script.

## Clone

```bash
git clone --recursive https://github.com/moments-behavior/orange.git
cd orange
```

The `--recursive` flag fetches the submodules under `third_party/` (Dear ImGui, ImPlot, ImGuiFileDialog, IconFontCppHeaders, FlatBuffers).

## Build

```bash
./build.sh
```

This runs CMake into a `release/` directory and produces three executables by default:

- **`release/orange`** — the GUI host app
- **`release/cam_server`** — the headless networked camera node
- **`release/yolo_offline`** — standalone offline YOLOv8 inference tool

You can disable any target by passing `-DBUILD_ORANGE=OFF`, `-DBUILD_CAM_SERVER=OFF`, or `-DBUILD_YOLO_OFFLINE=OFF` to CMake. For example, `server_build.sh` builds only `cam_server`:

```bash
./server_build.sh
```

## Run

Start the GUI app:

```bash
./run.sh
```

On a headless server node, use `start_server.sh`, which auto-detects the hostname and runs `cam_server` with it:

```bash
./start_server.sh
```

Both scripts run with `sudo` because Emergent GigE Vision cameras and PTP need raw socket access.

## Optional: install a desktop launcher

```bash
./install.sh                # installs to $HOME/.local by default
./install.sh /opt           # or pass a custom prefix
```

This copies the binary, fonts, and icon under `$PREFIX/opt/orange`, drops a wrapper at `$PREFIX/bin/orange`, and creates a `.desktop` entry. Use `./uninstall.sh` to remove.
