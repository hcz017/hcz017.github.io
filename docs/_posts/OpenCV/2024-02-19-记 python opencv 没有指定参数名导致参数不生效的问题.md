---
Date: 2024-02-19
tags:
  - OpenC
  - remap
  - boardMode
title: 记 python opencv 没有指定参数名导致参数不生效的问题
---

**省流**：在使用opencv remap 函数时，需要明确指定参数名才能正确应用参数。

在验证OpenCV remap 函数时，有一个参数的含义是复制边缘像素（BORDER_REPLICATE），也就是在无效像素区域重复复制有效像素的边缘，看起来有点像拉丝一样的效果。恰巧有一份 C++ 的代码用的就是这个参数，我在将它写成python 版本时却发现得不到一样的结果。
C++ 结果：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/other/_image/boardMode_1.jpg)

python 版本结果：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/other/_image/boardMode_0.jpg)

可以看到左上边界是黑边，没有填充颜色。
关键代码：

```python
output_image = cv2.remap(input_image, map_x, map_y, cv2.INTER_CUBIC, cv2.BORDER_REPLICATE)  
```

起初我以为是 map 表里面元素的问题，在几乎将两个 map 表调成一样之后发现效果还是没变。直到我看到了另一份代码可以正确生成复制像素的效果我发现只要指定参数名就可以了！

```python
output_image = cv2.remap(input_image, map_x, map_y, interpolation=cv2.INTER_CUBIC, borderMode=cv2.BORDER_REPLICATE)
```

但是不指定参数名程序运行不会报错，我也就一直没发现问题。
python 代码指定参数名之后的结果：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/other/_image/boardMode_1.jpg)
