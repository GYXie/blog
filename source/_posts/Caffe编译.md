title: Caffe编译
date: 2017-01-27 01:35:37
categories: Deep Learning
tags:
  - Caffe
  - Deep Learning
---

## 环境
- 系统：Ubuntu 16.04

## 遇到的问题
1. 找不到protobuf的文件  
原因：机器上装了Anaconda，而Anaconda默认装了很多东西，包括protobuf。  
解决方案：注释掉`.bashrc`中的Anaconda的环境变量，然后重新开一个Terminal开始编译。 
2. 找不到hdf5的文件   
解决方案：修改`Makefile.config`大致第九十几行的配置，加上hdf5的文件路径。  
```
# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include  /usr/include/hdf5/serial/
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib  /usr/lib/x86_64-linux-gnu/hdf5/serial/
```
