# Deep learning

### U-Net: Ronneberger2015MICCAI

>  架构类文章
>  Ronneberger O., et al. U-Net: Convolutional Networks for Biomedical Image Segmentation

#### 面临问题：

the success of CNN was limited due to the size of the **available training sets** and the size of the **considered networks.**

现有CNN的性能受限于数据集的大小以及网络架构的参数量

In many visual tasks, especially in biomedical image processing, the desired output should include localization, i.e., a class label is supposed to be assigned to each pixel.

对于除分类以外的视觉任务，如医学图像处理的定位任务，需要对于图片中的每个像素生成对应的标签。

a network in a sliding-window setup to predict the class label of each pixel by providing a local region (patch) around that pixel ... has two drawbacks.

已有在ISBI数据集上成功的方法仍然存在两个主要弊端：

1. 由于使用滑窗方法来预测像素的label，因此运算过程会有很多重复部分，数据冗余造成运行缓慢；
2. 在上下文使用和定位精度需要进行取舍

#### 解决方法：

- 采用了全卷积网络 fully convolutional network
- 使用 upsampling 取代pooling操作
- 并且结合contracting path，将下采样阶段的feature map与上采样后的进行维度上的拼接，以提高定位精度
- 并且使用 Overlap-tile strategy 用镜像样本，来用于后续分割
- 使用基于“弹性变形”的数据增广方法来生成训练样本 use excessive data augmentation by applying elastic deformations to the available training images
- 使用带有权重的损失函数，来进行相似物体的分离

---

- 训练阶段
  - 使用尽可能大的输入，因此将batch减少至一张图片 (favor large input tiles over a large batch size and hence reduce the batch to a single image)，并且使用momentum=0.99，用过去样本决定未来更新方向
  - 能量方程 = a pixel-wise soft-max + the cross entropy loss function
  - 预先计算权重图 weight map = 类出现频率 + 当前像素与最近两个细胞的靠近程度决定（靠近两细胞边界越近，权重越大）
  - 使用与层数和卷积核大小相关的参变量生成用于初始化的高斯分布 drawing the initial weights from a Gaussian distribution 
- 数据增广
  - 使用随机向量在3x3的网格内进行平滑变形操作，然后结合插值生成新样本 smooth deformations using random displacement vectors

#### 实验：

We demonstrate the application of the u-net to three different segmentation tasks, EM segmentation challenge, ISBI cell tracking challenge 2014 and 2015.

主要在三个数据集上进行测试，在速度和精度都取得很大进步。

---

### 3D U-Net: C¸ i¸cek2016MICCAI

> 架构类文章
> C¸ i¸cek O., et al. 3D U-Net: Learning Dense Volumetric Segmentation from Sparse Annotation

#### 面临问题：

- Volumetric data is abundant in biomedical data analysis. Annotation of such data with segmentation labels causes difficulty
- 在生物医疗图像中，体素信息非常丰富，但是标注异常困难。因此使用完全标注的三维图像得到训练数据，非常不可行。

#### 解决方法：

- 使用少量分割数据，来生成整体稠密的分割结果
- 使用完全新颖的3D架构，3维信息作为输入，然后中间设计的操作也都是三维 the network proposed in this paper takes 3D volumes as input and processes them with corresponding 3D operations, in particular, 3D convolutions, 3D max pooling, and 3D up-convolutional layers.
- 与U-Net不同的是，在3dmaxpooling丢失信息前的3dconv先进行维度扩增（扩增回路则是使用反操作，先压缩维度卷积再3dmaxpooling），来实现avoid bottlenecks in the network architecture 
- 使用BN层来加速训练时的收敛 se batch normalization [4] for faster convergence
- 同样使用特别设计的 weighted loss function 
- 结合特殊数据增广形式 （apply a smooth dense deformation field on both data and ground truth labels），然后相关数据使用的插值方法为：B-spline interpolation

#### 实验：

最后实验部分，分别进行了Semi-Automated Segmentation半标注分割，通过进行少数的标注，实现整个三维结构的分割；同时也进行了Full-Automated Segmentation全自动分割，通过再小数据集上训练，以实现在大数据集上的泛化。