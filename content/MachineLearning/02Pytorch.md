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


## 神经网路的基本骨架

### nn.Module的使用

从本节课开始讲网络的搭建。torch.nn有很多不同的类别，本节主要学习Containers。

1. nn=neural network

2. Containers中的Module模块：

3. 代码实战：

PyCharm的code栏下的generate功能可以辅助实现代码重写。

```python
import torch  
from torch import nn  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super().__init__()  
  
    def forward(self,input):  
        ouput=input+1  
        return ouput  
  
tudui=Tudui()  
x=torch.tensor(1.0)  
output=tudui(x)  
print(output)
```

### 土堆说卷积

本节将讲解神经网络中一些基本神经结构的使用。

1. 卷积层讲解

torch.nn是对torch.nn.functional的封装，比如车是对各个零部件的封装。

2. 举例讲解卷积操作
![[Pasted image 20260915184935.png]]
3. 代码实战：

本节主要讲了卷积运算和一些基本参数。

```python
import torch  
import torch.nn.functional as F  
  
input=torch.tensor([[1,2,0,3,1],  
                    [0,1,2,3,1],  
                    [1,2,1,0,0],  
                    [5,2,3,1,1],  
                    [2,1,0,1,1]])  
  
kernel=torch.tensor([[1,2,1],  
                     [0,1,0],  
                     [2,1,0],  
                     ])  
  
input=torch.reshape(input,(1,1,5,5))  
kernel=torch.reshape(kernel,(1,1,3,3))  
  
print(input.shape)  
print(kernel.shape)  
  
output=F.conv2d(input,kernel,stride=1)  
print(output)
```

### 神经网络——卷积层

1. Conv2d（Convolution Layers）讲解

