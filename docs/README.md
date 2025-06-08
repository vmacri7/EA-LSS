# EA-LSS Documentation

This document provides detailed information about the EA-LSS project including an overview of the paper, a description of the model architecture and practical instructions for setting up and running the code.

## Paper Overview
EA-LSS (Edge-aware Lift-splat-shot) addresses the **depth jump** problem in LSS-based 3D object detection. The framework introduces two key ideas:

1. **Edge-aware Depth Fusion (EADF)** – depth maps from multiple views are fused with edge information to better handle depth discontinuities.
2. **Fine-grained Depth (FGD)** supervision – an additional loss that encourages finer depth estimation and provides better geometric cues for BEV reasoning.

These improvements can be plugged into existing LSS models and are effective for both camera-only and multi-modal (camera+LiDAR) setups. The method achieves state-of-the-art accuracy on the nuScenes 3D object detection benchmark.

## Model Architecture
EA-LSS follows the common LSS pipeline while inserting edge-aware components. A simplified flow is:

1. **Image Backbone** – extracts 2D features from each camera.
2. **Depth Estimation with FGD** – predicts per-pixel depth distributions with additional fine-grained depth supervision.
3. **Lift-Splat Projection** – features are lifted along the depth dimension and splatted into the BEV plane.
4. **EADF Module** – uses edge cues to blend features from neighboring pixels to mitigate artifacts from depth jumps.
5. **BEV Encoder and Detection Head** – performs BEV feature processing and predicts 3D bounding boxes.
6. **(Optional) LiDAR Branch** – in multi-modal configurations, LiDAR features are fused before the detection head.

Figure `photo/EA-LSS_arch.png` in the repository illustrates the full architecture.

## Environment Setup

EA-LSS is built on [MMDetection3D](https://github.com/open-mmlab/mmdetection3d). The following steps show how to prepare an environment using conda:

```bash
# clone repository
git clone https://github.com/zhiwangzhang/EA-LSS.git
cd EA-LSS

# create environment (Python >=3.6 is required)
conda create -n ealss python=3.8 -y
conda activate ealss

# install PyTorch and CUDA (check https://pytorch.org for the correct command)
# Example for CUDA 11.3
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu113

# install other dependencies
pip install -r requirements/runtime.txt

# compile and install this project
pip install -v -e .
```

Download the nuScenes dataset following the instructions on the [nuScenes website](https://www.nuscenes.org/). Set the `data_root` in the config files to point to the dataset directory.

## Training and Evaluation

Below is the typical workflow to train and evaluate EA-LSS.

```bash
# train the camera branch
./tools/dist_train.sh configs/EALSS/cam_stream/ealss_4x8_20e_nusc_cam.py 8

# train the LiDAR branch (for multi-modal setup)
./tools/dist_train.sh configs/EALSS/lidar_stream/transfusion_nusc_voxel_L.py 8

# train the fusion model using pretrained camera and LiDAR weights
./tools/dist_train.sh configs/EALSS/ealss_4x8_10e_nusc_aug_large.py 8

# evaluate a trained model
./tools/dist_test.sh configs/EALSS/ealss_4x8_10e_nusc_aug_large.py WEIGHT_PATH 8 --eval bbox
```

The configs in `configs/EALSS/` include example settings for different setups. Refer to the comments inside each config for further hyper‑parameters.

## Citation
If you use this project in your research, please cite:

```bibtex
@article{hu2023ealss,
  title={EA-LSS: Edge-aware Lift-splat-shot Framework for 3D BEV Object Detection},
  author={Haotian Hu and Fanyi Wang and Jingwen Su and Yaonong Wang and Laifeng Hu and Weiye Fang and Jingwei Xu and Zhiwang Zhang},
  journal={arXiv preprint arXiv:2303.17895},
  year={2023}
}
```

## Acknowledgement
This repository is based on [mmdetection3d](https://github.com/open-mmlab/mmdetection3d) and adapts components from [BEVFusion](https://github.com/ADLab-AutoDrive/BEVFusion) and [TransFusion](https://github.com/XuyangBai/TransFusion).
