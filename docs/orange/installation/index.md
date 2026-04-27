# Installation

`orange` is Linux-only and has been tested on NVIDIA GPUs with compute capability **7.5 (Turing)** and **8.0 (Ampere)**. The CMake build produces a fatbin for both architectures by default. To target a different one, pass `-DCMAKE_CUDA_ARCHITECTURES=<code>` at configure time (e.g. `86` for RTX 30-series, `89` for Ada Lovelace, `90` for Hopper).

!!! tip "Skip the install — use the prebuilt image"
    An Ubuntu image is available with preinstalled `orange` and the labeling app `red`. Contact the developer for access — see [Clonezilla image](../clonezilla.md).

## Reference system stack

This is the stack shipped on the [Clonezilla image](../clonezilla.md) and is the reference configuration:

| Component       | Version          |
| --------------- | ---------------- |
| Ubuntu          | 22.04.4          |
| Linux kernel    | 6.5.0-44-generic |
| NVIDIA driver   | 535.183.06       |
| Emergent eSDK   | 2.55.02          |
| CUDA            | 12.2             |
| FFmpeg          | 4.4              |
| OpenCV          | 4.10             |
| TensorRT        | 10.0.1.6         |

## Hardware requirements

- **NVENC required.** Video encoding is done on the GPU via NVIDIA NVENC, so the GPU must support it. Most consumer/workstation NVIDIA GPUs do (RTX series, A6000, etc.); some datacenter compute parts (e.g. A100) have limited or no NVENC. Check NVIDIA's [Video Encode and Decode GPU Support Matrix](https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new) before buying hardware.
- **Kernel pinned to 6.5.** As of writing, the Emergent eSDK does not support kernel 6.8. Disable network access during Ubuntu install to prevent an automatic upgrade — see [CUDA & Emergent SDK setup](cuda-esdk.md).
- **BIOS settings.** Enable **PCIE Above 4G Decoding** and **Resizable Bar**. You may need to update your BIOS if these options aren't present.

## Software dependencies

Walkthroughs for each:

1. [CUDA & Emergent SDK](cuda-esdk.md) — Linux setup, NIC config, NVIDIA driver, CUDA, GPU-Direct
2. [FFmpeg 4.4](ffmpeg.md) — built with NVENC support
3. [OpenCV](opencv.md) — with `opencv_sfm`, optionally with cuDNN
4. [TensorRT](tensorrt.md) — for real-time detection
5. **OpenGL and GLEW** (one apt-get):

    ```bash
    sudo apt-get install libglfw3 libglfw3-dev libglew-dev
    ```

6. **ENET**: follow [enet.bespin.org](http://enet.bespin.org/Installation.html), then `sudo ldconfig` (or reboot).
7. **Submodules** (Dear ImGui, ImPlot, ImGuiFileDialog, IconFontCppHeaders, FlatBuffers) are bundled in `third_party/` — fetched automatically with `git clone --recursive`. No separate install.

## Build orange

Once dependencies are in place, see [Build orange](build.md).