>[Conv2d — PyTorch 2.14 documentation](https://docs.pytorch.org/docs/2.14/generated/torch.nn.Conv2d.html#torch.nn.Conv2d)

只需要定义kernel_size的大小，不需要定义其数据。

2. 实例讲解：

主要讲了如何进行卷积操作，并给了一个正向传播的例子。同时复习了导入数据的知识。
```python
import torch  
import torchvision  
from torch import nn  
from torch.utils.data import DataLoader  
from torch.utils.tensorboard import SummaryWriter  
  
dataset=torchvision.datasets.CIFAR10(root='./dataset',train=True,transform=torchvision.transforms.ToTensor(),download=True)  
  
dataloader=DataLoader(dataset,batch_size=64)  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui,self).__init__()  
        self.conv1=nn.Conv2d(in_channels=3,out_channels=6,kernel_size=3,stride=1,padding=0)  
  
    def forward(self,x):  
        x=self.conv1(x)  
        return x  
  
tudui=Tudui()  
  
writer=SummaryWriter("logs")  
  
step=0  
for data in dataloader:  
    imgs,targets=data  
    output=tudui(imgs)  
    # print(imgs.shape)  
    # print(output.shape)    writer.add_images("input",imgs,step)  
    #torch.size([64,6,30,30]) ->[xxx,3,30,30]  
  
    output=torch.reshape(output,(-1,3,30,30))  
    writer.add_images("output",output,step)  
  
    step+=1  
  
  
writer.close()
```
  

### 神经网络——最大池化的使用

1. Pooling Layers讲解

>[MaxPool2d — PyTorch 2.14 documentation](https://docs.pytorch.org/docs/2.14/generated/torch.nn.MaxPool2d.html#torch.nn.MaxPool2d)

最大池化有时也被称为下采样，对应的有上采样。注意ceil_mode参数的使用

2. 代码实战：

最大池化无法在长整型的数据上执行。生成tensor时可以使用dtype参数改变其数据类型，比如从长整型变为浮点型。

最大池化的作用在于保留输入数据的特征并减小数据的规模。
```python
import torch  
from numpy import dtype  
from torch import nn  
import torchvision  
from torch.utils.data import DataLoader  
from torch.utils.tensorboard import SummaryWriter  
  
dataset=torchvision.datasets.CIFAR10(root='./dataset',train=True,transform=torchvision.transforms.ToTensor(),download=True)  
  
dataloader=DataLoader(dataset,batch_size=64)  
  
#  
# input=torch.tensor([  
#     [1,2,0,3,1],  
#     [0,1,2,3,1],  
#     [1,2,1,0,0],  
#     [5,2,3,1,1],  
#     [2,1,0,1,1]  
# ],dtype=torch.float32)  
  
# input=torch.reshape(input,(-1,1,5,5))  
# print(input.shape)  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui,self).__init__()  
        self.maxpool=nn.MaxPool2d(kernel_size=3,ceil_mode=True)  
  
    def forward(self, input):  
        output=self.maxpool(input)  
        return output  
  
tudui=Tudui()  
  
writer=SummaryWriter("logs")  
  
# output=tudui(input)  
# print(output)  
  
step=0  
for data in dataloader:  
    imgs,targets=data  
    writer.add_images("input_maxpool",imgs,step)  
  
    output=tudui(imgs)  
    writer.add_images("output_maxpool",output,step)  
  
    step+=1  
  
writer.close()
```


### 神经网络——非线性激活

1.几种常见的非线性激活：

1. ReLU(Rectified Linear Unit)线性整流函数
2. Sigmoid

2.代码实战：

非线性变化的主要目的在于给网络引入非线性的特征。非线性特征越多，越能训练出符合各种曲线或特征的模型，从而提高模型的泛化能力。
```python
import torch  
import torchvision  
from torch import nn  
from torch.nn import ReLU, Sigmoid  
from torch.utils.tensorboard import SummaryWriter  
from torch.utils.data import DataLoader  
  
input=torch.tensor([[1,-0.5],[-1,3]])  
  
input=torch.reshape(input,(-1,1,2,2))  
print(input.shape)  
  
dataset=torchvision.datasets.CIFAR10(root='./dataset',train=True,download=True,transform=torchvision.transforms.ToTensor())  
  
dataloader=DataLoader(dataset,batch_size=2)  
  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui,self).__init__()  
        self.relu1=ReLU()  
        self.sigmoid1=Sigmoid()  
  
    def forward(self,input):  
        output=self.sigmoid1(input)  
        return output  
  
tudui=Tudui()  
  
writer=SummaryWriter("logs")  
  
step=0  
  
for data in dataloader:  
    imgs,targets=data  
    writer.add_images("input_relu",imgs,global_step=step)  
    output=tudui(imgs)  
    writer.add_images("output_relu",output,step)  
    step=step+1  
  
writer.close()
```

### 神经网络——线性层及其他层介绍

1.简要介绍nn模块里的各种层：

1. Normalization Layers正则化层
    正则化可以加快神经网络的训练速度，用的比较少，不作介绍，自己看文档
2. Recurrent Layers
    一般用于文字识别，自己看文档。
3. Transformer Layers
4. Linear Layers
5. Dropout Layers：
    主要作用是防止过拟合。
6. Saprse Layers：
    用于自然语言处理。
7. Distance Functions：
8. Loss Functions：
9. Linear Layers

Linear Layers的weight和bias的初始化是正态分布，可参考官方文档。


3.代码实战：

torch.flatten()可以展平数据

```python
import torch  
import torchvision  
from torch import nn  
from torch.nn import Linear  
from torch.utils.data import DataLoader  
  
dataset=torchvision.datasets.CIFAR10(root='./dataset',train=True,download=True,transform=torchvision.transforms.ToTensor())  
  
dataloader=DataLoader(dataset,batch_size=2)  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui,self).__init__()  
        self.linear1=Linear(6144,10)  
  
    def forward(self,input):  
        output=self.linear1(input)  
        return output  
  
tudui=Tudui()  
  
for data in dataloader:  
    imgs,targets=data  
    print(imgs.shape)  
    # output=torch.reshape(imgs,(1,1,1,-1))  
    output=torch.flatten(imgs)  
    print(output.shape)  
    output=tudui(output)  
    print(output.shape)
```
  
### 神经网络——搭建小实战和Sequential的使用

1.引子：

1. Sequential的使用：将网络结构放入其中即可，可以简化代码。
2. 找了一个对CIFAR10进行分类的模型，本节课将实现这个简单的模型。

2.代码实战：

nn.Flatten()和torch.flatten()有相同的效果。

3.Sequential讲解：

使用Sequential可以很大程度地简化代码。

4.利用TensorBoard进行数据可视化：

使用SummaryWriter的add_graph()方法进行数据可视化。

基本的网络搭建到此结束，但是下一节并不是PyTorch中提供的网络模型，P25才是。

```python
import torch  
import torchvision  
from torch import nn  
from torch.nn import Linear,Conv2d,MaxPool2d,Flatten,Sequential  
from torch.utils.data import DataLoader  
from torch.utils.tensorboard import SummaryWriter  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui,self).__init__()  
        # self.conv1 = Conv2d(3, 32, 5, padding=2)  
        # self.maxpool1 = MaxPool2d(2)        # self.conv2 = Conv2d(32, 32, 5, padding=2)        # self.maxpool2 = MaxPool2d(2,)        # self.conv3 = Conv2d(32, 64, 5, padding=2)        # self.maxpool3 = MaxPool2d(2)        # self.flatten = Flatten()        # self.linear1=Linear(1024,64)        # self.linear2=Linear(64,10)  
        self.model1=Sequential(  
            Conv2d(3, 32, 5, padding=2),  
            MaxPool2d(2),  
            Conv2d(32, 32, 5, padding=2),  
            MaxPool2d(2),  
            Conv2d(32, 64, 5, padding=2),  
            MaxPool2d(2),  
            Flatten(),  
            Linear(1024, 64),  
            Linear(64, 10),  
        )  
  
    def forward(self,x):  
        # x=self.conv1(x)  
        # x=self.maxpool1(x)        # x=self.conv2(x)        # x=self.maxpool2(x)        # x=self.conv3(x)        # x=self.maxpool3(x)        # x=self.flatten(x)        # x=self.linear1(x)        # x=self.linear2(x)        x=self.model1(x)  
        return x  
  
tuidui=Tudui()  
print(tuidui)  
input=torch.ones((64,3,32,32))  
output=tuidui(input)  
print(output.shape)  
  
writer=SummaryWriter("logs")  
writer.add_graph(tuidui,input)  
writer.close()
```

### 损失函数与反向传播

1.损失函数的作用：

1. 计算实际输出和目标之间的差距
2. 为我们更新输出提供一定的依据（反向传播）

2.介绍几种官方文档中的损失函数：

损失函数只能处理float类型的张量。

1. L1Loss (MAE)：
2. MSELoss：
3. CrossEntropyLoss：
4. 在神经网络中使用Loss Function：
```python
from torch import nn  
import torch  
from torch.nn import L1Loss  
  
inputs=torch.tensor([1,2,3],dtype=torch.float32)  
targets=torch.tensor([1,2,5],dtype=torch.float32)  
  
  
inputs=torch.reshape(inputs,(1,1,1,3))  
targets=torch.reshape(targets,(1,1,1,3))  
  
loss=L1Loss(reduction='sum')  
result=loss(inputs,targets)  
  
loss_mse=nn.MSELoss()  
result_mse=loss_mse(inputs,targets)  
  
print(result)  
print(result_mse)  
  
x=torch.tensor([0.1,0.2,0.3])  
y=torch.tensor([1])  
x=torch.reshape(x,(1,3))  
loss_cross=nn.CrossEntropyLoss()  
result_cross=loss_cross(x,y)  
print(result_cross)
```

```python
import torch  
import torchvision  
from torch import nn  
from torch.nn import Linear,Conv2d,MaxPool2d,Flatten,Sequential  
from torch.utils.data import DataLoader  
from torch.utils.tensorboard import SummaryWriter  
  
  
dataset=torchvision.datasets.CIFAR10("./dataset",train=True,transform=torchvision.transforms.ToTensor(),download=True)  
  
dataloader=DataLoader(dataset,batch_size=64)  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui,self).__init__()  
        self.model1=Sequential(  
            Conv2d(3, 32, 5, padding=2),  
            MaxPool2d(2),  
            Conv2d(32, 32, 5, padding=2),  
            MaxPool2d(2),  
            Conv2d(32, 64, 5, padding=2),  
            MaxPool2d(2),  
            Flatten(),  
            Linear(1024, 64),  
            Linear(64, 10),  
        )  
  
    def forward(self,x):  
        x=self.model1(x)  
        return x  
  
loss=nn.CrossEntropyLoss()  
  
tuidui=Tudui()  
  
for data in dataloader:  
    imgs,targets=data  
    outputs=tuidui(imgs)  
    result_loss=loss(outputs,targets)  
    result_loss.backward()  
    print("ok")  
    # print(result_loss)  
    # print(outputs)    # print(targets)
```
### 优化器（一）

优化器集中在torch.optim中。

1. 主要讲了优化器的基本使用，如果要知道各个优化器的详细用法需要对其有一定了解
2. 注意要多训练几轮
```python
import torch  
import torchvision  
from torch import nn  
from torch.nn import Linear,Conv2d,MaxPool2d,Flatten,Sequential  
from torch.utils.data import DataLoader  
from torch.utils.tensorboard import SummaryWriter  
  
  
dataset=torchvision.datasets.CIFAR10("./dataset",train=True,transform=torchvision.transforms.ToTensor(),download=True)  
  
dataloader=DataLoader(dataset,batch_size=64)  
  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui,self).__init__()  
        self.model1=Sequential(  
            Conv2d(3, 32, 5, padding=2),  
            MaxPool2d(2),  
            Conv2d(32, 32, 5, padding=2),  
            MaxPool2d(2),  
            Conv2d(32, 64, 5, padding=2),  
            MaxPool2d(2),  
            Flatten(),  
            Linear(1024, 64),  
            Linear(64, 10),  
        )  
  
    def forward(self,x):  
        x=self.model1(x)  
        return x  
  
loss=nn.CrossEntropyLoss()  
  
tuidui=Tudui()  
  
optim=torch.optim.SGD(tuidui.parameters(),lr=0.01)  
  
for epoch in range(20):  
    running_loss=0.0  
    for data in dataloader:  
        imgs,targets=data  
        outputs=tuidui(imgs)  
        result_loss=loss(outputs,targets)  
        optim.zero_grad()  
        result_loss.backward()  
        optim.step()  
        running_loss=running_loss+result_loss  
    print(running_loss)
```
  
### 现有网络模型的使用及修改

1.讲解VGG16网络模型：

ImageNet数据集太大了，仅训练集就有147.9g。

2.ImageNet数据集介绍


3.改变现有网络的参数

vgg16的导入以及和视频里讲的不一样了，并且我在导入预训练权重的时候遇到了url error的问题，不知道是不是因为被墙了。

```python
import torchvision

# train_data = torchvision.datasets.ImageNet("../data_image_net", split='train', download=True,
#                                            transform=torchvision.transforms.ToTensor())
from torch import nn

vgg16_false = torchvision.models.vgg16(pretrained=False)
vgg16_true = torchvision.models.vgg16(pretrained=True)

print(vgg16_true)

train_data = torchvision.datasets.CIFAR10('../data', train=True, transform=torchvision.transforms.ToTensor(),
                                          download=True)

vgg16_true.classifier.add_module('add_linear', nn.Linear(1000, 10))
print(vgg16_true)

print(vgg16_false)
vgg16_false.classifier[6] = nn.Linear(4096, 10)
print(vgg16_false)
```

### 网络模型的保存与读取

1. 模型的保存与读取方法1：

torch.save(实例, 保存名称), torch.load(实例, 保存名称)。

方法1保存的是：模型结构+模型参数

2. 模型的保存与读取方法2：

torch.save(实例.state_dict(), 保存名称), torch.load(实例.state_dict(), 保存名称)。

方法2保存的是：模型参数（官方推荐）。

3. 方法1的陷阱：

用方法1的时候一定要保证读取模型的文件里有定义该模型的类！

```python
import torch
from model_save import *
# 方式1-》保存方式1，加载模型
import torchvision
from torch import nn

model = torch.load("vgg16_method1.pth")
# print(model)

# 方式2，加载模型
vgg16 = torchvision.models.vgg16(pretrained=False)
vgg16.load_state_dict(torch.load("vgg16_method2.pth"))
# model = torch.load("vgg16_method2.pth")
# print(vgg16)

# 陷阱1
# class Tudui(nn.Module):
#     def __init__(self):
#         super(Tudui, self).__init__()
#         self.conv1 = nn.Conv2d(3, 64, kernel_size=3)
#
#     def forward(self, x):
#         x = self.conv1(x)
#         return x

model = torch.load('tudui_method1.pth')
print(model)
```



```python
import torch
import torchvision
from torch import nn

vgg16 = torchvision.models.vgg16(pretrained=False)
# 保存方式1,模型结构+模型参数
torch.save(vgg16, "vgg16_method1.pth")

# 保存方式2，模型参数（官方推荐）
torch.save(vgg16.state_dict(), "vgg16_method2.pth")

# 陷阱
class Tudui(nn.Module):
    def __init__(self):
        super(Tudui, self).__init__()
        self.conv1 = nn.Conv2d(3, 64, kernel_size=3)

    def forward(self, x):
        x = self.conv1(x)
        return x

tudui = Tudui()
torch.save(tudui, "tudui_method1.pth")
```



## 完整的模型训练套路

>完成CIFAR10的分类问题

1. 准备数据集

其实用len去查看数据集的长度已经不是新知识点了。当我们要重写Dataset类的时候，关键需要重写Dataset类的__len__()方法和__getitem__()方法。

2. 利用DataLoader来加载数据集

3. 搭建神经网络

将搭建的网络模型放入单独的一个.py文件中，并进行验证。

4. 创建网络模型

5. 创建损失函数

6. 设置优化器

推荐使用科学计数法表示学习率。

7. 设置训练网络的一些参数
8. 设置测试部分
用with torch.no_grad():环境取消梯度。
9. 用tensorboard显示loss的图像
10. 保存训练参数
11. 利用torch.argmax函数计算准确率

本节进行训练细节总结。

在官网的torch.nn.Module小节中可以查看train mode 和eval mode。

.train()和.eval()只对特殊的层起作用，如Dropout和BatchNorm层，但是最好还是加上。

即在训练开始前加上.train()，在测试开始前加上.eval()。

```python
import torchvision  
from torch import nn  
from torch.utils.data import DataLoader  
import torch  
from torch.utils.tensorboard import SummaryWriter  
  
  
from nn_linear import Tudui  
  
# 搭建神经网络  
class Tudui(nn.Module):  
    def __init__(self):  
        super(Tudui, self).__init__()  
        self.model = nn.Sequential(  
            nn.Conv2d(3, 32, 5, 1, 2),  
            nn.MaxPool2d(2),  
            nn.Conv2d(32, 32, 5, 1, 2),  
            nn.MaxPool2d(2),  
            nn.Conv2d(32, 64, 5, 1, 2),  
            nn.MaxPool2d(2),  
            nn.Flatten(),  
            nn.Linear(64*4*4, 64),  
            nn.Linear(64, 10)  
        )  
  
    def forward(self, x):  
        x = self.model(x)  
        return x  
  
  
if __name__ == '__main__':  
    tudui = Tudui()  
    input = torch.ones((64, 3, 32, 32))  
    output = tudui(input)  
    # print(output.shape)  
  
  
train_data=torchvision.datasets.CIFAR10(root='./dataset',train=True,download=True,transform=torchvision.transforms.ToTensor())  
  
test_data=torchvision.datasets.CIFAR10(root='./dataset',train=False,download=True,transform=torchvision.transforms.ToTensor())  
  
train_data_size=len(train_data)  
test_data_size=len(test_data)  
  
print("训练数据集的长度为：{}".format(train_data_size))  
print("测试数据集的长度为：{}".format(test_data_size))  
  
train_dataloader=DataLoader(train_data,batch_size=64)  
test_dataloader=DataLoader(test_data,batch_size=64)  
  
# 创建网络模型  
tudui=Tudui()  
  
# 损失函数  
loss_fn=nn.CrossEntropyLoss()  
  
# 优化器  
# learning_rate=0.01  
# 1e-2=1x(10)^(-2)=1/100=0.01  
learning_rate=1e-2  
optimizer=torch.optim.SGD(tudui.parameters(),lr=learning_rate)  
  
# 设置训练网络的一些参数  
# 记录训练的参数  
total_train_step=0  
# 记录测试的次数  
total_test_step=0  
# 训练的轮数  
epoch=10  
  
# 添加tensorboard  
writer=SummaryWriter("logs")  
  
for i in range(epoch):  
    print("------第{}轮训练开始------".format(i+1))  
  
    #训练步骤开始  
    tudui.train()  
    for data in train_dataloader:  
        imgs,targets=data  
        outputs=tudui(imgs)  
        loss=loss_fn(outputs,targets)  
  
        #优化器优化模型  
        optimizer.zero_grad()  
        loss.backward()  
        optimizer.step()  
  
        total_train_step=total_train_step+1  
        if total_train_step%100==0:  
            print("训练次数：{}，Loss:{}".format(total_train_step,loss.item()))  
            writer.add_scalar('train_loss',loss.item(),total_train_step)  
  
    #测试步骤开始  
    tudui.eval()  
    total_test_loss=0  
    total_accuracy=0  
    with torch.no_grad():  
        for data in test_dataloader:  
            imgs,targets=data  
            outputs=tudui(imgs)  
            loss=loss_fn(outputs,targets)  
            total_test_loss=total_test_loss+loss.item()  
            accuracy=(outputs.argmax(1) == targets).sum()  
            total_accuracy=total_accuracy+accuracy  
  
    print("整体测试集上的Loss:{}".format(total_test_loss))  
    print("整体测试集上的正确率：{}".format(total_accuracy/train_data_size))  
    writer.add_scalar('test_loss',total_test_loss,total_test_step)  
    writer.add_scalar('test_accuracy',total_accuracy/train_data_size,total_test_step)  
    total_test_step=total_test_step+1  
  
    torch.save(tudui,"tudui_{}.pth".format(i))  
    print("模型已保存")  
  
writer.close()
```

## 利用GUP训练（一）

本节介绍两种实现代码在GPU上训练的方式。

1.方法1：

找到：

1. 网络模型
2. 数据（输入、标注）
3. 损失函数

这三样，然后调用其.cuda()方法，最后返回即可。不过使用这种方法很麻烦。

2.使用Google Colab（前提是能访问Google）：

3.可以尝试使用kaggle提供的虚拟机。

## 利用GUP训练（二）

1.代码实战：

1. 调用.to()方法即可。
2. 网络模型和损失函数是不需要重新赋值的，只有数据需要重新赋值。

## 完整的模型验证（测试，demo）套路

利用已经训练好的模型，然后给它提供输入

1.png格式的图片是4通道，jpg格式的图片是3通道，可以用PIL.Image.convert('RGB')来转换（不同截图软件保留的通道数也不一样）。

2.小细节：

将model设置到eval mode

with torch.no_grad()可以节约内存，提高性能。

3.将在gpu上训练的模型导入到在cup上运行的程序时要添加：

map_location=torch.device('cpu')关键字参数。
  

## 【完结】看看开源项目

推荐把参数设置中的required替换成default!



