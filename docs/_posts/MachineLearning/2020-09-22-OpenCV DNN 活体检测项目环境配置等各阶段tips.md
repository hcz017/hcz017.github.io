---
date: 2020-09-22 14:53
status: public
title: OpenCV DNN 活体检测项目环境配置等各阶段tips
---

资料来源《OpenCV深度学习应用与性能优化实践》第八章。

在复现这个项目的时候发现一些可以调整的小tips。

# 环境配置阶段
使用conda 创建python 工作环境时，注释掉requirems.txt 里的opencv-python-inference-engine==4.1.2.1，安装OpenVINO 时包含这个了，如果使用requirements 里的版本，imshow 会不可用。
另外安装OpenVINO 后一定要配置环境，指定下面的命令是配置生效，也可以选择加到~/.bashrc 文件里
```bash
$source /opt/intel/openvino/bin/setupvars.sh
[setupvars.sh] OpenVINO environment initialized
```

# 采集数据阶段
涉及文件 gather_examples.py
## 调整采集数据频率
如果觉得采集的的速度较慢/快，可以采集的时候加--skip 参数 来调整（或者直接修改），此处含义为每16 帧处理一帧。

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/ml/_image/code_frames.png)

## 中断后继续采集数据
如果采集数据的时候中途被迫停止了，继续采集数据想要接上之前的编号，修改：

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/ml/_image/code_save.png)

## 实时显示采集数据的图片
实时显示color image 和输出的depth face，方便观察数据优劣（距离角度等），方便动态调整。做以下修改：

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/ml/_image/code_show_input.png)

# 训练阶段
如果遇到模块找不到的提示，将train/train_FeatherNet.py 需要移到根目录。
默认参数来自 train/cfgs/FeatherNet.yaml，经试验，这里面已经包含的参数，在使用命令训练的时候是不会被覆盖的。比如你想调整训练的最大迭代(epochs)次数，train_FeatherNet.py --epochs 是不会生效的。要么直接改上面的文件，要么注释掉文件里的配置再在训练的时候跟参数。

# 推理阶段
即demo run 的阶段。
代码中有个bug，活体检测的输入图不是单张人脸，而是整张图，这可能包含多张人脸，于是多张人脸的检测见过其实用的是同一张图，结果也就一样，即同为false 或同为true。

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/ml/_image/code_bug_fix.png)



源码地址： https://github.com/hcz017/OpenCV_DNN_face_anti_spoofing