# 神经网络和深度学习 期末作业

物体重建和新视图合成

### 基本要求：
1. **选取身边的物体拍摄多角度图片/视频**，并使用 COLMAP 估计相机参数，随后使用现成的框架进行训练。
2. **基于训练好的 NeRF 渲染环绕物体的视频，并在预留的测试图片上评价定量结果**。

本项目是在 [nerf-pytorch](https://github.com/yenchenlin/nerf-pytorch) 基础上完成的。

### 准备
请确保安装以下依赖：
- requirements.txt 中的库
- tensorboard

建议在Ubuntu 20.04 + Pytorch 1.10.1上运行

### 数据集准备
1. 对需要重建的物体进行环绕拍照，使用 COLMAP 进行参数估计，计算位姿。
（一般情况使用simple_pinhole进行特征点提取，再默认模式特征点匹配、重建即可；
对于镜头畸变效果强的数据可以考虑radical模式，对于视频抽帧形成的连续、编号连续图片集可以用sequential模式，但需要加载词汇树，实测没加载出来过）
2. 使用 [LLFF](https://github.com/Fyusion/LLFF) 中的 `imgs2poses.py` 脚本创建数据集。
3. 由于原本run_nerf.py中自带的图片缩放函数在运行中会无效（不降采样同时不报错，导致内存爆炸），因此使用 `change_size.py` 脚本进行手动下采样。


### 训练
1. 在 `/configs` 目录下新建 `my_scene.txt`，参数设置可以参考 `fern.txt`（参考示例图片，和自己图片最像的图对应config效果较好）。
2. 执行以下命令开始训练：
   ```sh
   python run_nerf.py --config configs/my_scene.txt --spherify --no_ndc
   ```

### 测试
1. 直接执行以下命令进行测试：
   ```sh
   python run_nerf.py --config configs/test.txt --render_only
   ```
2. 或者在全部训练完成的基础上执行以下命令进行测试，自动加载最后的检查点：
   ```sh
   python run_nerf.py --config configs/test.txt --spherify --no_ndc
   ```

## TensoRF

### 数据集准备
1. 可以使用NeRF训练中处理好的数据your_object/文件夹中包含images/、sparse/0/{cameras.txt,images.txt,points3D.txt}即可
2. 也可以安装colmap配备path后命令行执行
   ```sh
   colmap feature_extractor --database_path database.db --image_path ./images
   colmap exhaustive_matcher --database_path database.db
   colmap mapper --database_path database.db --image_path ./images --output_path ./sparse
   colmap image_undistorter --image_path ./images --input_path ./sparse/0 --output_path ./colmap_undistorted --output_type COLMAP
   ```

### 训练
环境配置：
   ```sh
   git clone https://github.com/apchenstu/TensoRF.git
   cd TensoRF
   conda create -n TensoRF python=3.8
   conda activate TensoRF
   pip install torch torchvision
   pip install tqdm scikit-image opencv-python configargparse lpips imageio-ffmpeg kornia tensorboard
   ```
生成 COLMAP → NeRF 格式
   ```sh
   python dataLoader/colmap2nerf.py --colmap_matcher exhaustive --run_colmap --aabb_scale 4
   ```

类似NeRF创建配置文件，进行训练

   ```sh
   python train.py --config configs/your_object.txt --expname your_object_TensoRF --basedir ./logs
   ```
有相机轨迹json文件可渲染
   ```sh
   python train.py --config configs/your_object.txt --ckpt ./logs/your_object_TensoRF/ckpt/latest.pth --render_only 1 --render_path 1
   ```

## 3DGS

数据集格式需要将“images”文件夹改为“input”文件夹

环境配置：
   ```sh
   # 克隆仓库（包含子模块）
   git clone https://github.com/graphdeco-inria/gaussian-splatting --recursive
   cd gaussian-splatting

   # 创建 Conda 环境
   conda env create -f environment.yml
   conda activate gaussian_splatting
   ```
### 生成 SfM 数据
   ```sh
   python convert.py -s <location> --resize
   ```

### 训练
   ```sh
   python train.py -s <location>  --model_path <output_dir> 
   ```
基本使用默认值即可，NeRF Synthetic 数据集可选white_background = True

模型保存：在 --save_iterations 指定的迭代保存 .ply 模型

### 渲染
   ```sh
   python render.py -m <output_dir>  
   python metrics.py -m <output_dir> 
   ```
