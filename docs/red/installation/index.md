# Installation

`red` is Linux-only and shares most of its dependency stack with [orange](../../orange/index.md). If you've already set up an `orange` host, the only `red`-specific additions are **cuDNN** and **LibTorch**.

## Dependencies

| Dependency               | Notes                                                                                                            |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| NVIDIA driver + CUDA     | See [orange's CUDA & Emergent SDK guide](../../orange/installation/cuda-esdk.md) (skip the eSDK section — `red` doesn't need it). |
| FFmpeg 4.4               | See [orange's FFmpeg guide](../../orange/installation/ffmpeg.md).                                                |
| OpenCV (with cuDNN)      | Build OpenCV with the **cuDNN-enabled** options. See [orange's OpenCV guide](../../orange/installation/opencv.md), Build option A. |
| TensorRT                 | See [orange's TensorRT guide](../../orange/installation/tensorrt.md).                                            |
| **cuDNN**                | Required by `red`. See [cuDNN](cudnn.md).                                                                        |
| **LibTorch (CPU)**       | Required by `red`. See [LibTorch](libtorch.md).                                                                  |
| OpenGL + GLEW            | `sudo apt-get install libglfw3 libglfw3-dev libglew-dev`                                                         |

`red`'s submodules (Dear ImGui, ImPlot, ImGuiFileDialog, IconFontCppHeaders) live under `lib/` and are fetched automatically by `git clone --recursive`.

## Build & run

Once dependencies are in place, see [Build red](build.md).
