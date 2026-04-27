# FFmpeg with GPU encoder support

`orange` is built against FFmpeg 4.4 with NVIDIA NVENC support. Install the NVIDIA driver and CUDA toolkit before starting (see [CUDA & Emergent SDK](cuda-esdk.md)).

## Prerequisites

1. To compile FFmpeg with NVIDIA, you need `ffnvcodec`. Clone the headers repo:

    ```bash
    mkdir ~/nvidia/ && cd ~/nvidia/
    git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
    ```

2. Install `ffnvcodec`:

    ```bash
    cd nv-codec-headers && sudo make install
    ```

3. Get FFmpeg source:

    ```bash
    cd ~/nvidia/
    git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg/
    ```

4. Install build tools:

    ```bash
    sudo apt install build-essential yasm cmake libtool libc6 libc6-dev unzip wget libnuma1 libnuma-dev
    ```

    You may also need:

    ```bash
    sudo apt-get install git make nasm pkg-config libx264-dev libxext-dev libxfixes-dev zlib1g-dev
    ```

## Check out FFmpeg 4.4

This is the version `orange` builds against:

```bash
cd ~/nvidia/ffmpeg
git checkout release/4.4
mkdir build
```

If you have trouble compiling, the GPU architecture in `--nvccflags` may be too low — check `config.log`.

## Configure & build

The `--nvccflags` option pins the CUDA architecture. Pick the one matching your GPU:

For Turing (compute capability 7.5):

```bash
./configure --prefix=$(pwd)/build \
    --disable-static --enable-shared --enable-nonfree \
    --enable-cuda-nvcc --enable-libnpp \
    --extra-cflags=-I/usr/local/cuda/include \
    --extra-ldflags=-L/usr/local/cuda/lib64 \
    --nvccflags="-gencode arch=compute_75,code=sm_75 -O2"
```

For Hopper (compute capability 9.0):

```bash
./configure --prefix=$(pwd)/build \
    --disable-static --enable-shared --enable-nonfree \
    --enable-cuda-nvcc --enable-libnpp \
    --extra-cflags=-I/usr/local/cuda/include \
    --extra-ldflags=-L/usr/local/cuda/lib64 \
    --nvccflags="-gencode arch=compute_90,code=sm_90 -O2"
```

(Substitute your own arch as needed.)

Compile:

```bash
make -j $(nproc)
```

Verify the executable:

```bash
ls -l ffmpeg
./ffmpeg
```

Install:

```bash
make install
```

## Add to PATH

If you followed the steps above, FFmpeg lives at `$HOME/nvidia/ffmpeg/build`. Add it to `.bashrc`:

```bash
export PATH=/usr/local/cuda/bin:/home/$USER/nvidia/ffmpeg/build/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/home/$USER/nvidia/ffmpeg/build/lib:$LD_LIBRARY_PATH
```

`orange`'s `CMakeLists.txt` assumes `$HOME/nvidia/ffmpeg`. If you installed elsewhere, edit `DIR_FFMPEG` in `CMakeLists.txt`.
