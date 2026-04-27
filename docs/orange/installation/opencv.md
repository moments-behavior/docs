# OpenCV

`orange` requires OpenCV with the `opencv_sfm` module. If you also use the [Red labeling app](https://github.com/moments-behavior/red), build OpenCV with cuDNN support; otherwise the simpler CPU-only build is sufficient.

## Install dependencies first

Download and unzip `opencv-4.8.0.zip` and `opencv_contrib-4.8.0.zip` to `~/build/`. (For CUDA 12.2, use OpenCV 4.10 instead.)

To build OpenCV with `opencv_sfm`, follow [OpenCV's SFM installation guide](https://docs.opencv.org/4.x/db/db8/tutorial_sfm_installation.html) to install the SFM dependency.

Ceres solver is optional. If you want it, see [Ceres' installation guide](http://ceres-solver.org/installation.html#linux). For simplicity, disable CUDA in Ceres:

```bash
git clone --recurse-submodules https://github.com/ceres-solver/ceres-solver
cd ceres-solver
mkdir build && cd build
cmake -DUSE_CUDA=OFF ..
make -j $(nproc)
make test
sudo make install
```

## Build option A — with cuDNN (recommended if you use Red)

### Install cuDNN

Requires CUDA. We use cuDNN 8.9.3 with driver 525.105.17 and CUDA 12.0.

Download `cudnn-linux-x86_64-8.9.3.28_cuda12-archive.tar.xz` from the [cuDNN archives](https://developer.nvidia.com/rdp/cudnn-archive). Extract and copy:

```bash
sudo cp cudnn-*-archive/include/cudnn*.h /usr/local/cuda/include
sudo cp -P cudnn-*-archive/lib/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

Verify:

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

### Build OpenCV with CUDA + cuDNN

```bash
cd opencv-4.8.0/
mkdir build
cd build
```

```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_TBB=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D WITH_CUDA=ON \
-D BUILD_opencv_cudacodec=OFF \
-D WITH_CUDNN=ON \
-D OPENCV_DNN_CUDA=ON \
-D CUDA_ARCH_BIN=7.5 \
-D WITH_V4L=ON \
-D WITH_QT=ON \
-D WITH_OPENGL=ON \
-D WITH_GSTREAMER=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D OPENCV_PC_FILE_NAME=opencv.pc \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/build/opencv_contrib-4.8.0/modules \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D INSTALL_C_EXAMPLES=ON \
-D BUILD_EXAMPLES=ON ..
```

```bash
make -j $(nproc)
sudo make install
```

## Build option B — without CUDA (simpler)

If you only need `orange` (no Red), this is enough:

```bash
cd opencv-4.8.0/
mkdir build
cd build
```

```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_TBB=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUDA=OFF \
-D BUILD_opencv_cudacodec=OFF \
-D WITH_OPENGL=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D OPENCV_PC_FILE_NAME=opencv.pc \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/build/opencv_contrib-4.8.0/modules \
-D INSTALL_PYTHON_EXAMPLES=OFF ..
```

This assumes `opencv_contrib-4.8.0` is at `~/build/opencv_contrib-4.8.0` and installs OpenCV at `/usr/local`.

```bash
make -j $(nproc)
sudo make install
```
