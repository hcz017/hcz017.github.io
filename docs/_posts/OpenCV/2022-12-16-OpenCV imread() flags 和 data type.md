---
Date: 2022-12-16
tags:
  - opencv
title: OpenCV imread() flags 和 data type
---

默认值为1：即
always convert image to the 3 channel BGR color image

```c++
enum ImreadModes {
       IMREAD_UNCHANGED            = -1, //!< If set, return the loaded image as is (with alpha channel, otherwise it gets cropped).//原始图像有alpha通道的话，就保存该通道数据，否则，别的flags会直接将alpha通道丢弃。
       IMREAD_GRAYSCALE            = 0,  //!< If set, always convert image to the single channel grayscale image.
       IMREAD_COLOR                = 1,  //!< If set, always convert image to the 3 channel BGR color image.
       IMREAD_ANYDEPTH             = 2,  //!< If set, return 16-bit/32-bit image when the input has the corresponding depth, otherwise convert it to 8-bit.
       IMREAD_ANYCOLOR             = 4,  //!< If set, the image is read in any possible color format.
       IMREAD_LOAD_GDAL            = 8,  //!< If set, use the gdal driver for loading the image.
       IMREAD_REDUCED_GRAYSCALE_2  = 16, //!< If set, always convert image to the single channel grayscale image and the image size reduced 1/2.
       IMREAD_REDUCED_COLOR_2      = 17, //!< If set, always convert image to the 3 channel BGR color image and the image size reduced 1/2.
       IMREAD_REDUCED_GRAYSCALE_4  = 32, //!< If set, always convert image to the single channel grayscale image and the image size reduced 1/4.
       IMREAD_REDUCED_COLOR_4      = 33, //!< If set, always convert image to the 3 channel BGR color image and the image size reduced 1/4.
       IMREAD_REDUCED_GRAYSCALE_8  = 64, //!< If set, always convert image to the single channel grayscale image and the image size reduced 1/8.
       IMREAD_REDUCED_COLOR_8      = 65, //!< If set, always convert image to the 3 channel BGR color image and the image size reduced 1/8.
       IMREAD_IGNORE_ORIENTATION   = 128 //!< If set, do not rotate the image according to EXIF's orientation flag.
     };
```

OpenCV 数据类型

```C++
#define CV_8U   0
#define CV_8S   1
#define CV_16U  2
#define CV_16S  3
#define CV_32S  4
#define CV_32F  5
#define CV_64F  6
#define CV_USRTYPE1 7
```
其他格式的值：
```c++
CV_8U 0
CV_8S 1
CV_16U 2
CV_16S 3
CV_32S 4
CV_32F 5
CV_64F 6
CV_16F 7
CV_8UC1 0
CV_8UC2 8
CV_8UC3 16
CV_8UC4 24
CV_8SC1 1
CV_8SC2 9
CV_8SC3 17
CV_8SC4 25
CV_16UC1 2
CV_16UC2 10
CV_16UC3 18
CV_16UC4 26
CV_16SC1 3
CV_16SC2 11
CV_16SC3 19
CV_16SC4 27
CV_32SC1 4
CV_32SC2 12
CV_32SC3 20
CV_32SC4 28
CV_32FC1 5
CV_32FC2 13
CV_32FC3 21
CV_32FC4 29
CV_64FC1 6
CV_64FC2 14
CV_64FC3 22
CV_64FC4 30
CV_16FC1 7
CV_16FC2 15
CV_16FC3 23
CV_16FC4 31
```
