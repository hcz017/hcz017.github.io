---
date: 2022-04-12 20:14
title: 【转】opencv光流预测和remap重映射函数使用
tags:
  - opencv
  - 光流
---

原文链接： https://cloud.tencent.com/developer/article/1831477

# 光流

optical flow （光流） 表示的是相邻两帧图像中每个像素的运动速度和运动方向。
假设我们有如下光流的颜色空间表示：

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/ml/_image/flow_vis.png)

光流颜色空间表示
再假设整个图片中物体均向左上移动，那么可以得到如下的光流图。

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/ml/_image/flow_vis-2.png)

# 光流法

光流法就是通过检测图像像素点的强度随时间的变化进而推断出物体的光流的方法。
今天主要介绍opencv中计算光流接口cv2.calcOpticalFlowFarneback的使用，以及如果已知当前帧和预测光流，我们如何通过重映射cv2.remap得到预测图像的方法。

# cv2.calcOpticalFlowFarneback函数

cv2.calcOpticalFlowFarneback是opencv中使用Gunnar Farneback算法计算稠密光流的函数。
算法论文：[https://www.ida.liu.se/ext/WITAS-ev/Computer_Vision_Technologies/Papers/scia03_farneback.pdf](https://www.ida.liu.se/ext/WITAS-ev/Computer_Vision_Technologies/Papers/scia03_farneback.pdf)

flow = cv2.calcOpticalFlowFarneback(prev, next, flow, pyr_scale, levels, winsize, iterations, poly_n, poly_sigma, flags)

函数参数：

- prev：当前帧图像，单通道图像，彩色图像通常需要使用cv2.COLOR_BGR2GRAY
- next：下一帧单通道图像,大小和prev一致
- flow： 计算的光流图，和prev大小一致，CV_32FC2类型;
- pyr_scale： 金字塔上下两层之间的尺度关系，该参数一般设置为pyrScale=0.5，表示图像金字塔上一层是下一层的2倍降采样
- levels：图像金字塔的层数，levels = 1意味着不会创建额外的图层，只会使用原始图像。
- winsize：平均窗口大小，winsize越大，算法对图像噪声越鲁棒，并且能提升对快速运动目标的检测效果，但也会引起运动区域模糊。
- iterations：算法在图像金字塔每层的迭代次数
- poly_n：用于在每个像素点处计算多项式展开的相邻像素点的个数。poly_n越大，图像的近似逼近越光滑，算法鲁棒性更好，也会带来更多的运动区域模糊。通常，poly_n=5 or 7
- poly_sigma：用于平滑导数的高斯的标准偏差，用作多项式展开的基础，通常poly_n=5时，poly_sigma = 1.1；poly_n=7时，poly_sigma = 1.5
- flags：可选参数值OPTFLOW_USE_INITIAL_FLOW 和 OPTFLOW_FARNEBACK_GAUSSIAN

函数使用：

```python
i_t0 = cv2.imread("1_t0.jpg")
i_t1 = cv2.imread("1_t2.jpg")
g_i_t0 = cv2.cvtColor(i_t0, cv2.COLOR_BGR2GRAY)
g_i_t1 = cv2.cvtColor(i_t1, cv2.COLOR_BGR2GRAY)
flow = cv2.calcOpticalFlowFarneback(g_i_t0, g_i_t1, None, 0.5, 3, 15, 3, 5, 1.1, 0)
```

这样就计算出flow01的光流，注意这里计算的是反向光流，光流方向：左正右负，上正下负。上面代码计算出是从t1到t0的光流，整体向右下运动，光流图橘色，对应光流值为负值。
假设我们得到了光流flow，就可以通过t0的图像和flow，来预测t1时刻的图像。这里需要使用remap重映射函数。

# cv2.remap函数

cv2.remap是opencv的重映射函数
cv2.remap(src, map1, map2, interpolation, borderMode, borderValue )

- src： 代表原始图像
- map1：表示(x，y)点的一个映射点或者仅表示(x，y)点的x值
- map2：如果map1表示(x,y)的映射值，map2为空，否者表示(x，y)点的y值
- Interpolation: 插值方式
- borderMode： 边界模式。当该值为 BORDER_TRANSPARENT时,表示目标图像内的对应源图像内奇异点( outliers)的像素不会被修改
- borderValue： 代表边界值,默认为0

remap函数实际就是通过修改像素点的位置得到一幅新图像。我们要构建一个目标图像，就需要知道目标图像每个像素点在原始图像中的位置。由于map得到的是float，所以可能映射到多个坐标之间的位置，而且新图像的大小也可能变化，所以参数中有个插值方法。
remap在图像变形，图像扭曲等应用中都会用到。
在本文中，我们通过上文已经有前一帧的图像数据，又有了图像的光流数据，就可以得到map。再通过重映射就可以通过光流预测恢复出下一帧的数据。
使用代码：

```python
def cv_warp(input, flow):
    h, w = flow.shape[:2]
    warp_grid_x, warp_grid_y = np.meshgrid(np.linspace(0, w-1, w), np.linspace(0, h-1, h))
    flow_inv = flow + np.stack((warp_grid_x, warp_grid_y), axis=-1)
    flow_inv = flow_inv.astype(np.float32)
    warped = cv2.remap(input, flow_inv, None, cv2.INTER_LINEAR)
    return warped
```

目前也有许多使用深度学习来预测光流的模型，后续有时间进行介绍。

原文链接： https://cloud.tencent.com/developer/article/1831477