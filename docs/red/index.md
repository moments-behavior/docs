# red

A 3D multi-camera labeling tool for fast review and triangulation across many synchronized video streams.

![gui](../assets/red-gui.png)

## Overview

`red` is the labeling counterpart to [orange](../orange/index.md). It takes multi-view video (typically recorded with `orange`) and lets you label keypoints across all camera views simultaneously, with real-time GPU decoding (h264 / hevc), synchronized playback across all cameras, and multi-view triangulation. Labeled data can be exported for downstream training (YOLO detection, YOLO pose, JARVIS).

[Source on GitHub](https://github.com/moments-behavior/red)

## Video demo

<div style="position: relative; padding-bottom: 56.25%; height: 0;">
  <iframe src="https://www.youtube.com/embed/9eOJaadE1Nc"
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
    frameborder="0" allowfullscreen></iframe>
</div>

## Features

1. Real-time GPU-accelerated decoding (h264, hevc)
2. Synchronized playback across multiple cameras
3. Multi-view keypoint labeling and triangulation
4. Skeleton authoring with `skeleton.json`
5. Exporters for YOLO and JARVIS training pipelines

## Where to start

- **New install:** [Installation](installation/index.md) — dependencies and build.
- **Already built:** [Configuration](configuration.md) (default folders, project layout).
- **Exporting labels for training:** [Data export](data-export.md).
