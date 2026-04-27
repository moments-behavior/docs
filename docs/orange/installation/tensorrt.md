# TensorRT

`orange` has been tested with TensorRT **8.6.1.6**, **10.0.1.6**, and **10.11.0.33**. Pick the version that matches your CUDA: 8.6.x for CUDA 12.0, 10.x for CUDA 12.2+. The version examples below are illustrative — substitute the version that matches your CUDA install.

!!! warning "Engine ↔ runtime version match"
    The `tensorrt` Python wheel used to compile a YOLO `.engine` file (see [Real-time detection](../realtime-detection.md)) must match the C++ TensorRT runtime that `orange` was built against, at the same major.minor version. Engines are not portable across TensorRT versions.

Based on [NVIDIA's TAR install instructions](https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html#installing-tar) (more details there if needed).

## 1. Download and extract TensorRT

We use `TensorRT-8.6.1.6` with `cuda 12.0` — you can download it directly. For CUDA 12.2 and above, use TensorRT 10 instead (e.g. `TensorRT-10.6.0.26.Linux.x86_64-gnu.cuda-12.6`). The installation steps are similar.

```bash
cd /home/$USER/nvidia
wget https://developer.nvidia.com/downloads/compute/machine-learning/tensorrt/secure/8.6.1/tars/TensorRT-8.6.1.6.Linux.x86_64-gnu.cuda-12.0.tar.gz
tar -xzvf TensorRT-8.6.1.6.Linux.x86_64-gnu.cuda-12.0.tar.gz
```

This extracts a folder `TensorRT-8.6.1.6` with the following subdirectories:

```
bin  data  doc  include  lib  python  samples  targets
```

Rename the folder to `TensorRT`:

```bash
mv TensorRT-8.6.1.6 TensorRT
```

## 2. Add TensorRT to `LD_LIBRARY_PATH`

Add the absolute path to TensorRT's lib directory to `LD_LIBRARY_PATH`:

```bash
export LD_LIBRARY_PATH=/home/$USER/nvidia/TensorRT/lib:$LD_LIBRARY_PATH
source ~/.bashrc
```

## 3. Verify installation

Try building one of the sample programs (e.g. `trtexec`):

```bash
cd /home/$USER/nvidia/TensorRT/samples/trtexec
make
```

Run it:

```bash
cd /home/$USER/nvidia/TensorRT/bin/
./trtexec
```

`orange`'s `CMakeLists.txt` assumes TensorRT is at `$HOME/nvidia/TensorRT`. If you installed elsewhere, edit `DIR_TENSORRT` in `CMakeLists.txt`.
