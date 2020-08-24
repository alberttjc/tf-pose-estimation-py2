
# Important
This repo is modified to implement Python 2.7 and Tensorflow 1.14 for the purposes of integrating it in ROS Melodic

Original repo utilised Python 3 which is not compatible with ROS (Python 2.7)
Source : https://github.com/ildoonet/tf-pose-estimation

# tf-pose-estimation

'Openpose', human pose estimation algorithm, have been implemented using Tensorflow. It also provides several variants that have some changes to the network structure for **real-time processing on the CPU or low-power embedded devices.**

**You can even run this on your macbook with a descent FPS!**

Original Repo(Caffe) : https://github.com/CMU-Perceptual-Computing-Lab/openpose

| CMU's Original Model</br> on Macbook Pro 15" | Mobilenet-thin </br>on Macbook Pro 15" | Mobilenet-thin</br>on Jetson TX2 |
|:---------|:--------------------|:----------------|
| ![cmu-model](/etcs/openpose_macbook_cmu.gif)     | ![mb-model-macbook](/etcs/openpose_macbook_mobilenet3.gif) | ![mb-model-tx2](/etcs/openpose_tx2_mobilenet3.gif) |
| **~0.6 FPS** | **~4.2 FPS** @ 368x368 | **~10 FPS** @ 368x368 |
| 2.8GHz Quad-core i7 | 2.8GHz Quad-core i7 | Jetson TX2 Embedded Board |

Implemented features are listed here : [features](./etcs/feature.md)

## Important Updates

- 2019.3.12 Add new models using mobilenet-v2 architecture. See : [experiments.md](./etcs/experiments.md)
- 2018.5.21 Post-processing part is implemented in c++. It is required compiling the part. See: https://github.com/ildoonet/tf-pose-estimation/tree/master/src/pafprocess
- 2018.2.7 Arguments in run.py script changed. Support dynamic input size.

## Install

### Dependencies

You need dependencies below.

- ~~python3~~
- **python2.7**
- tensorflow 1.14
- opencv3, protobuf, python3-tk
- slidingwindow
  - https://github.com/adamrehn/slidingwindow
  - I copied from the above git repo to modify few things.

### 1. Pre-Install Jetson case

```bash
$ sudo apt-get install libllvm-7-ocaml-dev libllvm7 llvm-7 llvm-7-dev llvm-7-doc llvm-7-examples llvm-7-runtime
$ export LLVM_CONFIG=/usr/bin/llvm-config-7
```

### 2. Installing 3rd-party Libraries

> **for supporting python2, you need follow step before install requirements.txt**
```bash
//llvmlite latest py2 version is 0.32.0
$ python -m pip install llvmlite==0.32.0
//slim support
$ python -m pip install tf_slim
//numba suport
$ python -m pip install numba --no-deps
//matplotlib support (IDK why pip matplotlib always load GTK, use apt instead pip to fix this issue)
$ sudo apt-get install python-matplotlib
```
### 3. Clone repo and install requirements

```bash
$ git clone https://github.com/alberttjc/tf-pose-estimation-py2.git
$ cd tf-pose-estimation
$ pip install -r requirements.txt
```
### 4. Build c++ Library for Post Processing

```
$ sudo apt install swig
$ cd tf_pose/pafprocess
$ swig -python -c++ pafprocess.i && python setup.py build_ext --inplace
```
Details : https://github.com/ildoonet/tf-pose-estimation/tree/master/tf_pose/pafprocess

### 5. Package Install
Install this repo as a shared package using pip.
```
$ cd tf-pose-estimation-py2
$ python setup.py install
$ pip install -e .
```

## Models & Performances

See [experiments.md](./etcs/experiments.md)

### Download Tensorflow Graph File(pb file)

Before running demo, you should download graph files. You can deploy this graph on your mobile or other platforms.

- cmu (trained in 656x368)
- mobilenet_thin (trained in 432x368)
- mobilenet_v2_large (trained in 432x368)
- mobilenet_v2_small (trained in 432x368)

CMU's model graphs are too large for git, so I uploaded them on an external cloud. You should download them if you want to use cmu's original model. Download scripts are provided in the model folder.

```
$ cd models/graph/cmu
$ bash download.sh
```

## Demo

### Test Inference

You can test the inference feature with a single image.

```
$ python run.py --model=mobilenet_thin --resize=432x368 --image=./images/p1.jpg
```

The image flag MUST be relative to the src folder with no "~", i.e:
```
--image ../../Desktop
```

Then you will see the screen as below with pafmap, heatmap, result and etc.

![inferent_result](./etcs/inference_result2.png)

### Realtime Webcam

```
$ python run_webcam.py --model=mobilenet_thin --resize=432x368 --camera=0
```

Then you will see the realtime webcam screen with estimated poses as below. This [Realtime Result](./etcs/openpose_macbook13_mobilenet2.gif) was recored on macbook pro 13" with 3.1Ghz Dual-Core CPU.

### Video

You can test the inference feature with the video ./etcs/dance.mp4

```
$ python run_video.py --model=mobilenet_thin --resolution=432x368 --video=./etcs/dance.mp4
```

## Python Usage

This pose estimator provides simple python classes that you can use in your applications.

See [run.py](run.py) or [run_webcam.py](run_webcam.py) as references.

```python
e = TfPoseEstimator(get_graph_path(args.model), target_size=(w, h))
humans = e.inference(image)
image = TfPoseEstimator.draw_humans(image, humans, imgcopy=False)
```

If you installed it as a package,

```python
import tf_pose
coco_style = tf_pose.infer(image_path)
```

## ROS Support

See : [etcs/ros.md](./etcs/ros.md)

## Training

See : [etcs/training.md](./etcs/training.md)

## References

See : [etcs/reference.md](./etcs/reference.md)
