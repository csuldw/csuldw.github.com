---
layout: post
date: 2016-02-25 17:24
title: "机器学习数据集-MNIST"
categories: ML
tag: 
	- MNIST
	- dataset
	- Python
comment: true
---

## 介绍

在学习机器学习的时候，首当其冲的就是准备一份通用的数据集，方便与其他的算法进行比较。在这里，我写了一个用于加载MNIST数据集的方法，并将其进行封装，主要用于将MNIST数据集转换成numpy.array()格式的训练数据。直接下面看下面的代码吧(主要还是如何用python去读取binnary file)！

<!-- more -->

MNIST数据集原网址：[http://yann.lecun.com/exdb/mnist/](http://yann.lecun.com/exdb/mnist/)

Github源码下载：[数据集（源文件+解压文件+字体图像jpg格式）](https://github.com/csuldw/MachineLearning/tree/master/dataset/MNIST)， [py源码文件](https://github.com/csuldw/MachineLearning/tree/master/utils/)

## 文件目录

- [/utils/data_util.py](https://github.com/csuldw/MachineLearning/tree/master/utils/data_util.py) 用于加载MNIST数据集方法文件
- [/utils/test.py](https://github.com/csuldw/MachineLearning/tree/master/utils/test.py) 用于测试的文件，一个简单的KNN测试MNIST数据集
- [/data/train-images.idx3-ubyte](https://github.com/csuldw/MachineLearning/tree/master/dataset/MNIST) 训练集X
- [/dataset/train-labels.idx1-ubyte](https://github.com/csuldw/MachineLearning/tree/master/dataset/MNIST) 训练集y
- [/dataset/data/t10k-images.idx3-ubyte](https://github.com/csuldw/MachineLearning/tree/master/dataset/MNIST) 测试集X
- [/dataset/data/t10k-labels.idx1-ubyte](https://github.com/csuldw/MachineLearning/tree/master/dataset/MNIST) 测试集y


## MNIST数据集解释

将MNIST文件解压后，发现这些文件并不是标准的图像格式。这些图像数据都保存在二进制文件中。每个样本图像的宽高为28*28。

mnist的结构如下，选取train-images

    TRAINING SET IMAGE FILE (train-images-idx3-ubyte):

    [offset] [type]          [value]          [description] 
    0000     32 bit integer  0x00000803(2051) magic number 
    0004     32 bit integer  60000            number of images 
    0008     32 bit integer  28               number of rows 
    0012     32 bit integer  28               number of columns 
    0016     unsigned byte   ??               pixel 
    0017     unsigned byte   ??               pixel 
    ........ 
    xxxx     unsigned byte   ??               pixel


首先该数据是以二进制存储的，我们读取的时候要以'rb'方式读取；其次，真正的数据只有[value]这一项，其他的[type]等只是来描述的，并不真正在数据文件里面。也就是说，在读取真实数据之前，我们要读取4个`32 bit integer`.由[offset]我们可以看出真正的pixel是从0016开始的，一个int 32位，所以在读取pixel之前我们要读取4个 32 bit integer，也就是magic number, number of images, number of rows, number of columns. 当然，在这里使用struct.unpack_from()会比较方便.


## 源码

说明：

- '>IIII'指的是使用大端法读取4个unsinged int 32 bit integer
- '>784B'指的是使用大端法读取784个unsigned byte


data_util.py文件

```
# -*- coding: utf-8 -*-
"""
Created on Thu Feb 25 14:40:06 2016
load MNIST dataset
@author: liudiwei
"""
import numpy as np 
import struct
import matplotlib.pyplot as plt 
import os

class DataUtils(object):
    """MNIST数据集加载
    输出格式为：numpy.array()    
    
    使用方法如下
    from data_util import DataUtils
    def main():
        trainfile_X = '../dataset/MNIST/train-images.idx3-ubyte'
        trainfile_y = '../dataset/MNIST/train-labels.idx1-ubyte'
        testfile_X = '../dataset/MNIST/t10k-images.idx3-ubyte'
        testfile_y = '../dataset/MNIST/t10k-labels.idx1-ubyte'
        
        train_X = DataUtils(filename=trainfile_X).getImage()
        train_y = DataUtils(filename=trainfile_y).getLabel()
        test_X = DataUtils(testfile_X).getImage()
        test_y = DataUtils(testfile_y).getLabel()
        
        #以下内容是将图像保存到本地文件中
        #path_trainset = "../dataset/MNIST/imgs_train"
        #path_testset = "../dataset/MNIST/imgs_test"
        #if not os.path.exists(path_trainset):
        #    os.mkdir(path_trainset)
        #if not os.path.exists(path_testset):
        #    os.mkdir(path_testset)
        #DataUtils(outpath=path_trainset).outImg(train_X, train_y)
        #DataUtils(outpath=path_testset).outImg(test_X, test_y)
    
        return train_X, train_y, test_X, test_y 
    """


    def __init__(self, filename=None, outpath=None):
        self._filename = filename
        self._outpath = outpath
        
        self._tag = '>'
        self._twoBytes = 'II'
        self._fourBytes = 'IIII'    
        self._pictureBytes = '784B'
        self._labelByte = '1B'
        self._twoBytes2 = self._tag + self._twoBytes
        self._fourBytes2 = self._tag + self._fourBytes
        self._pictureBytes2 = self._tag + self._pictureBytes
        self._labelByte2 = self._tag + self._labelByte
    
    def getImage(self):
        """
        将MNIST的二进制文件转换成像素特征数据
        """
        binfile = open(self._filename, 'rb') #以二进制方式打开文件
        buf = binfile.read() 
        binfile.close()
        index = 0
        numMagic,numImgs,numRows,numCols=struct.unpack_from(self._fourBytes2,\
                                                                    buf,\
                                                                    index)
        index += struct.calcsize(self._fourBytes)
        images = []
        for i in range(numImgs):
            imgVal = struct.unpack_from(self._pictureBytes2, buf, index)
            index += struct.calcsize(self._pictureBytes2)
            imgVal = list(imgVal)
            for j in range(len(imgVal)):
                if imgVal[j] > 1:
                    imgVal[j] = 1
            images.append(imgVal)
        return np.array(images)
        
    def getLabel(self):
        """
        将MNIST中label二进制文件转换成对应的label数字特征
        """
        binFile = open(self._filename,'rb')
        buf = binFile.read()
        binFile.close()
        index = 0
        magic, numItems= struct.unpack_from(self._twoBytes2, buf,index)
        index += struct.calcsize(self._twoBytes2)
        labels = [];
        for x in range(numItems):
            im = struct.unpack_from(self._labelByte2,buf,index)
            index += struct.calcsize(self._labelByte2)
            labels.append(im[0])
        return np.array(labels)

    def outImg(self, arrX, arrY):
        """
        根据生成的特征和数字标号，输出png的图像
        """
        m, n = np.shape(arrX)
        #每张图是28*28=784Byte
        for i in range(1):
            img = np.array(arrX[i])
            img = img.reshape(28,28)
            outfile = str(i) + "_" +  str(arrY[i]) + ".png"
            plt.figure()
            plt.imshow(img, cmap = 'binary') #将图像黑白显示
            plt.savefig(self._outpath + "/" + outfile)
```


test.py文件:简单地测试了一下KNN算法，代码如下

```
# -*- coding: utf-8 -*-
"""
Created on Thu Feb 25 16:09:58 2016
Test MNIST dataset 
@author: liudiwei
"""

from sklearn import neighbors  
from data_util import DataUtils
import datetime  


def main():
    trainfile_X = '../dataset/MNIST/train-images.idx3-ubyte'
    trainfile_y = '../dataset/MNIST/train-labels.idx1-ubyte'
    testfile_X = '../dataset/MNIST/t10k-images.idx3-ubyte'
    testfile_y = '../dataset/MNIST/t10k-labels.idx1-ubyte'
    train_X = DataUtils(filename=trainfile_X).getImage()
    train_y = DataUtils(filename=trainfile_y).getLabel()
    test_X = DataUtils(testfile_X).getImage()
    test_y = DataUtils(testfile_y).getLabel()

    return train_X, train_y, test_X, test_y 


def testKNN():
    train_X, train_y, test_X, test_y = main()
    startTime = datetime.datetime.now()
    knn = neighbors.KNeighborsClassifier(n_neighbors=3)  
    knn.fit(train_X, train_y)  
    match = 0;  
    for i in xrange(len(test_y)):  
        predictLabel = knn.predict(test_X[i])[0]  
        if(predictLabel==test_y[i]):  
            match += 1  
      
    endTime = datetime.datetime.now()  
    print 'use time: '+str(endTime-startTime)  
    print 'error rate: '+ str(1-(match*1.0/len(test_y)))  

if __name__ == "__main__":
    testKNN()
```

通过main方法，最后直接返回numpy.array()格式的数据：train_X, train_y, test_X, test_y。如果你需要，直接条用main方法即可！

---
