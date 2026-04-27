# Real-time detection

`orange` can run a TensorRT-compiled YOLOv8 model in real time on the camera streams. The application looks for compiled engine files under `orange_data/detect/`. Make sure each camera's `gpu_id` in its config JSON matches the GPU the engine was compiled for.

The workflow below is adapted from [YOLOv8-TensorRT](https://github.com/triple-Mu/YOLOv8-TensorRT).

## Preparation

You'll need an `.mp4` recording to create training data for your YOLO model. Label frames from your recording and create a YOLOv8-compatible dataset.

## Set up the YOLOv8-TensorRT repo

```bash
cd ~/src/
git clone https://github.com/triple-Mu/YOLOv8-TensorRT
cd YOLOv8-TensorRT

python3 -m venv .venv
source .venv/bin/activate
pip install ultralytics
```

## Train a YOLOv8 model

Download your preferred model size from [Ultralytics' Hugging Face](https://huggingface.co/Ultralytics/YOLOv8/tree/main). Larger models give higher prediction latency.

Train on your custom dataset (run from the dataset folder):

```bash
yolo task=detect mode=train model=yolov8m.pt data=data.yaml epochs=100 imgsz=640
```

You'll get a `.pt` file in `runs/detect/train/weights/`. Copy `best.pt` into the YOLOv8-TensorRT clone:

```bash
cd runs/detect/train/weights
cp ./best.pt ~/src/YOLOv8-TensorRT/best.pt
cd ~/src/YOLOv8-TensorRT/
```

## Compile the engine

Install requirements:

```bash
pip install -r requirements.txt
pip install tensorrt==10.11.0.33
```

!!! warning "Pin `tensorrt` to match your `orange` build"
    The Python wheel used here compiles the `.engine` file, and the C++ TensorRT runtime that `orange` is linked against must load it. They have to match at the same major.minor version. `orange` has been tested with TensorRT 8.6.1.6, 10.0.1.6, and 10.11.0.33 — replace the pin above with whichever runtime version you installed (see [TensorRT installation](installation/tensorrt.md)).

Convert `.pt` to `.onnx`:

```bash
python3 export-det.py \
  --weights best.pt \
  --iou-thres 0.65 \
  --conf-thres 0.25 \
  --topk 100 \
  --opset 11 \
  --sim \
  --input-shape 1 3 640 640 \
  --device cuda:0
```

Adjust to your use case:

- `--iou-thres` and `--conf-thres` per setup
- `--topk` is the maximum number of bounding boxes drawn

Compile `.onnx` to `.engine` with `trtexec`:

```bash
~/nvidia/TensorRT/bin/trtexec --onnx=best.onnx --saveEngine=best.engine --device=0 --fp16
```

`trtexec` will print the device used for compilation and the available devices. Note which device your video output is on — change the `--device` flag if needed.

## Drop the engine into orange

Move the `.engine` into the directory `orange` looks at:

```bash
cp ./best.engine ~/orange_data/detect/best.engine
```

In the camera config JSONs (in `~/orange_data/config/local/<preset>/` or `~/orange_data/config/network/<preset>/`), set every camera's `gpu_id` to the ID of the GPU you used for compilation.
