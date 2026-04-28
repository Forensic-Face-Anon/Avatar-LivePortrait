<h1 align="center">LivePortrait: Efficient Portrait Animation with Stitching and Retargeting Control</h1>


<!-- ===== LivePortrait – Quick Start & Links ===== -->
<div align="center">

  <!-- 🎬 Showcase GIF -->
  <p><img src="./assets/docs/showcase2.gif" alt="LivePortrait showcase GIF"></p>
  <p>🔥 For the actual repo, visit their <a href="https://liveportrait.github.io/" target="_blank"><strong>homepage</strong></a> 🔥</p>

</div>
<!-- ===== /LivePortrait ===== -->



## Introduction
This repo, named **LivePortrait**, contains the official PyTorch implementation of the paper [LivePortrait: Efficient Portrait Animation with Stitching and Retargeting Control](https://arxiv.org/pdf/2407.03168).

<p><img src="./assets/docs/results.gif" alt="LivePortrait showcase GIF"></p>

For our study, we propose the use of portrait animation on a neutral face mesh to extract emotions while preserving identity instead of having to rendering a 3D facial mesh.


## Installation
### 1. Clone the code and prepare the environment 🛠️

> [!Note]
> Make sure your system has [`git`](https://git-scm.com/), [`conda`](https://anaconda.org/anaconda/conda), and [`FFmpeg`](https://ffmpeg.org/download.html) installed. For details on FFmpeg installation, see [**how to install FFmpeg**](assets/docs/how-to-install-ffmpeg.md).

```bash
git clone https://github.com/KlingTeam/LivePortrait
cd LivePortrait

# create env using conda
conda create -n LivePortrait python=3.10
conda activate LivePortrait
```

#### For Linux 🐧 or Windows 🪟 Users
[X-Pose](https://github.com/IDEA-Research/X-Pose), required by Animals mode, is a dependency that needs to be installed. The step of `Check your CUDA versions` is **optional** if you only want to run Humans mode.

<details>
  <summary>Check your CUDA versions</summary>

  Firstly, check your current CUDA version by:
  ```bash
  nvcc -V # example versions: 11.1, 11.8, 12.1, etc.
  ```

  Then, install the corresponding torch version. Here are examples for different CUDA versions. Visit the [PyTorch Official Website](https://pytorch.org/get-started/previous-versions) for installation commands if your CUDA version is not listed:
  ```bash
  # for CUDA 11.1
  pip install torch==1.10.1+cu111 torchvision==0.11.2 torchaudio==0.10.1 -f https://download.pytorch.org/whl/cu111/torch_stable.html
  # for CUDA 11.8
  pip install torch==2.3.0 torchvision==0.18.0 torchaudio==2.3.0 --index-url https://download.pytorch.org/whl/cu118
  # for CUDA 12.1
  pip install torch==2.3.0 torchvision==0.18.0 torchaudio==2.3.0 --index-url https://download.pytorch.org/whl/cu121
  # ...
  ```

  **Note**: On Windows systems, some higher versions of CUDA (such as 12.4, 12.6, etc.) may lead to unknown issues. You may consider downgrading CUDA to version 11.8 for stability. See the [downgrade guide](https://github.com/dimitribarbot/sd-webui-live-portrait/blob/main/assets/docs/how-to-install-xpose.md#cuda-toolkit-118) by [@dimitribarbot](https://github.com/dimitribarbot).
</details>


Finally, install the remaining dependencies:
```bash
pip install -r requirements.txt
```


### 2. Download pretrained weights

The easiest way to download the pretrained weights is from HuggingFace:
```bash
# !pip install -U "huggingface_hub[cli]"
huggingface-cli download KlingTeam/LivePortrait --local-dir pretrained_weights --exclude "*.git*" "README.md" "docs"
```

If you cannot access to Huggingface, you can use [hf-mirror](https://hf-mirror.com/) to download:
```bash
# !pip install -U "huggingface_hub[cli]"
export HF_ENDPOINT=https://hf-mirror.com
huggingface-cli download KlingTeam/LivePortrait --local-dir pretrained_weights --exclude "*.git*" "README.md" "docs"
```

Alternatively, you can download all pretrained weights from [Google Drive](https://drive.google.com/drive/folders/1UtKgzKjFAOmZkhNK-OYT0caJ_w2XAnib) or [Baidu Yun](https://pan.baidu.com/s/1MGctWmNla_vZxDbEp2Dtzw?pwd=z5cn). Unzip and place them in `./pretrained_weights`.

Ensuring the directory structure is as or contains [**this**](assets/docs/directory-structure.md).

### 3. Inference

#### Fast hands-on (humans) 👤
```bash
# For Linux and Windows users
python inference.py

```

If the script runs successfully, you will get an output mp4 file named `animations/s6--d0_concat.mp4`. This file includes the following results: driving video, input image or video, and generated result.

<p align="center">
  <img src="./assets/docs/inference.gif" alt="image">
</p>

Or, you can change the input by specifying the `-s` and `-d` arguments:

```bash
# source input is an image
python inference.py -s data/source/Mask_with_iris.jpg -d adata/d0.mp4

# source input is a video ✨
python inference.py -s assets/examples/source/s13.mp4 -d assets/examples/driving/d0.mp4

# more options to see
python inference.py -h
```

A gradio version also exist under `./app.py` but was not used in this study.
<p align="center">
  <img src="assets/docs/retargeting-video-2024-08-02.jpg">
</p>

#### Fast hands-on (animals) 🐱🐶
Animals mode is ONLY tested on Linux and Windows with NVIDIA GPU.

You need to build an OP named `MultiScaleDeformableAttention` first (refer to the <a href="#for-linux--or-windows--users">Check your CUDA versions</a> if needed), which is used by [X-Pose](https://github.com/IDEA-Research/X-Pose), a general keypoint detection framework.

```bash
cd src/utils/dependencies/XPose/models/UniPose/ops
python setup.py build install
cd - # equal to cd ../../../../../../../
```

Then
```bash
python inference_animals.py -s assets/examples/source/s39.jpg -d assets/examples/driving/wink.pkl --driving_multiplier 1.75 --no_flag_stitching
```
If the script runs successfully, you will get an output mp4 file named `animations/s39--wink_concat.mp4`.
<p align="center">
  <img src="./assets/docs/inference-animals.gif" alt="image">
</p>

#### Driving video auto-cropping 📢📢📢
> [!IMPORTANT]
> To use your own driving video, we **recommend**: ⬇️
> - Crop it to a **1:1** aspect ratio (e.g., 512x512 or 256x256 pixels), or enable auto-cropping by `--flag_crop_driving_video`.
> - Focus on the head area, similar to the example videos.
> - Minimize shoulder movement.
> - Make sure the first frame of driving video is a frontal face with **neutral expression**.

Below is an auto-cropping case by `--flag_crop_driving_video`:
```bash
python inference.py -s assets/examples/source/s9.jpg -d assets/examples/driving/d13.mp4 --flag_crop_driving_video
```

If you find the results of auto-cropping is not well, you can modify the `--scale_crop_driving_video`, `--vy_ratio_crop_driving_video` options to adjust the scale and offset, or do it manually.

#### Motion template making
You can also use the auto-generated motion template files ending with `.pkl` to speed up inference, and **protect privacy**, such as:
```bash
python inference.py -s assets/examples/source/s9.jpg -d assets/examples/driving/d5.pkl # portrait animation
python inference.py -s assets/examples/source/s13.mp4 -d assets/examples/driving/d5.pkl # portrait video editing
```

## Ethics Considerations 🛡️
Portrait animation technologies come with social risks, particularly the potential for misuse in creating deepfakes. To mitigate these risks, it’s crucial to follow ethical guidelines and adopt responsible usage practices. At present, the synthesized results contain visual artifacts that may help in detecting deepfakes. Please note that we do not assume any legal responsibility for the use of the results generated by this project.

## Acknowledgements 💖
We would like to thank the contributors of [FOMM](https://github.com/AliaksandrSiarohin/first-order-model), [Open Facevid2vid](https://github.com/zhanglonghao1992/One-Shot_Free-View_Neural_Talking_Head_Synthesis), [SPADE](https://github.com/NVlabs/SPADE), [InsightFace](https://github.com/deepinsight/insightface) and [X-Pose](https://github.com/IDEA-Research/X-Pose) repositories, for their open research and contributions.

```bibtex
@article{guo2024liveportrait,
  title   = {LivePortrait: Efficient Portrait Animation with Stitching and Retargeting Control},
  author  = {Guo, Jianzhu and Zhang, Dingyun and Liu, Xiaoqiang and Zhong, Zhizhou and Zhang, Yuan and Wan, Pengfei and Zhang, Di},
  journal = {arXiv preprint arXiv:2407.03168},
  year    = {2024}
}
```
Contact
[**Jianzhu Guo (郭建珠)**](https://guojianzhu.com); **guojianzhu1994@gmail.com**

*Long live in arXiv.*
