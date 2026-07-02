# SynCamMaster: Synchronizing Multi-Camera Video Generation from Diverse Viewpoints

<div align="center">
<div align="center" style="margin-top: 0px; margin-bottom: 0px;">
<img src=https://github.com/user-attachments/assets/b33c5b67-3881-4fa3-b853-f932eebc9c50 width="50%"/>
</div>

### [<a href="https://arxiv.org/abs/2412.07760" target="_blank">arXiv</a>] [<a href="https://jianhongbai.github.io/SynCamMaster/" target="_blank">Project Page</a>] [<a href="https://huggingface.co/datasets/KwaiVGI/SynCamVideo-Dataset/" target="_blank">Dataset</a>]

_**[Jianhong Bai<sup>1*</sup>](https://jianhongbai.github.io/), [Menghan Xia<sup>2†</sup>](https://menghanxia.github.io/), [Xintao Wang<sup>2</sup>](https://xinntao.github.io/), [Ziyang Yuan<sup>3</sup>](https://scholar.google.ru/citations?user=fWxWEzsAAAAJ&hl=en), [Xiao Fu<sup>4</sup>](https://fuxiao0719.github.io/), <br>[Zuozhu Liu<sup>1</sup>](https://person.zju.edu.cn/en/lzz), [Haoji Hu<sup>1</sup>](https://person.zju.edu.cn/en/huhaoji), [Pengfei Wan<sup>2</sup>](https://scholar.google.com/citations?user=P6MraaYAAAAJ&hl=en), [Di Zhang<sup>2</sup>](https://openreview.net/profile?id=~Di_ZHANG3)**_
<br>
(*Work done during an internship at KwaiVGI, Kuaishou Technology †corresponding author)

<sup>1</sup>Zhejiang University, <sup>2</sup>Kuaishou Technology, <sup>3</sup>Tsinghua University, <sup>4</sup>CUHK.

**ICLR 2025**

</div>

**Important Note:** This open-source repository is intended to provide a reference implementation. Due to the difference in the underlying T2V model's performance, the open-source version may not achieve the same performance as the model in our paper.

## 🔥 Updates
- __[2025.04.15]__: Please feel free to explore our subsequent work, [ReCamMaster](https://github.com/KwaiVGI/ReCamMaster).
- __[2025.04.15]__: Update a new version of the [SynCamVideo Dataset](https://huggingface.co/datasets/KwaiVGI/SynCamVideo-Dataset).
- __[2025.04.15]__: Release the [training and inference code](https://github.com/KwaiVGI/SynCamMaster?tab=readme-ov-file#%EF%B8%8F-code-syncammaster--wan21-inference--training), [model checkpoint](https://huggingface.co/KwaiVGI/SynCamMaster-Wan2.1/blob/main/step20000.ckpt).
- __[2024.12.10]__: Release the [project page](https://jianhongbai.github.io/SynCamMaster/) and the [SynCamVideo Dataset](https://huggingface.co/datasets/KwaiVGI/SynCamVideo-Dataset/).
  
## 📖 Introduction

**TL;DR:** We propose SynCamMaster, an efficient method to lift pre-trained text-to-video models for open-domain multi-camera video generation from diverse viewpoints. We also release a multi-camera synchronized video [dataset](https://huggingface.co/datasets/KwaiVGI/SynCamVideo-Dataset) rendered with Unreal Engine 5. <br>

https://github.com/user-attachments/assets/1ecfaea8-5d87-4bb5-94fc-062f84bd67a1

## ⚙️ Code: SynCamMaster + Wan2.1 (Inference & Training)
The model utilized in our paper is an internally developed T2V model, not [Wan2.1](https://github.com/Wan-Video/Wan2.1). Due to company policy restrictions, we are unable to open-source the model used in the paper. Consequently, we migrated SynCamMaster to Wan2.1 to validate the effectiveness of our method. Due to differences in the underlying T2V model, you may not achieve the same results as demonstrated in the demo.
### Inference
Step 1: Set up the environment

[DiffSynth-Studio](https://github.com/modelscope/DiffSynth-Studio) requires Rust and Cargo to compile extensions. You can install them using the following command:
```shell
curl --proto '=https' --tlsv1.2 -sSf [https://sh.rustup.rs](https://sh.rustup.rs/) | sh
. "$HOME/.cargo/env"
```

Install [DiffSynth-Studio](https://github.com/modelscope/DiffSynth-Studio):
```shell
git clone https://github.com/KwaiVGI/SynCamMaster.git
cd SynCamMaster
pip install -e .
```

Step 2: Download the pretrained checkpoints
1. Download the pre-trained Wan2.1 models

```shell
cd SynCamMaster
python download_wan2.1.py
```
2. Download the pre-trained SynCamMaster checkpoint

Please download from [huggingface](https://huggingface.co/KwaiVGI/SynCamMaster-Wan2.1/blob/main/step20000.ckpt) and place it in ```models/SynCamMaster/checkpoints```.

Step 3: Test the example videos
```shell
python inference_syncammaster.py --cam_type "az"
```

We provide several preset camera types. Additionally, you can generate new camera poses for testing.

### Training

Step 1: Set up the environment

```shell
pip install lightning pandas websockets
```

Step 2: Prepare the training dataset

1. Download the [SynCamVideo dataset](https://huggingface.co/datasets/KwaiVGI/SynCamVideo-Dataset).

2. Extract VAE features

```shell
CUDA_VISIBLE_DEVICES="0,1,2,3,4,5,6,7" python train_syncammaster.py   --task data_process   --dataset_path path/to/the/SynCamVideo/Dataset   --output_path ./models   --text_encoder_path "models/Wan-AI/Wan2.1-T2V-1.3B/models_t5_umt5-xxl-enc-bf16.pth"   --vae_path "models/Wan-AI/Wan2.1-T2V-1.3B/Wan2.1_VAE.pth"   --tiled   --num_frames 81   --height 480   --width 832 --dataloader_num_workers 2
```

3. Generate Captions for Each Video

You can use video caption tools like [LLaVA](https://github.com/haotian-liu/LLaVA) to generate captions for each video and store them in the ```metadata.csv``` file.

4. Calculate the available sample list

```shell
python generate_sample_list.py
```

Step 3: Training
```shell
CUDA_VISIBLE_DEVICES="0,1,2,3,4,5,6,7" python train_syncammaster.py   --task train   --output_path ./models/train   --dit_path "models/Wan-AI/Wan2.1-T2V-1.3B/diffusion_pytorch_model.safetensors"   --steps_per_epoch 8000   --max_epochs 100   --learning_rate 1e-4   --accumulate_grad_batches 1   --use_gradient_checkpointing  --dataloader_num_workers 4
```
We do not explore the optimal set of hyper-parameters and train with a batch size of 1 on each GPU. You may achieve better model performance by adjusting hyper-parameters such as the learning rate and increasing the batch size.

Step 4: Test the model

```shell
python inference_syncammaster.py --cam_type "az" --ckpt_path path/to/the/checkpoint
```

## 📷 Dataset: SynCamVideo Dataset
### 1. Dataset Introduction

**TL;DR:** The SynCamVideo Dataset is a multi-camera synchronized video dataset rendered using Unreal Engine 5. It includes synchronized multi-camera videos and their corresponding camera poses. The SynCamVideo Dataset can be valuable in fields such as camera-controlled video generation, synchronized video production, and 3D/4D reconstruction. The camera is stationary in the SynCamVideo Dataset. If you require footage with moving cameras rather than stationary ones, please explore our [MultiCamVideo](https://huggingface.co/datasets/KwaiVGI/MultiCamVideo-Dataset) Dataset.

https://github.com/user-attachments/assets/b49fc632-d1df-49fd-93d2-8513fbdb9377

The SynCamVideo Dataset is a multi-camera synchronized video dataset rendered using Unreal Engine 5. It includes synchronized multi-camera videos and their corresponding camera poses.
It consists of 3.4K different dynamic scenes, each captured by 10 cameras, resulting in a total of 34K videos. Each dynamic scene is composed of four elements: {3D environment, character, animation, camera}. Specifically, we use animation to drive the character, 
and position the animated character within the 3D environment. Then, Time-synchronized cameras are set up to render the multi-camera video data.
<p align="center">
  <img src="https://github.com/user-attachments/assets/107c9607-e99b-4493-b715-3e194fcb3933" alt="Example Image" width="70%">
</p>

**3D Environment:** We collect 37 high-quality 3D environments assets from [Fab](https://www.fab.com). To minimize the domain gap between rendered data and real-world videos, we primarily select visually realistic 3D scenes, while choosing a few stylized or surreal 3D scenes as a supplement. To ensure data diversity, the selected scenes cover a variety of indoor and outdoor settings, such as city streets, shopping malls, cafes, office rooms, and the countryside.

**Character:** We collect 66 different human 3D models as characters from [Fab](https://www.fab.com) and [Mixamo](https://www.mixamo.com).

**Animation:** We collect 93 different animations from [Fab](https://www.fab.com) and [Mixamo](https://www.mixamo.com), including common actions such as waving, dancing, and cheering. We use these animations to drive the collected characters and create diverse datasets through various combinations.

**Camera:** To enhance the diversity of the dataset, each camera is randomly sampled on a hemispherical surface centered around the character.

### 2. Statistics and Configurations

Dataset Statistics:

| Number of Dynamic Scenes | Camera per Scene | Total Videos |
|:------------------------:|:----------------:|:------------:|
| 3400                   | 10               | 34,000      |

Video Configurations:

| Resolution  | Frame Number | FPS                      |
|:-----------:|:------------:|:------------------------:|
| 1280x1280   | 81           | 15                       |

Note: You can use 'center crop' to adjust the video's aspect ratio to fit your video generation model, such as 16:9, 9:16, 4:3, or 3:4.

Camera Configurations:

| Focal Length            | Aperture           | Sensor Height | Sensor Width |
|:-----------------------:|:------------------:|:-------------:|:------------:|
| 24mm  | 5.0     | 23.76mm       | 23.76mm      |



### 3. File Structure
```
SynCamVideo-Dataset
├── train
│   └── f24_aperture5
│       ├── scene1    # one dynamic scene
│       │   ├── videos
│       │   │   ├── cam01.mp4    # synchronized 81-frame videos at 1280x1280 resolution
│       │   │   ├── cam02.mp4
│       │   │   ├── ...
│       │   │   └── cam10.mp4
│       │   └── cameras
│       │       └── camera_extrinsics.json    # 81-frame camera extrinsics of the 10 cameras 
│       ├── ...
│       └── scene3400
└── val
    └── basic
        ├── videos
        │   ├── cam01.mp4    # example videos corresponding to the validation cameras
        │   ├── cam02.mp4
        │   ├── ...
        │   └── cam10.mp4
        └── cameras
            └── camera_extrinsics.json    # 10 cameras for validation
```

### 3. Useful scripts
- Data Extraction
```bash
tar -xzvf SynCamVideo-Dataset.tar.gz
```
- Camera Visualization
```python
python vis_cam.py
```

The visualization script is modified from [CameraCtrl](https://github.com/hehao13/CameraCtrl/blob/main/tools/visualize_trajectory.py), thanks to their inspiring work.

<p align="center">
  <img src="https://github.com/user-attachments/assets/2a4e4063-9868-4b7f-8626-1e6d6f611e3f" alt="Example Image" width="40%">
</p>

## 🤗 Awesome Related Works
Feel free to explore these outstanding related works, including but not limited to:

[GCD](https://gcd.cs.columbia.edu/): synthesize large-angle novel viewpoints of 4D dynamic scenes from a monocular video.

[CVD](https://collaborativevideodiffusion.github.io): multi-view video generation with multiple camera trajectories.

[SV4D](https://sv4d.github.io): multi-view consistent dynamic 3D content generation.

Additionally, check out our "MasterFamily" projects:

[ReCamMaster](https://jianhongbai.github.io/ReCamMaster/): re-capture in-the-wild videos with novel camera trajectories.

[3DTrajMaster](http://fuxiao0719.github.io/projects/3dtrajmaster): control multiple entity motions in 3D space (6DoF) for text-to-video generation.

[StyleMaster](https://zixuan-ye.github.io/stylemaster/): enable artistic video generation and translation with reference style image.


## Acknowledgments
We thank Jinwen Cao, Yisong Guo, Haowen Ji, Jichao Wang, and Yi Wang from Kuaishou Technology for their invaluable help in constructing the SynCamVideo-Dataset. We thank [Guanjun Wu](https://guanjunwu.github.io/) and Jiangnan Ye for their help on running 4DGS.

## 🌟 Citation

Please leave us a star 🌟 and cite our paper if you find our work helpful.
```
@article{bai2024syncammaster,
  title={SynCamMaster: Synchronizing Multi-Camera Video Generation from Diverse Viewpoints},
  author={Bai, Jianhong and Xia, Menghan and Wang, Xintao and Yuan, Ziyang and Fu, Xiao and Liu, Zuozhu and Hu, Haoji and Wan, Pengfei and Zhang, Di},
  journal={arXiv preprint arXiv:2412.07760},
  year={2024}
}
```
