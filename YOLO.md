# 概述

## 安装

```bash
pip install ultralytics
```

# 使用

## 1.加载模型

> 可以加载官方提供的已经得到部分训练的模型，可以直接从网络下载

```python
from ultralytics import YOLO

m1 = YOLO('yolov8n.pt')
```

> 注意:
>
> - 如果路径没有yolov8n.pt，会自动从网络下载模型

## 2.检测目标

> 将需要检测的文件作为参数传入模型进行检测

```python
m1('1.jpg', show=True,save=True)
```

> 注意：
>
> - show参数控制是否展示训练中的图片或视频，save参数控制是否保存检测后的图片或视频

# 训练模型

## 1.准备数据集

## 2.编写配置文件

```yaml
path: E:\Python\PythonProject\YOLOStudy\MyImages
train: images\train
val: images\val
nc: 4
names: ['马叔', '彪哥', '玉芬', '桂英']
```

## 3.加载预训练模型

```
from ultralytics import YOLO

m1 = YOLO('yolov8n.pt')

```

## 4.开始训练模型

```
m1.train(
    data='data.yaml',
    epochs=500, //训练轮次，推荐300~500
    imgsz=640,	//输入图片的尺寸，会将输入的图片自动转换为这个尺寸，推荐640
    batch=32,	//训练的批量，推荐16或32
    device='cpu', //训练使用的装置，使用gpu写0，使用cpu写'cpu'
    )
```

> 注意：
>
> - 模型训练完成后，会保留两个模型，一个是最优模型(适用于应用)，一个是最后一次训练完成后的模型(适用于继续训练)

# 小知识

## 标准图片数据集结构

![image-20250515164653368](D:\笔记\图片\image-20250515164653368.png)

### yolo标注格式

1. 每个图像对应一个 `.txt` 文本文件，文件名与图像文件名一致

2. 每个标注文件中的每一行对应图像中的一个目标（物体），格式为：

   ```
   <class_id> <x_center> <y_center> <width> <height>
   ```

   - **`<class_id>`**: 类别索引（整数，从 `0` 开始）。
   - **`<x_center>`**: 目标中心点的 x 坐标（相对于图像宽度的归一化值，范围 `0-1`）。
   - **`<y_center>`**: 目标中心点的 y 坐标（相对于图像高度的归一化值，范围 `0-1`）。
   - **`<width>`**: 目标的宽度（相对于图像宽度的归一化值，范围 `0-1`）。
   - **`<height>`**: 目标的高度（相对于图像高度的归一化值，范围 `0-1`）。

3. 一张图像有多个目标时，通过换行隔开。

## 当前屏幕检测