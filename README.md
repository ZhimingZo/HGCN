# High-order Graph Convolutional Networks for 3D Human Pose Estimation (BMVC 2020)

This repository holds the Pytorch implementation of [High-order Graph Convolutional Networks for 3D Human Pose Estimation](https://www.bmvc2020-conference.com/assets/papers/0550.pdf) by Zhiming Zou, Kenkun Liu, Le Wang, and Wei Tang. If you find our code useful in your research, please consider citing:

```
@article{zouhigh,
  title={High-order Graph Convolutional Networks for 3D Human Pose Estimation},
  author={Zou, Zhiming and Liu, Kenkun and Wang, Le and Tang, Wei}
}
```

## Introduction

The proposed High-order Graph Convolutional Network (HGCN) is a graph convolutional network architecture that operates on regression tasks with graph-structured data. We evaluate our model for 3D human pose estimation on the [Human3.6M Dataset](http://vision.imar.ro/human3.6m/).

In this repository, only 2D joints of the human pose are exploited as inputs. We utilize the method described in Pavllo et al. [2] to normalize 2D and 3D poses in the dataset. To be specific, 2D poses are scaled according to the image resolution and normalized to [-1, 1]; 3D poses are aligned with respect to the root joint. Please refer to the corresponding part in Pavllo et al. [2] for more details. For the 2D ground truth, we predict 16 joints (as the skeleton in Martinez et al. [1] and Zhao et al. [3] without the 'Neck/Nose' joint). For the 2D pose detections, the 'Neck/Nose' joint is reserved. 

### Results on Human3.6M

Under Protocol 1 (mean per-joint position error) and Protocol 2 (mean per-joint position error after rigid alignment).

| Method | 2D Detections | # of Epochs | # of Parameters | MPJPE (P1) | P-MPJPE (P2) |
|:-------|:-------:|:-------:|:-------:|:-------:|:-------:|
| Martinez et al. [1] | Ground truth | 200  | 4.29M | 44.40 mm | 35.25 mm |
| SemGCN | Ground truth | 50 | 0.27M | 42.14 mm | 33.53 mm |
| SemGCN (w/ Non-local) | Ground truth | 30 | 0.43M | 40.78 mm | 31.46 mm |
| HGCN   | Ground truth | 50 |  1.20M  | **39.52 mm** | **31.07** |

Results using Ground truth are reported. 
The results are borrowed from [SemGCN](https://github.com/garyzhao/SemGCN).

## Quickstart

This repository is build upon Python v3.7 and Pytorch v1.3.1 on Ubuntu 18.04. All experiments are conducted on a single NVIDIA RTX 2080 Ti GPU. See [`requirements.txt`](requirements.txt) for other dependencies. We recommend installing Python v3.7 from [Anaconda](https://www.anaconda.com/) and installing Pytorch (>= 1.3.1) following guide on the [official instructions](https://pytorch.org/) according to your specific CUDA version. Then you can install dependencies with the following commands.

```
git clone git@github.com:ZhimingZou/HGCN.git
cd HGCN
pip install -r requirements.txt
```

### Dataset setup
CPN 2D detections for Human3.6M datasets are provided by [VideoPose3D](https://github.com/facebookresearch/VideoPose3D) Pavllo et al. [2], which can be downloaded by the following steps:

```
cd data
wget https://dl.fbaipublicfiles.com/video-pose-3d/data_2d_h36m_cpn_ft_h36m_dbb.npz
wget https://dl.fbaipublicfiles.com/video-pose-3d/data_2d_h36m_detectron_ft_h36m.npz
cd ..
```

GT 2D keypoints for Human3.6M datasets are provided by [SemGCN](https://github.com/garyzhao/SemGCN) Zhao et al. [3], which can be downloaded by the following steps:
```
cd data
pip install gdown
gdown https://drive.google.com/uc?id=1Ac-gUXAg-6UiwThJVaw6yw2151Bot3L1
python prepare_data_h36m.py --from-archive h36m.zip
cd ..
```
After this step, you should end up with two files in the data directory: data_3d_h36m.npz for the 3D poses, and data_2d_h36m_gt.npz for the ground-truth 2D poses.

### Evaluating our pre-trained models
The pre-trained models can be downloaded from [Google Drive](https://drive.google.com/drive/folders/13gBcCX6nQwzRN0jrhP5Fl7KwVa57-MCI?usp=sharing). Put `checkpoint` in the project root directory.

To evaluate HGCN with CPN detectors as input, run:
```
python main_gcn.py --evaluate checkpoint/pretrained/hgcn_cpn_best.pth.tar --keypoints cpn_ft_h36m_cbb
```

To evaluate HGCN with ground truth as input, run:
```
python main_gcn.py --evaluate checkpoint/pretrained/hgcn_gt_best.pth.tar --keypoints gt
```
Note that the results will be reported in an **action-wise** manner.

### Training from scratch
If you want to reproduce the results of our pretrained models, run the following commands.
For HGCN:
```
python main_gcn.py --keypoints gt
```
By default the application runs in training mode. This will train a new model for 50 epochs, using ground truth 2D detections.
If you want to try different network settings, please refer to [`main_gcn.py`](main_gcn.py) for more details. Note that the 
default setting of hyper-parameters is used for training model with CPN detectors as input, please refer to the paper for implementation details.  

For training and evaluating models using 2D detections generated by the Cascaded Pyramid Network, add `--keypoints cpn_ft_h36m_dbb` to the commands:
```
python main_gcn.py --keypoints cpn_ft_h36m_dbb
python main_gcn.py --evaluate ${CHECKPOINT_PATH} --keypoints cpn_ft_h36m_dbb

```
### Visualization
You can generate visualizations of the model predictions by running:
```
python viz.py --architecture gcn --evaluate checkpoint/pretrained/hgcn_gt_best.pth.tar --viz_subject S11 --viz_action Walking --viz_camera 0 --viz_output output.gif --viz_size 3 --viz_downsample 2 --viz_limit 60
```
The script can also export MP4 videos and supports a variety of parameters (e.g. downsampling/FPS, size, bitrate). See [`viz.py`](viz.py) for more details.

### References

[1] Martinez et al. [A simple yet effective baseline for 3d human pose estimation](https://arxiv.org/pdf/1705.03098.pdf). ICCV 2017.

[2] Pavllo et al. [3D human pose estimation in video with temporal convolutions and semi-supervised training](https://arxiv.org/pdf/1811.11742.pdf). CVPR 2019.

[3] Zhao et al. [Semantic Graph Convolutional Networks for 3D Human Pose Regression](https://arxiv.org/pdf/1904.03345.pdf). CVPR 2019.


## Acknowledgement
This code is extended from the following repositories.
- [3d-pose-baseline](https://github.com/una-dinosauria/3d-pose-baseline)
- [3d_pose_baseline_pytorch](https://github.com/weigq/3d_pose_baseline_pytorch)
- [VideoPose3D](https://github.com/facebookresearch/VideoPose3D)
- [Semantic GCN](https://github.com/garyzhao/SemGCN)

We thank the authors for releasing their codes. Please also consider citing their works.
