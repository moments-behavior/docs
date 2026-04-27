# Clonezilla image

An Ubuntu image is available with preinstalled `orange` and the labeling app `red`. Contact [Jinyao Yan](mailto:yanj11@janelia.hhmi.org) for access.

!!! warning
    Restoring the image will erase all existing data on the target disk. Back up first.

## Steps

1. Download the image zip file, unzip it onto an Ubuntu local disk or USB.
2. Make a [Clonezilla live USB](https://clonezilla.org/clonezilla-live.php).
3. [Restore disk image](https://clonezilla.org/fine-print-live-doc.php?path=clonezilla-live/doc/02_Restore_disk_image).

## Post-restore: BIOS settings

Enable **PCIE Above 4G Decoding** and **Resizable Bar**. Upgrade the BIOS if your current version doesn't support them.

## System stack

The image ships with:

```
ubuntu_version:        22.04.4
kernel_version:        6.5.0-44-generic
nvidia_driver_version: 535.183.06
esdk_version:          2.55.02
cuda_version:          12.2
ffmpeg_version:        4.4
opencv_version:        4.10
tensorrt_version:      10.0.1.6
```
