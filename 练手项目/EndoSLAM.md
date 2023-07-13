[toc]

---

# 我要干啥
复现EndoSLAM方法，在c3vd数据集上跑训练

## 该咋整
先瞅瞅整个项目是干啥工作地，人家给了服务器，试着在服务器上面先部署试试

然后这个数据集不是很清楚，到时候可以问下**宋师兄**

# EndoSLAM是啥

## 神奇海螺
该项目的目标是利用胶囊内的摄像头和传感器数据，实现对人体内部环境的三维重建和定位。SLAM技术是一种基于传感器数据的自主定位和地图构建方法，可以应用于无人机、机器人等领域。EndoSLAM则专注于胶囊内视角下的SLAM问题。

该项目的主要功能包括：

视觉定位：通过分析摄像头图像数据，识别关键特征点并进行特征匹配，从而实现定位功能。这样可以确定胶囊在人体内的位置和姿态信息。

建图：根据摄像头图像序列和传感器数据，构建人体内部环境的三维模型。通过将多个图像帧进行融合，生成高质量的内部环境地图。

轨迹回放：提供轨迹回放功能，可以可视化显示胶囊在人体内移动的路径。这对于后续的医学分析和研究非常有用。

该项目使用的主要技术包括计算机视觉、图像处理、传感器融合和SLAM算法等。开发者可以通过GitHub页面获取源代码、文档和示例数据等资源，以便进行进一步的研究和开发。

总之，EndoSLAM是一个旨在实现胶囊内视觉定位与建图的开源项目，为医学领域提供了重要的技术支持。

## 扎不多得了
### EndoSLAM
endo指的是endoscopy，内窥镜的意思。

SLAM是同时定位与地图构建（英语：Simultaneous localization and mapping，简称SLAM）是一种概念：希望机器人从未知环境的未知地点出发，在运动过程中通过重复观测到的地图特征（比如，墙角，柱子等）定位自身位置和姿态，再根据自身位置增量式的构建地图，从而达到同时定位和地图构建的目的。

所以说二者和一块儿大概是指内窥镜同时定位与地图构建

