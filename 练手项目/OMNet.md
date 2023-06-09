[toc]
# 帮帮我哆啦A梦 ChatGPT
这篇文章介绍了一个名为OMNet的深度学习模型，用于对部分点云进行配准。传统的点云配准方法通常要求两个点云之间没有重叠区域，但在现实中，这种情况很少出现。因此，该论文提出了一种重叠掩模学习方法，帮助模型处理重叠的局部点云。OMNet使用了一个编码器-解码器结构，同时在编码和解码过程中学习生成重叠掩模。然后，将该重叠掩模用于特征匹配和空间变换。实验结果表明，OMNet比现有的点云配准方法更准确、更鲁棒，并且能够处理具有较大重叠区域的点云。

# 能不能说人话 Glossary

## Registration 配准

## overlapping masks 重叠掩模
好像官方的说法叫做重叠掩模。~~但是我一开始翻译成了重叠蒙版、重叠遮盖，后面看的乱的话就这样吧，鸽了= =~~

重叠蒙版指的是使用多层蒙版，每层覆盖被蒙版的一部分区域。使用重叠蒙版的目的是为了确保蒙版中没有空隙，并创造一个更精确的蒙版。在图像处理中，重叠蒙版可用于隔离图像的特定部分或从图像中去除不需要的元素。

## ICP 迭代最近点
迭代最接近点（ICP）是一种用于计算机视觉和机器人学的算法，用于对齐两个或多个点云。ICP的目标是找到最佳的转换，将一个点云映射到另一个点云上。

ICP的基本思想是迭代地最小化两个点云中相应点之间的平方距离之和。这是通过首先选择一组对应点，或可能匹配的点对来实现的。然后，根据这些对应关系计算出一个转换，将一个云中的点映射到另一个云中。该变换可以是一个刚体变换（平移和旋转）或一个仿生变换（缩放和剪切）。

一旦转换被计算出来，它就被应用到其中一个点云上，然后重复这个过程。在每次迭代中，根据转换后的点云中与另一个点云最接近的点选择新的对应关系。这个过程一直持续到收敛，当转换的变化低于某个阈值时，就会出现收敛。

ICP可用于计算机视觉和机器人学的各种任务，如三维扫描的注册、物体识别和跟踪以及机器人定位。

## Deep Closest Point (DCP) 深度最近点
深度最接近点（DCP）是一种用于计算机图形学和计算机视觉的方法，用于对齐两个三维形状。DCP的目标是在两个形状的点之间找到尽可能接近的对应关系，同时也使形状之间的整体距离最小化。

术语 "深度 "是指使用深度学习技术，如神经网络，来学习可用于计算对应关系的形状表示。DCP有许多应用，包括形状匹配、形状检索和三维重建。它被广泛用于机器人、虚拟现实和增强现实等领域。

## point-wise features 逐点特征
一般来说，逐点特征指的是以每个点为基础进行评估的东西，而不是在一组点上。在机器学习中，逐点特征通常是指为数据集中的每个单独的数据点计算的一组特征。

例如，假设我们有一个图片数据集，我们想把每张图片分类为是否包含一只狗。我们可能会提取点状特征，如图像中的边缘数量、像素的平均亮度、某些颜色的存在等等。这些特征中的每一个都将针对数据集中的每一张图片进行计算，这使得我们可以训练一个分类模型，该模型可以学习基于这些特征进行预测。

## Rigid Transformation Regression 刚性旋转回归

## concatenated features 串联特征
串联特征是指通过将两个或多个单独的特征串联成一个单一的特征。串联是将两个或多个字符串、数组或其他数据结构组合成一个实体的过程。在机器学习中，串联的特征经常被用来表示变量之间的复杂关系，并提高模型的预测性能。例如，如果你有两个特征 "年龄 "和 "性别"，你可以将它们串联成一个单一的特征 "age_gender"，以捕捉关于年龄和性别的联合信息。

## ablation studies 消融研究
在人工智能（尤其是機器學習）中，消融是去除AI系統的組件。消融研究通過刪除某些組件來調查AI系統的性能，以了解組件對整個系統的貢獻。

# 吹的牛逼 bright spot

 1. OMNet，一个迭代的网络
 2. overlapping masks，重叠蒙版，将原有的部分到部分的识别转换成形状的识别
 3. sample twitce，二次采样，减少过拟合的问题 
 4. 利用了重叠点云的几何知识，解决了非重叠区域的负面影响

## OMNet
OMNet 是一个端到端的迭代网络，通过由粗到细的方法实现3D刚性旋转的同时还能够保持对噪声和偏移的稳定性

