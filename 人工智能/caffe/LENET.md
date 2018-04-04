# LeNet神经网络

|

 

** 1512

LeNet神经网络

文章作者：Tyan
博客：[noahsnail.com](http://noahsnail.com/)  |  [CSDN](http://blog.csdn.net/quincuntial)  |  [简书](http://www.jianshu.com/users/7731e83f3a4e/latest_articles)

## 1. LeNet神经网络介绍

LeNet神经网络由深度学习三巨头之一的Yan LeCun提出，他同时也是卷积神经网络 (CNN，Convolutional Neural Networks)之父。LeNet主要用来进行手写字符的识别与分类，并在美国的银行中投入了使用。LeNet的实现确立了CNN的结构，现在神经网络中的许多内容在LeNet的网络结构中都能看到，例如卷积层，Pooling层，ReLU层。虽然LeNet早在20世纪90年代就已经提出了，但由于当时缺乏大规模的训练数据，计算机硬件的性能也较低，因此LeNet神经网络在处理复杂问题时效果并不理想。虽然LeNet网络结构比较简单，但是刚好适合神经网络的入门学习。

## 2. LeNet神经网络结构

LeNet的神经网络结构图如下：

[![LeNet网络图](http://img.blog.csdn.net/20171217210132224?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217210132224?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

LeNet网络的执行流程图如下：

[![LeNet图像处理流程](http://img.blog.csdn.net/20171217210233200?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217210233200?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 2.1 LeNet第一层（卷积运算）

接下来我们来具体的一层层的分析LeNet的网络结构。首先要了解图像（输入数据）的表示。在LeNet网络中，输入图像是手写字符，图像的表示形式为二维数据矩阵，如下图所示：

[![图像的表示](http://img.blog.csdn.net/20171217210308504?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217210308504?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

LeNet网络除去输入输出层总共有六层网络。第一层是卷积层（C1层），卷积核的大小为`5\*5`，卷积核数量为`6`个，输入图像的大小为`32*32`，因此输入数据在进行第一层卷积之后，输出结果为大小为`28*28`，数量为`6`个的feature map。卷积操作如下面两幅图所示：

[![卷积演示](http://img.blog.csdn.net/20171217210359992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217210359992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

[![卷积演示](http://upload-images.jianshu.io/upload_images/3232548-3bf21864a5897262.gif?imageMogr2/auto-orient/strip)](http://upload-images.jianshu.io/upload_images/3232548-3bf21864a5897262.gif?imageMogr2/auto-orient/strip)

卷积操作的过程可描述为：卷积核在图像上滑动，滑动步长为1（即每次移动一格，水平方向从左到右，到最右边之后再从最左边开始，向下移动一格，重复从左到右滑动），当卷积核与图像的一个局部块重合时进行卷积运行，卷积计算方式为图像块对应位置的数与卷积核对应位置的数相乘，然后将所有相乘结果相加即为feature map的值，**相乘累加之后的结果位于卷积核中心点的位置**，因此如果是`3\*3`的卷积核，feature map比原图像在水平和垂直方向上分别减少两行（上下各一行）和两列（左右各一列），因此上面图像原图为`5*5`，卷积核为`3\*3`，卷积结果大小为`3*3`，即`(5-2)*(5-2)`，如果卷积核为`5*5`，则卷积结果大小为`(5-4)*(5-4)`。上图中的卷积核为：
[![卷积核](http://img.blog.csdn.net/20171217211323831?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217211323831?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由于神经网络层与层的结构是通过连接来实现的，因此输入层与第一个卷积层的连接数量应为`(32-2-2)\*(32-2-2)\*(5\*5+1)\*6= 28\*28\*156 =122304`。

卷积的作用主要是：通过卷积运算，可以使原信号特征增强，并且降低噪音。在图像上卷积之后主要是减少图像噪声，提取图像的特征。例如sobel算子就是一种卷积运算，主要是提取图像的边缘特征。卷积网络能很好地适应图像的平移不变性：例如稍稍移动一幅猫的图像，它仍然是一幅猫的图像。卷积操作保留了图像块之间的空间信息，进行卷积操作的图像块之间的相对位置关系没有改变。图像在不同卷积核上进行卷积之后的效果图如下：

[![同一幅图像用不同卷积核处理的效果](http://img.blog.csdn.net/20171217211732300?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217211732300?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 2.2 LeNet第二层（pooling运算）

图像在LeNet网络上进行第一层卷积之后，结果为大小为`28*28`，数量为`6`个的feature map。LeNet网络的第二层为pooling层（S2层），也称为下采样。在图像处理中，下采样之后，图像的大小会变为原来的`1/4`，即水平方向和垂直方向上图像大小分别减半。Pooling有多种，这里主要介绍两种，max-pooling和average-pooling。max-pooling即为从四个元素中选取一个最大的来表示这四个元素，average-pooling则用四个元素的平均值来表示这四个元素。Pooling示意图如下：

[![Pooling示意图](http://img.blog.csdn.net/20171217211823115?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217211823115?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

[![Pooling示意图](http://img.blog.csdn.net/20171217211903693?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217211903693?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在LeNet在进行第二层Pooling运算后，输出结果为`14*14`的`6`个feature map。其连接数为`(2*2+1) * 14 * 14 *6 = 5880`。Pooling层的主要作用就是减少数据，降低数据纬度的同时保留最重要的信息。在数据减少后，可以减少神经网络的纬度和计算量，可以防止参数太多过拟合。LeNet在这一层是将四个元素相加，然后乘以参数w再加上偏置b，然后计算sigmoid值。

### 2.3 LeNet第三层（卷积运算）

LeNet第三层（C3层）也是卷积层，卷积核大小仍为`5*5`，不过卷积核的数量变为`16`个。第三层的输入为`14*14`的`6`个feature map，卷积核大小为`5*5`，因此卷积之后输出的feature map大小为`10*10`，由于卷积核有`16`个，因此希望输出的feature map也为`16`个，但由于输入有`6`个feature map，因此需要进行额外的处理。输入的`6`个feature map与输出的`16`个feature map的关系图如下：

[![S2层与C3层的关系图](http://img.blog.csdn.net/20171217211953651?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://img.blog.csdn.net/20171217211953651?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUXVpbmN1bnRpYWw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如上图所示，第一个卷积核处理前三幅输入的feature map，得出一个新的feature map。

### 2.4 LeNet第四层（Pooling运算）

上一层卷积运算之后，结果为大小为`10*10`的`16`个feature map，因此在第四层（S4层）进行pooling运算之后，输出结果为`16`个大小为`5*5`的feature map。与S2层进行同样的操作。

### 2.5 LeNet第五层

LeNet第五层是卷积层(C5层)，卷积核数目为120个，大小为`5*5`，由于第四层输出的feature map大小为`5*5`，因此第五层也可以看成全连接层，输出为120个大小为`1*1`的feature map。

### 2.6 LeNet第六层

LeNet第六层是全连接层（F6层），有84个神经元（84与输出层的设计有关），与C5层全连接。

### 2.7 LeNet各层的参数变化

- C1
  输入大小：32*32
  核大小：5*5
  核数目：6
  输出大小：28*28*6
  训练参数数目：(5*5+1)*6=156
  连接数：(5*5+1)*6*(32-2-2)*(32-2-2)=122304
- S2
  输入大小：28*28*6
  核大小：2*2
  核数目：1
  输出大小：14*14*6
  训练参数数目：2*6=12，2=(w,b)
  连接数：(2*2+1)*1*14*14*6 = 5880
- C3
  输入大小：14*14*6
  核大小：5*5
  核数目：16
  输出大小：10*10*16
  训练参数数目：6*(3*5*5+1) + 6*(4*5*5+1) + 3*(4*5*5+1) + 1*(6*5*5+1)=1516
  连接数：(6*(3*5*5+1) + 6*(4*5*5+1) + 3*(4*5*5+1) + 1*(6*5*5+1))*10*10=151600
- S4
  输入大小：10*10*16
  核大小：2*2
  核数目：1
  输出大小：5*5*16
  训练参数数目：2*16=32
  连接数：(2*2+1)*1*5*5*16=2000
- C5
  输入大小：5*5*16
  核大小：5*5
  核数目：120
  输出大小：120*1*1
  训练参数数目：(5*5*16+1)*120*1*1=48120（因为是全连接）
  连接数：(5*5*16+1)*120*1*1=48120
- F6
  输入大小：120
  输出大小：84
  训练参数数目：(120+1)*84=10164
  连接数：(120+1)*84=10164

### 3. LeNet在Caffe中的配置

LeNet神经网络结构在Caffe中的配置文件如下：

```
name: "LeNet" //神经网络名字
//本层只有top，没有bottom，说明是数据输入层
layer {
  name: "mnist" //layer名字
  type: "Data" //层的数据类型，如果是Data，说明是leveldb或lmdb
  top: "data" //top表示输入数据，类型为data
  top: "label" //top表示输入数据，类型为label，(data,label)配对是分类模型所必需的
  //一般训练的时候和测试的时候，模型的层是不一样的。该层（layer）是属于训练阶段的层，还是属于测试阶段的层，需要用include来指定。如果没有include参数，则表示该层既在训练模型中，又在测试模型中。
  include {
    phase: TRAIN
  }
  //数据的预处理，可以将数据变换到定义的范围内。如设置scale为0.00390625，实际上就是1/255, 即将输入数据由0-255归一化到0-1之间
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "examples/mnist/mnist_train_lmdb"
    batch_size: 64
    backend: LMDB
  }
}
layer {
  name: "mnist"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TEST
  }
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "examples/mnist/mnist_test_lmdb"
    batch_size: 100
    backend: LMDB
  }
}
layer {
  name: "conv1"
  type: "Convolution"
  bottom: "data"
  top: "conv1"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  convolution_param {
    num_output: 20
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool1"
  type: "Pooling"
  bottom: "conv1"
  top: "pool1"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "conv2"
  type: "Convolution"
  bottom: "pool1"
  top: "conv2"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  convolution_param {
    num_output: 50
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool2"
  type: "Pooling"
  bottom: "conv2"
  top: "pool2"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "ip1"
  type: "InnerProduct"
  bottom: "pool2"
  top: "ip1"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 500
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "relu1"
  type: "ReLU"
  bottom: "ip1"
  top: "ip1"
}
layer {
  name: "ip2"
  type: "InnerProduct"
  bottom: "ip1"
  top: "ip2"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "ip2"
  bottom: "label"
  top: "accuracy"
  include {
    phase: TEST
  }
}
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "ip2"
  bottom: "label"
  top: "loss"
}
```