顺带一提，这里的capsule指的应该是[胶囊内窥镜](https://en.wikipedia.org/wiki/Capsule_endoscopy)，不是胶囊网络

### 杂谈
看下来感觉还行，甚至给了如何实现以及前置安装的内容，还算是比较的贴心，接下来就是该折腾这些乱七八糟的东西了


# 开整！day1
说实话一路搞来问题不是很少，目前的内容还是第一天，只简单地折腾了一下环境以及其他，下面说一下遇到的问题

## 下不下来
不清楚是什么情况，ping是能ping通国内网络的，但是外网是没有办法的。

一开始报错`recv erro -110`后来报错`fail connect to github 430`，这个服务器没给人管理员权限，没有办法下载东西，所以最后采用的是笨办法，电脑上下好了然后传过去

给条指令参考下`scp /home/heiwater/Downloads/EndoSLAM-master.zip zsy@192.168.151.157:/home/zsy/EndoSLAM`

这是上传，下载的话反过来就可以了，如果是上传/下载整个目录在scp后面加个`-r`就可以了

## 没有环境
没有初始化conda环境，一开始连接到了服务器上面的时候连python环境都没有，在宋学长的帮助下成功的整出了conda环境

没有记错是`cd`到了`opt`目录下`conda init`，初始化了conda环境

### vscode上ssh
一开始只是想单纯的用terminal上面直接试着跑得，但是说实话纯控制台纯代码操控的话属实有点难受，而且文件看得也不是很直观，还是选择了vscode

这里用的是微软的remote ssh，配置什么的非常的简单，就是输入地址然后每次登录的时候输入下密码就可以了

硬要说是什么问题的话就是一开始刚下好的事vscode没有加载出来，导致以为自己配置的出了问题。估计是安装好之后需要等待vscode自动配置好相关的文件，需要多等一会儿，自己多次重启vscode后解决。

## 数据集缺少camera calibration parameters
这个camera calibration parameters大概就是摄像机初始化参数，搜了下[教程](https://www.mathworks.com/help/vision/ug/camera-calibration.html)看到其实就是一个矩阵。

用这个矩阵来处理每一个摄像头的镜头畸变，保证每一张图片都是**正常的**。这个矩阵的内容如下：

$$\begin{bmatrix}
f_x&s&c_x\\
0&f_y&c_y\\
0&0&1\\
\end{bmatrix}$$

很幸运的，c3vd数据集给出了相应的初始化信息，在`intrinsics.txt`文件内，并且是以矩阵形式给出来的。现在需要处理得就是将原有的矩阵改成EndoSLAM所要求的`cam.txt`文件了

cam.txt给出的文件内容如下：
```
    fx: 156.0418
    fy: 155.7529
    cx: 178.560
    cy: 181.8043
    skew: 0
    k1: -0.2486
    k2: 0.0614
```

这个skew可以推测为s，然而这两个k1,k2不清楚是什么东西，想着看看mathworks的教程看看

查阅后发现这个k1 k2是镜头的畸变系数，但是现在怎么计算出来就比较的尴尬了

## 畸变系数k1&k2
看人家的介绍，两个畸变系数就可以表示出来，但是不清楚是不是可以直接标记成0，这里又一个[参考](https://blog.csdn.net/Jeff_zjf/article/details/118579649)，人家好像只是给了一个相机内参矩阵，别的都填的0，不知道填0有没有影响，明天试试。

# day2
说实话在开始跑之前还没有搞明白C3VD的数据集组成是什么，所以想着在搞之前先了解下这个数据集的组成，至少知道这些sigmod之类的对应的什么先

## C3VD数据集组成
找到了数据集的[介绍](https://durrlab.github.io/C3VD/)

沟通了一下，发现只用随便跑一个练练手就可以了，喜！

## 数据集地址

### 训练
`/home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/datasets/C3VD/cecum_t1_a_under_review`
训练集一共175个图片

### 测试
`/home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/datasets/C3VD_Test/trans_t2_b_under_review`
测试集一共105个图片

### 修改
看了下人家的要求，最后改成了如下内容`/home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/data`

```bash
Data_Path                # DIR_TO_TRAIN_DATASET
 ├──  train_dataset
 |      ├── cam.txt      #camera calibration parameters
 |      ├── 1.jpg
 |      ├── 2.jpg
 |      ├── ...
 ├──  validation_dataset     
 |      ├── cam.txt      #camera calibration parameters
 |      ├── 1.jpg
 |      ├── 2.jpg
 |      ├── ...
 ├──  train.txt #including the folder names for training dataset
 └──  val.txt   #including the folder names for validation dataset

```

## 开整

### 训练的指令
`CUDA_VISIBLE_DEVICES=0 python train.py /home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/data --name cecum_t1_a_under_review`

---

这个指令没有明确输出，下面的那个`DIR_TO_TEST_DATASET`应该是给你想输出的目标文件夹里面的，用来存放训练好的模型（大概）
~~`CUDA_VISIBLE_DEVICES=0 python train.py /home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/data --name cecum_t1_a_under_review`~~

### 测试的指令
`python test_vo.py  --pretrained-posenet DIR_TO_PRETRAINED_MODEL --dataset-dir DIR_TO_TEST_DATASET --output-dir DIR_TO_RESULTS`

## FileNotFoundError: /home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/data/cecum_t1_a_under_revie/cam.txt not found.
大概说的是找不到cam.txt这个文件，但是很显然的我已经把文件放进去了，然并卵= =

一开始以为是我的格式不对，试过了直接3*3矩阵中间空格输入，不管用
fx，fy这样的格式不管用
1，2，3，这样的格式也不管用

这就很气人。
### AC
你妈的，最后在`train.txt`文件里面打了个回车，成了，🤡:point_left:

很有趣的在`terminal`里面敲cat指令的时候如果没有加上回车，缩放窗口会有无数个@

### 写个loader

至少现在不是很需要用
~~上面的问题依旧没有解决，聊了下可以考虑写一个loader，来吧数据读进去~~

## ValueError: cannot reshape array of size 3 into shape (3,3)
这玩意应该说的是我那个`cam.txt`里面的格式不是很对，我试着改改先，改之前先贴上最开始的格式

### 最初版
```bash
7.6803816042173048e+02,0,6.8274332390281131e+02,
0,7.6803998896289613e+02,5.4107062189054227e+02,
0,0,1

```

### 最终版
```bash
7.6803816042173048e+02 0 6.8274332390281131e+02 
0 7.6803998896289613e+02 5.4107062189054227e+02 
0 0 1
```

所以你就说这玩意讨厌不，让你给又没告诉你格式，但是人家报错的时候确实告诉了你要有个shape，空格隔开属实咩有毛病

## FileNotFoundError: [Errno 2] No such file or directory: Path('/home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/data/val.txt')
这个是缺少测试集？

就是单纯的忘创建一个`val.txt`了，创建完了记得打上回车就是了

## ValueError: num_samples should be a positive integer value, but got num_samples=0
看样子应该是没有加载进去数据集，看着前面的报错提示有说

```bash
=> will save everything to checkpoints/cecum_t1_a_under_review/07-12-17:15
=> fetching scenes in '/home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/data'
0 samples found in 1 train scenes
0 samples found in 1 valid scenes
Traceback (most recent call last):
```

# day3
挺好的，现在要看的东西又变多了，没学过python是这样的，问题不大，都是好事儿

## 修改代码
现在遇到的问题是代码没有对相应的数据集做适配，有两个说法：

 1. 偷懒，直接按照人家需要的格式稍微改改看看这东西能不能直接跑起来
 2. 老老实实改代码，改出来

现在先老老实实改改代码吧，人家师兄专门跑过来给你瞅的东西，不能白瞎人家的好意不是

## 