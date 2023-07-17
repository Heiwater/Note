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

> 其实不用，用人家数据集自带的`intrinsics.txt`改个名就对了，就只要输入个33矩阵，问题不大

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

> 后来发现完全没有必要用这个东西，就只要了个初始化参数

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

> 现在的情况是，整个改一改代码qwq，问题不大，都是好事儿

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

## 代码结构

### 梳理之前
其实很好奇，现在在试着复现的东西什么个原理，也是计算机视觉，根据视觉训练出来的东西来确定现在的内窥镜在什么位置是么？没有看论文，不清楚是什么个原理，看着样子是根据内窥镜的图片输入，输出一个运动轨迹，和以前的识别不是很一样就是了

## 初始化修改
这里就是想要修改相应的初始化内容，以求能够用于c3vd数据集

```python
    def __init__(self, root, seed=None, train=True, sequence_length=3, transform=None, skip_frames=1, dataset='kitti'):
        np.random.seed(seed)
        random.seed(seed)
        self.root = Path(root)
        # [TODO] remove
        scene_list_path = self.root/'train.txt' if train else self.root/'val.txt'
        # [TODO] modify this to listdir
        self.scenes = [self.root/folder[:-1] for folder in open(scene_list_path)]
        # [TODO] modify the sequence_length to the length of self.scenes
        self.transform = transform
        self.dataset = dataset
        self.k = skip_frames
        self.crawl_folders(sequence_length)

```

### scene_list_path
这里就是确定你训练集或者是测试集地方的，原本是根据你的`train.txt`文件内的内容来确定每一个训练的内容，现在想改成直接读取当前目录下的所有文件

这一段看上去应该是确定了你的训练的子集的列表位置，让后开始读？

#### [:-1]是啥
菜鸟教程给了个例子：
```python
    d=a[:-1]  #从位置0到位置-1之前的数
    print(d)  #pytho
```

结合自己之前看到的来说，这个可能就是为什么要加回车的原因了，相当于你的文件位置是`/dataset/DIR`，但是由于你用的是`txt`格式的文件存储的，所以说你需要在文件结束后敲上一个回车，这样你`self.root/folder[:-1]`输出的就是正常的不带回车的。

或者是删除`\0`之类的？大概是这个意思

#### transform
transform改不改还另说

#### 要干啥
现在想的是用python实现： 读取当前文件下的文件列表以及`intrinsic.txt`

现在这一段的内容就是实现咱说的事情的，但是相较之下还有些事情没有做:
```python
    self.transform = transform
    self.dataset = dataset
    self.k = skip_frames
    self.crawl_folders(sequence_length)
```

这些都是**超参数？**还是什么东西，看着这些都在开头的定义里面有给出：
```python
def __init__(self, root, seed=None, train=True, sequence_length=3, transform=None, skip_frames=1,dataset='kitti'):
```

所以推测默认的是给`kitti`数据集使用的，所以需要改改么？

~~transform，这些里面感觉就只有最后一个是要看看得，所以现在先考虑下如何用python实现文件列表的读取吧~~
> 其实最后一个是调用的下面的函数，所以鼠标以后可以没事儿多点点，能弹出来高亮的 = =

#### 文件列表读取
靠chatgpt写了个读取当前目录下的文件的程序，还不错

```python
import os

def get_subdirectories(folder_path):
    subdirectories = []
    for dirpath, dirnames, filenames in os.walk(folder_path):
        for dirname in dirnames:
            subdirectories.append(dirname)
    return subdirectories

folder_path = '/path/to/your/folder'  # 替换为你想要读取的文件夹路径
output_file = 'output.txt'  # 输出文件名

subdirectories = get_subdirectories(folder_path)

with open(output_file, 'w') as file:
    for subdir in subdirectories:
        file.write(subdir + '\n')
```

其中主要的是这个`os.walk`，里面包含了三个`list`分别是:`dirpath`、`dirnames`、`filenames`，稍微改了下，放到了`/home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/datasets/C3VD`下面的`test.py`里面了

