# cuDNN

`red` requires NVIDIA cuDNN. The installation depends on a working CUDA install — see [orange's CUDA & Emergent SDK guide](../../orange/installation/cuda-esdk.md) (the eSDK section is `orange`-specific and can be skipped for `red`).

We use **cuDNN 8.9.3** with **driver 525.105.17** and **CUDA 12.0**. For other CUDA versions, pick a matching cuDNN from the [cuDNN version archives](https://developer.nvidia.com/rdp/cudnn-archive).

## 1. Download and extract

Download `cudnn-linux-x86_64-8.9.3.28_cuda12-archive.tar.xz` from the cuDNN archives. Extract it.

## 2. Copy files into your CUDA install

Assuming CUDA is at `/usr/local/cuda`:

```bash
sudo cp cudnn-*-archive/include/cudnn*.h /usr/local/cuda/include
sudo cp -P cudnn-*-archive/lib/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

## 3. Verify installation

```bash
source ~/.bashrc
cat /usr/local/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```

Expected output:

```
#define CUDNN_MAJOR 8
#define CUDNN_MINOR 9
#define CUDNN_PATCHLEVEL 3
--
#define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)
```

cuDNN is now available to OpenCV (rebuild OpenCV with `WITH_CUDNN=ON` — see [orange's OpenCV guide](../../orange/installation/opencv.md), Build option A) and to any other framework that links against it.