## overlapping masks
通过重叠蒙版，*非重叠点的问题*在提取全局特征时被解决，并将整体的行为由*部分到部分的识别*转换到*形状的识别*。由于*全局特征*，使得*刚性变换回归*变得更加的容易且没有干扰。

## 采样
和*其他人*不同，我们的数据*来源和参考*独立的从CAD模型中进行随机采样，并且移除了*轴对称类别*

# related works

## Correspondence Matching based Methods 基于对应匹配的方法
常见的对应匹配的方法无非是重复下面两个工作：
 1. 建立*原点云*与*参考点云*之间的对应关系
 2. 计算对应关系间的*最小二乘刚性变换(least-squares rigid transformastion)*

然后拉踩其他的方法 （= =）

与他们不同，我们可以使用预测的*重叠区域*来聚合*全局特征*

## Global Feature based Methods 基于全局特征的方法
介绍了PointNetLK，虽然人家是先驱者，但是人忽略了*非重叠点的负面影响*，无法识别到*部分到部分的输入*。

## Partial-to-partial Registration Methods 部分到部分的识别方法
上来还是先拉踩（= =||），说虽然别人做了很多但是面临着相同的问题，他们只能用*稀疏点*。

但与之不同的是我们的方法可以利用到*全部重合点的信息*。

> 他真的很爱拉踩别人（= =||）

# 方法
下面基本上写的是文章插图2的各个详细的流程。~~但是由于数学字符敲起来属实难受，这里就不重新敲一遍了，在笔记的第三页开始第三章Method 段落里面都有注释，倒是候如有需要自己锻炼一波英语阅读，简言之问题不大，鸽了(QAQ)~~

## Global Feature Extraction 全局特征提取
利用一个多层感知的神经网络函数`f(·)`，求得输入的点云$\tilde{X^i}$和$Y$得到逐点特征值$f^{i}_{\tilde{X}}$和$f^{i}_{Y}$

## Overlapping Mask Prediction 重叠蒙版预测
在部分对部分的情况下，尤其是存在*噪音*的情况下，存在*非拟合区域*，该区域不但*对识别没有帮助*还会*干扰全局特征*。

我们使用*蒙版预测模型*以自动分割拟合区域，点云分割以单一点云为输入并结合*局部和全局认知(local and global knowledge)*

在每次迭代中*全局特征*$F^{i}_{\tilde{X}}$和$F^{i}_{Y}$与*每个点的特征*$f^{i}_{\tilde{X}}$和$f^{i}_{Y}$相连接返回*逐点特征值*。

## Rigid Transformation Regression 刚性旋转回归

## Transform Regression Loss 转换回归损失

# 实验

# 讨论

# 总结

# 项目部署
喜闻乐见的，最后还是用了colab进行项目的部署，这就引出了另一件事情，想要整一张能用的信用卡来购买个colab会员，不过那都是后话了，现在在折腾环境的部署

## 所需环境
这里还是插一句嘴，colab可以直接当成linux虚拟机用，但是还是逃不过喜闻乐见的配置环境的问题，OMNet提供了所必须要的系统环境，这里还是按照惯例列举下：

|名称|版本|
|:-:|:-:|
|Pytorch|>=1.5.0|
|coloredlogs|==15.0.1|
|h5py|==3.1.0|
|numpy|==1.19.5|
|pytorch3d|==0.3.0|
|scipy|==1.5.3|
|termcolor|==1.1.0|
|torch|==1.9.1|
|torchvision|==0.10.1|
|tqdm|==4.62.3|
|transforms3d|==0.3.1|

## 出现的问题

### numpy无法安装

好像colab自带numpy，网上看的都是直接`imprott numpy as np`

---
不清楚是不是colab的问题numpy现在是没有办法直接pip install的，下面还是贴上是什么原因：
```
Building wheels for collected packages: numpy
  error: subprocess-exited-with-error
  
  × Building wheel for numpy (pyproject.toml) did not run successfully.
  │ exit code: 1
  ╰─> See above for output.
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
  Building wheel for numpy (pyproject.toml) ... error
  ERROR: Failed building wheel for numpy
Failed to build numpy
ERROR: Could not build wheels for numpy, which is required to install pyproject.toml-based projects
Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
```

### scipy 无法安装

这个好像也有

---

```
Collecting scipy==1.5.3
  Downloading scipy-1.5.3.tar.gz (25.2 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 25.2/25.2 MB 64.2 MB/s eta 0:00:00
  error: subprocess-exited-with-error
  
  × pip subprocess to install build dependencies did not run successfully.
  │ exit code: 1
  ╰─> See above for output.
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
  Installing build dependencies ... error
error: subprocess-exited-with-error

× pip subprocess to install build dependencies did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.
Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
```


### data目录下缺少 shape_names.txt
看了下人家项目里面的issue，然后就解决了QAQ