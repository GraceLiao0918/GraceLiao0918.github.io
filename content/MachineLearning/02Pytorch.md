---
title: Pytorch Learning
tags:
  - Pytorch
---

# Pytorch Learning

>[PyTorch深度学习快速入门教程（绝对通俗易懂！）【小土堆】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hE411t7RN/?vd_source=33637faf50548461f8a85c799878c82d&spm_id_from=333.788.videopod.episodes)


前置环境看[[09EnvironmentSet]]

# pytorch概述

## Pytorch 两大法宝函数

-  dir()
- help()
### Pytorch 两大法宝函数实战

```
dir(torch)
dir(torch.cuda)
help(torch.cuda.is_available)    # 函数作为被传递的参数或者返回值时不需要加括号，在定义或被调用时需要加括号。
```

## pytorch加载数据

PyTorch中读取数据主要设计两个类：Dataset和Dataloader
1. Dataset：提供一种方式去获取数据及其label
    - 如何获取每一个数据及其label
    - 告诉我们总共有多少的数据
2. Dataloader：为后面的网络提供不同的数据形式

如何使用Dataset：

首先下载数据集，“蚂蚁蜜蜂数据集”下载地址：https://download.pytorch.org/tutorial/hymenoptera_data.zip

三种常见的数据组织方式：

1. 文件夹名是标签
2. 每个训练文件有对应的标签文件
3. 文件名是标签

Dataset类的使用主要来说就是继承+重写__getitem__方法和__add__方法

### DataSet类代码

1. 使用PIL库读取第一张图片。

输入图片路径是可以直接在路径字符串前加r将其变为原生字符串，从而规避转义字符的问题。
2. 重写Dataset类
同一个Dataset子类的实例可以用加号进行合并（视频里看起来是这样的，通过+运算将蚂蚁数据集和蜜蜂数据集合并起来了）

```python
from torch.utils.data import Dataset  
from PIL import Image  
  
import os  
  
class MyData(Dataset):  
    def __init__(self,root_dir,label_dir):  
        self.root_dir = root_dir  
        self.label_dir = label_dir  
        self.path=os.path.join(self.root_dir,self.label_dir)  
        self.img_path=os.listdir(self.path)  
  
  
  
    def __getitem__(self,idx):  
        img_name=self.img_path[idx]  
        img_item_path = os.path.join(self.root_dir,self.label_dir,img_name)  
        img = Image.open(img_item_path)  
        label=self.label_dir  
        return img,label  
  
    def __len__(self):  
        return len(self.img_path)  
  
root_dir="hymenoptera_data/train"  
ants_label_dir="ants"  
ants_dataset = MyData(root_dir,ants_label_dir)  
  
len(ants_dataset)  
# print(len(ants_dataset))  
  
# 数据可视  
img,label = ants_dataset[0]  
img.show()
```

## TensorBoard的使用

### TensorBoard的安装与使用：

如果没有安装TensorBoard首先需要在选择的虚拟环境中安装。

查看TensorBoard的事件文件：

- 首选需要cd到事件文件所在的**文件夹**目录，然后在命令行中输入：tensorboard --logdir=事件文件所在的文件夹名 --port=6007（默认打开6006端口）

- TensorBoard的页面打开后，如果事件文件发生变化，只需刷新浏览器页面即可。不同事件不要用一个tag。

### SummaryWriter类的使用：

在PyCharm中按住Ctrl键然后点击从外部引入的对象可以查看源码
  
#### add_scalar()方法的使用

tag指标题，scalar_value对应y轴，global_step对应x轴

add_scalar()的使用（常用来绘制train/val loss）：先创建一个SummaryWriter的实例，如writer = SummaryWriter('logs')，然后调用其方法，writer.add_scalar(tag, scalar, step)。最后记得关闭SummaryWriter。

```python
from torch.utils.tensorboard import SummaryWriter  
  
writer = SummaryWriter("logs")  
  
for i in range(100):  
    writer.add_scalar("y=2x", 2*i, i)  
  
writer.close()
```

#### add_image()的使用（常用来观察训练结果）

准备：载好read_data.py和练手数据集这两样。（但是本次貌似没有用到read_data.py）

1.利用Opencv读取图片，获得numpy型图片数据（由于下载太慢，下次再讲，这是第二次试图下载Opencv了）

2.利用numpy.array()，对PIL图片进行转换：

从PIL到numpy，需要在add_image()中通过dataformat参数指定shape中每一个数字/维度表示的含义。**或者用np.ndarray的transpose方法，将img_array进行多维度的转置，将通道所在的维度放在最前面，示例代码：img_array = img_array.transpose([2, 0, 1])**

## transforms的使用

之前用的工具都来自torch.utils，本次课开始接触torchvision

>transforms主要作用是对图片进行变换

### transforms的结构

