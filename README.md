Metal Surface Wear Prediction

基于东北大学钢材表面缺陷数据集的双阶段潜空间扩散模型

本项目实现了一个面向金属表面磨损演化预测的深度生成模型。模型参考了人脸衰老生成任务中的 Two-Pass Diffusion 思想，将“人脸年龄变化”中的时间演化过程迁移到金属表面纹理变化任务中。

项目采用 Autoencoder + Latent Diffusion + AdaNI + Two-Pass Refinement 的整体框架，在潜空间中学习金属表面纹理，并通过不同程度的噪声注入模拟表面随时间产生的变化。

注意： 当前实验使用的是从数据集中实际读取的 20 张图像，且没有真实的逐日磨损时间标签。因此，目前的实验结果更准确地描述为金属表面磨损演化生成/模拟，而不是经过真实时间序列数据验证的工业磨损寿命预测。

⸻

1. 项目简介

金属表面在长期使用过程中会产生划痕、凹坑、粗糙度变化以及纹理退化等现象。传统的图像分类方法通常只能判断当前表面缺陷类别，而无法进一步模拟：

“如果当前金属表面继续使用一段时间，它可能会变成什么样？”

本项目尝试利用潜空间扩散模型解决这一问题。

模型输入一张当前金属表面图像，并通过扩散过程生成未来状态，最终得到：

Original
   ↓
Day 1
   ↓
Day 2
   ↓
...
   ↓
Day 10

从而形成金属表面磨损的视觉演化过程。

⸻

2. 项目框架

整体模型由四个主要部分组成：

                Input Image
                     │
                     ▼
              ┌─────────────┐
              │ Autoencoder │
              │    Encoder  │
              └──────┬──────┘
                     │
                     ▼
              Latent Representation
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
   Texture Condition       Noise Injection
          │                     │
          └──────────┬──────────┘
                     ▼
              Latent U-Net
                     │
                     ▼
             Reverse Diffusion
                     │
                     ▼
              Pass 1 Generation
                     │
                     ▼
              Pass 2 Refinement
                     │
                     ▼
               Decoder
                     │
                     ▼
             Predicted Surface

核心模块

* Autoencoder：将 64×64 金属表面图像压缩到潜空间。
* Latent U-Net：在潜空间中预测扩散噪声。
* AdaNI：根据预测时间跨度动态决定噪声注入强度。
* Two-Pass Refinement：先进行较大幅度的变化，再使用低噪声进行细化。
* Interpolation：将最终预测结果生成 Day 1–Day 10 的连续变化过程。

⸻

3. 数据集

本项目使用东北大学钢材表面缺陷数据相关图像。

数据读取目录：

~/Downloads/Spot-Defect Images(SDI)

程序会递归搜索以下格式：

.bmp
.png
.jpg
.jpeg
.tif
.tiff

当前 Notebook 实际读取：

Total metal images: 20

图像统一转换为：
64 × 64
Grayscale
1 Channel
预处理代码：
transform = transforms.Compose([
    transforms.Resize((64, 64)),
    transforms.Grayscale(num_output_channels=1),
    transforms.ToTensor()
]
当前实验结果总结

项目	当前结果
输入尺寸	64 × 64
图像通道	Grayscale
实际读取图像	20
Latent Size	4 × 16 × 16
Diffusion Steps	200
Autoencoder Epochs	30
Diffusion Epochs	50
Autoencoder Final Loss	0.0015
LDM Loss (Epoch 10)	1.0027
LDM Loss (Epoch 50)	0.9401
LDM Loss下降	≈6.24%
Prediction Horizon	10 Days

⸻

局限性

当前版本主要存在以下限制：数据量较小
当前实验实际使用：
20 images
因此不能据此证明模型具有良好的泛化能力。
缺少真实时间标签
目前没有：
surface_day_0
surface_day_1
surface_day_2
...
形式的真实磨损数据。
时间变化规则是人为设计的
AdaNI 中：
3 days
7 days
是当前实验设定的工程规则，而不是通过数据学习得到的。
中间日期采用插值

Day 1–Day 10 并不是模型分别预测的 10 个真实时间状态，
进行图像空间插值得到。
5. 当前模型不是严格意义上的 VAE
虽然代码类名为：
SimpleVAE
但实际上没有 KL Loss 和概率潜变量建模。
⸻
19. 后续改进方向
为了将项目进一步发展成真正的金属表面磨损预测模型，下一步可以加入：
Temporal Dataset
构造：
(x_t, x_t+Δt)
训练样本。
例如：

Day 0 → Day 5
Day 5 → Day 10
Day 10 → Day 20

Continuous Time Conditioning

将：

Day 1
Day 2
...
Day 10

改为连续时间变量：
Δt = 1, 2, 3, ..., 10
使模型学习：
Wear = f(Image, Δt)
增加评价指标
未来可以加入：

MAE
RMSE
SSIM
PSNR
LPIPS

以及金属表面相关的：
纹理粗糙度
边缘变化
缺陷面积
灰度统计
频域特征

引入物理信息
进一步加入：
载荷
温度
压力
摩擦次数
使用时间
材料类型
使模型从单纯的图像生成进一步发展为：
数据驱动 + 物理约束的金属磨损预测模型

⸻

20. 项目定位

本项目的核心思想可以概括为：

金属表面图像
      ↓
Latent Representation
      ↓
Adaptive Noise Injection
      ↓
Latent Diffusion
      ↓
Two-Pass Refinement
      ↓
未来表面状态
项目重点探索的是：
如何利用潜空间扩散模型模拟金属表面纹理随时间发生的视觉演化。
当前版本属于一个研究型 Prototype，后续通过加入真实时间序列磨损数据，可以进一步发展为真正具有定量预测能力的金属磨损预测系统。
⸻

21. Reference

本项目的方法设计参考了人脸衰老生成任务中的 Two-Pass Diffusion 思想，并针对金属表面纹理变化进行了调整。

主要参考方向：

Two-Pass Diffusion
Adaptive Noise Injection
Latent Diffusion
Conditioned U-Net
Image Refinement

⸻

License

本项目仅用于学习、研究和实验目的。
如果用于工业生产环境，需要进一步使用真实磨损时间序列数据进行训练、验证和可靠性评估。
