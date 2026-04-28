# LibTorch

`red` links against the PyTorch C++ distribution (LibTorch). The CPU build is sufficient.

## Download

From the [PyTorch download page](https://pytorch.org/get-started/locally/), select:

- OS: **Linux**
- Package: **LibTorch**
- Language: **C++ / Java**
- Compute Platform: **CPU**

Download the resulting zip.

## Install

Unzip into `red/lib/`:

```bash
cd ~/src/red
unzip ~/Downloads/libtorch-*.zip -d lib/
```

Result:

```
red/lib/libtorch/
├── bin
├── include
├── lib
└── share
```

`red`'s `CMakeLists.txt` references `${CMAKE_SOURCE_DIR}/lib/libtorch` — no further config needed.