可以使用PyCharm界面左侧的structure按钮查看transforms的结构：由许多类定义组成的**模块**。

### transforms的使用

由选择的类定义创建一个实例并给予其输入，最后接收其输出即可。

在函数的引用括号里按Ctrl+P可以显示函数需要传入什么参数及参数类型（P代表parameter）

1. 查看tensor数据类型的一些相关信息，可以理解为包含了神经网络理论基础的一些参数的数据类型。

2. 用opencv读取ndarray类型的数据（这是第三次安装Opencv了，终于成功了）

注：从anaconda.org上装opencv，并且注意opencv的读取目录中不能有中文。目前我用的Opencv版本为4.5.3（2023/4/25）

> 小结：目前学过的3种打开图片并往SummaryWriter类传参的方式：
> 
> 1. PIL.Image.open打开，np.array转换
> 2. PIL.Image.open打开，transforms.ToTensor转换
> 3. opencv打开（这个打开就是ndarray类型）

### 常见的transforms

1. Compose（介绍）：组合transforms中的多个类（多个工具）

Python中__call__的用法：Python类的魔法方法，可以让**实例**接收参数并被调用

传入的参数是一个由transfroms这个模块中类的**实例**组成的**列表**。

2. ToTensor：

将PIL Image和numpy.ndarray类型转换为tensor类型

3. Normalize：

对图片进行归一化的变换，具体效果见代码，注意计算公式

4. Resize：

作用是改变图片的尺寸，输入是PIL Image，输出同样是PIL Image。

5. RandomCrop：

随机裁剪的尺寸大小不能大于图片的尺寸，否则会报错。

4.总结使用方法：

1. 关注输入和输出类型
2. 关注官方文档
3. 关注方法需要什么参数
4. 不知道返回值类型的时候可以试错或者搜索（print，print(type())，debug）

```python
from PIL import Image  
from torchvision import transforms  
from torch.utils.tensorboard import SummaryWriter  
  
writer = SummaryWriter("logs")  
img=Image.open("hymenoptera_data_label/train/ants_image/0013035.jpg")  
print(img)  
  
#ToTensor  
trans_totensor = transforms.ToTensor()  
img_tensor = trans_totensor(img)  
writer.add_image("Totensor", img_tensor)  
  
#Normalize  
print(img_tensor[0][0][0])  
trans_norm=transforms.Normalize([0.485, 0.456, 0.406],[0.485, 0.456, 0.406])  
img_norm=trans_norm(img_tensor)  
print(img_norm[0][0][0])  
writer.add_image("Normalize", img_norm,2)  
  
#Resize  
print(img.size)  
trans_resize=transforms.Resize((512,512))  
img_resize=trans_resize(img)  
print(img_resize)  
print(type(img_resize))  
  
#img_resize PIL->totensor->img_resize tensor  
img_resize=trans_totensor(img_resize)  
writer.add_image("Resize", img_resize,0)  
  
#Compose -resize -2  
trans_resize_2=transforms.Resize(512)  
trans_compose=transforms.Compose([trans_resize_2,trans_totensor])  
img_resize_2=trans_compose(img)  
writer.add_image("Resize", img_resize_2,1)  
  
#RandomCrop  
trans_random=transforms.RandomCrop((200,200))  
trans_compose_2=transforms.Compose([trans_random,trans_totensor])  
for i in range(10):  
    img_crop=trans_compose_2(img)  
    writer.add_image("RandomCrop", img_crop,i)  
  
writer.close()
```

### torchvision中的数据集使用

#### 往期回顾

1. 自定义数据集
2. transforms中的类

**本节将讲解如何将数据集和transforms结合在一起。同时将介绍在科研或毕设中能够使用的标准数据集，包括下载->组织->查看->使用。totchvision.models可能在毕设或科研中使用到。**

#### CIFAR10数据集的下载与导入

如果下载比较慢，可以把下载链接放到迅雷中进行下载。

导入的datasets和之前讲解的Dataset类是很相似的，实现了__getitem__()方法和__len__()方法。

#### 将CIFAR10数据集的图片转换成tensor类型

1. 如果tensorboard的step不全可能是因为没加writer.close()

#### torchvision中的其他数据集

按住Ctrl键再点击可以查看源代码，找到url链接之后可以使用迅雷下载。

### Dataloader的使用

先看官网，再找例子。

1.去PyTorch官网上查找Dataloader的文档：

使用测试集的原因是测试集样本数更少，运行起来时间会短一些（指用DataLoader遍历的时间）

1. windows系统下如果出现BrokenPipeError的错误，可以考虑将 num_workers设置为0

2.采样器默认是随机采样：


3.用TensorBoard展示图片（注意要用add_images而不是之前用过的add_image）：


一定要等到程序运行结束了再在Terminal（终端）里使用tensorboard命令，不然会出现图片加载不全的情况！可以在程序末尾加一行语句来打印程序结束。