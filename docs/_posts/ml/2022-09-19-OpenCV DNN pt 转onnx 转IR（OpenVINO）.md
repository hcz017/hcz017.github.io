---
date: 2020-09-18 21:53
status: public
title: 'OpenCV DNN pt 转onnx 转IR（OpenVINO）'
---

Open Neural Network Exchange（ONNX，开放神经网络交换）

# pt 转onnx
举例
```python
import torch
import os

save_path = "train/checkpoints/FeatherNet54"

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

model_path=os.path.join(save_path, "2020-09-15-12_01_14__10_best.pth")
model = torch.load(model_path, map_location=device)
model.eval()
batch_size = 1
input_shape = (3, 224, 224)

input_data_shape = torch.randn(batch_size, *input_shape, device=device)

save_onnx_dir = os.path.join(save_path, "best.pth.onnx")
torch.onnx.export(model, input_data_shape, save_onnx_dir, verbose=True)

```

# pt checkpoint 转onnx
用checkpoint 转onnx 依赖训练时参数，不上代码了。

参考：

https://pytorch.org/tutorials/beginner/saving_loading_models.html#saving-loading-a-general-checkpoint-for-inference-and-or-resuming-training

https://pytorch.org/tutorials/recipes/recipes/saving_and_loading_a_general_checkpoint.html

# onnx 转IR
需要提前安装的依赖包
```bash
pip install networkx
pip install defusedxml
```
安装转换所需的依赖包
```bash
cd /opt/intel/openvino/deployment_tools/model_optimizer/install_prerequisites
sudo ./install_prerequisites_onnx.sh
```
转换
Cmd至打开安装好的OpenVINO: deployment_tools\model_optimizer
目录下，执行下面的命令行语句：
```bash
cd /opt/intel/openvino/deployment_tools/model_optimizer
python mo_onnx.py --input_model D:\python\pytorch_tutorial\resnet18.onnx
```
可以用 --output_dir 指定输出目录，更多选项可以查看源码


# 参考链接
1. https://cloud.tencent.com/developer/article/1659615
2. https://cloud.tencent.com/developer/article/1677340
3. https://zhuanlan.zhihu.com/p/116065374
4. https://docs.openvinotoolkit.org/2020.1/_docs_install_guides_installing_openvino_linux.html