小编也是第一次知道捏，那么剩下的就是添加一个小功能，统计下当前文件夹下面的子文件的个数就可以了

## 感觉能整了
EndoSLAM！启动！

### 寄
现在能读进去图片了，但是又有新的问题出现了`AttributeError: module 'torchvision.models.resnet' has no attribute 'model_urls'`

说是应该是版本不对，或者是因为更新到了新的版本没有相应的东西了，所以说该找找**相应的版本了**，所以说现在的问题是用了错误了`torchvision`版本

# day4
天道好轮回，环境饶过谁。

## 试试版本
昨天基本上已经锁定了问题是因为版本不对，新版的`torchvision`删掉了一些api，然后人家项目给的是`pytorch>=1.5.1`，找了个[教程](https://blog.csdn.net/jorg_zhao/article/details/106883420)，当然不是广义上的教程，只是单纯的告诉你对应的版本，还挺不错的。

试了试`conda install pytorch==1.5.1 torchvision==0.6.1 cudatoolkit=10.2 -c pytorch`，然而不是很管用的样子。

搜了下另一个[教程](https://ptorch.com/news/198.html)，好像有新的说法

`conda install pytorch=1.5.1 -c soumith`

但是后面的`-c soumith`不清楚是什么情况，`-c`是指的channal，所以说后面的soumith就是相应的通道的名字了，那么我需要执行以下的指令：
 1. `conda install pytorch=1.5.1 -c soumith`
 2. `conda install torchvision=0.6.1 -c soumith`
 3. `conda install cudatoolkit=10.2 -c soumith`

还是不行，所以说是不是可能是因为python版本过高的缘故？现在的python版本是`3.10`，人家推荐的是`>=3.6`，所以说有没有说法自己新建一个conda环境？

但是转念一想模板库好像不是那么容易更新的吧？所以说是不是可能是代码在调用木板地时候出了问题？

看得人家的代码里面用的好像就是18层的resnet，在`train.py line157`说了是`pose_net = models.PoseResNet(18, args.with_pretrain).to(device)`

感觉不像，可能就是环境的问题，下面试着折腾下conda

## 试试[conda](https://blog.csdn.net/SARACH_WONG/article/details/89328307)

### 查看conda现存版本
`conda info --envs`

### 好像有[bug](https://github.com/conda/conda/issues/10969)
说是好像两位是版本的python在conda上的支持不是很好，现在每一次试着创建低版本的环境竟会报错Translate

# Day5
有个靠谱的学长真不错

## 解决
之前困扰的问题，只需要稍微改一下就可以：
```python
    if pretrained:
        # new
        if num_layers == 18:
            from torchvision.models import ResNet18_Weights
            pretraine_weights = ResNet18_Weights.IMAGENET1K_V1
        if num_layers == 50:
            from torchvision.models import ResNet50_Weights
            pretraine_weights = ResNet50_Weights.IMAGENET1K_V1
        # the old version:
        # loaded = model_zoo.load_url(models.resnet.model_urls['resnet{}'.format(num_layers)])
        loaded = model_zoo.load_url(pretraine_weights.url)
        loaded['conv1.weight'] = torch.cat(
            [loaded['conv1.weight']] * num_input_images, 1) / num_input_images
        model.load_state_dict(loaded)
    return model
```
### 为啥
版本更新了，不能用旧的`model_urls`这个功能了，所以改成了根据不同的情况引用不同的预训练权重。

所以有的时候没有必要完完全全的复现原作者当时所使用的环境，根据现在的更新的开发者文档来有针对性的更新程序本身可能不失为一件更有意义的事情

## RuntimeError: Given groups=1, weight of size [64, 3, 7, 7], expected input[4, 1080, 1350, 3] to have 3 channels, but got 1080 channels instead got 1080 channels instead

现在的问题大概率是输入的有问题，所以要修改的也是在`train`文件内，但是看不懂这件事情就比较的尴尬

> 最后在学长精湛的操作和咱一旁吃瓜注视下基本完美的解决了，顺带发现自己还有好多东西基本上根本没有接触过，不过都是好事儿，至少自己有一段时间有很多东西可以折腾了

这个是因为自己没有整理好输入的图片内容，以及相应的输入的格式，EndoSLAM所有的输入基本上都在`sequence_floder.py`，添加了`center_crop`这个类来实现中心剪裁以及深度值重新缩放。

添加的内容如下：

### center_crop
这里主要是实现中心剪裁以及深度值的重新缩放

```python
def img_center_crop(img, size):
    """
    Center crop a PIL image
    Args:
        img: a PIL image
        size: a list contain the width and height , eg. [1024, 2048]
    Return:
        Center croped PIL image
    """
    width, height = img.size
    new_height, new_width = size
    left = (width - new_width)/2
    top = (height - new_height)/2
    right = (width + new_width)/2
    bottom = (height + new_height)/2
    return img.crop((left, top, right, bottom))

def c3vd_depth_rescale(depth):
    """
    Linearly rescale the depthmap of c3vd dataset
    Args:
        depth:  a NUMPY ARRAY, PIL image MUST be converted to numpy array before processing
                use np.array(img) to convert a PIL image to numpy array
                where img is a PIL object.
    Return:
        depth:  a depth map in which depth ranged from 0 to 100
    """
    scale_factor = scale_factor = 100 / 65535
    depth = depth * scale_factor
    # [H, W] to [B, H, W]
    depth = depth.np.expand_dims(depth, axis=-1)
    return depth
```

### getitem
这里是读进来数据的部分，然后顺理成章的就在这里讲读入的数据剪裁成合适的大小，并且修改读入的数据以求符合模型

```python
def __getitem__(self, index):
        sample = self.samples[index]
        tgt_img = load_as_float(sample['tgt'])
        ref_imgs = [load_as_float(ref_img) for ref_img in sample['ref_imgs']]

        # preprocess for c3vd
        img_size = [1056, 1344]
        tgt_img = img_center_crop(tgt_img, img_size)
        ref_imgs = [img_center_crop(ref_img, img_size) for ref_img in ref_imgs]

        # convert to numpy array
        # [H, W, C] to [B, C, H, W]
        tgt_img = np.array(tgt_img, dtype=np.float32).transpose(2, 1, 0)
        ref_imgs = [np.array(ref_img, dtype=np.float32).transpose(2, 1, 0) for ref_img in ref_imgs]

        if self.transform is not None:
            imgs, intrinsics = self.transform([tgt_img] + ref_imgs, np.copy(sample['intrinsics']))
            tgt_img = imgs[0]
            ref_imgs = imgs[1:]
        else:
            intrinsics = np.copy(sample['intrinsics'])
        return tgt_img, ref_imgs, intrinsics, np.linalg.inv(intrinsics)
```

## 炫酷操作
这里是炫酷的debug环节，只能说十分地好用，可以在`debug console`里面查看相应的参数，说实话自己以前的debug用过，但是用的不多，这次人家的操作属实是小刀剌屁股，开了眼了。

里面的args，就是代替原本输入的参数，即文件所在的位置，输出的地方，这个`-b`是`batchsize`大小，因为文件太大了，现存不是很够，所以只能调小一次性扔进去的数据大小。

`env`这里是修改运行的显卡，`0`号卡学长在用，所以只能修改到`1`号卡上面来试着跑一下。

`program`这里是这个EndoSLAM的训练的主程序，这样子写相当于只调试这一个程序的内容，但是由于会调用其他程序所以说问题不大，至少这样子不用像常见的通用的调试那样调到相应的程序在点击调试运行就是了。

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "train",
            "type": "python",
            "request": "launch",
            "program": "EndoSfMLearner/train.py",
            "args": [
                "/home/zsy/EndoSLAM/EndoSLAM-master/EndoSfMLearner/data",
                "--name", "cecum_t1_a_under_review",
                "-b", "1"
            ],
            "console": "integratedTerminal",
            "env": {
                "CUDA_VISIBLE_DEVICES": "1"
            },
            "justMyCode": true
        }
    ]
}
```

## 跑起来力！盖瑞！
喜闻乐见的，跑起来了，然后就是慢慢的等着呗。