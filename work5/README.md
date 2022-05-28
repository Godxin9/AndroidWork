# 实验5——TensorFlow Lite 模型生成

## 1.预备工作

首先安装程序运行必备的一些库。

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ tflite-model-maker
```

![image-20220527174839419](C:/Users/Godxin/AppData/Roaming/Typora/typora-user-images/image-20220527174839419.png)

发现报错了

解决问题：

1. 进入`E:\Anaconda3\Lib\site-packages`找到llvmlite直接删除

![image-20220527175024506](%E5%AE%9E%E9%AA%8C5%E2%80%94%E2%80%94TensorFlow%20Lite%20%E6%A8%A1%E5%9E%8B%E7%94%9F%E6%88%90.assets/image-20220527175024506.png)

2. 打开Anaconda的Environments搜索llvmlite进行remove
   ![image-20220527175117833](%E5%AE%9E%E9%AA%8C5%E2%80%94%E2%80%94TensorFlow%20Lite%20%E6%A8%A1%E5%9E%8B%E7%94%9F%E6%88%90.assets/image-20220527175117833.png)

3. 重新install即可

![image-20220527180815757](%E5%AE%9E%E9%AA%8C5%E2%80%94%E2%80%94TensorFlow%20Lite%20%E6%A8%A1%E5%9E%8B%E7%94%9F%E6%88%90.assets/image-20220527180815757.png)

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ anaconda-project==0.10.1
```

导入相关库

```
import os

import numpy as np

import tensorflow as tf
assert tf.__version__.startswith('2')

from tflite_model_maker import model_spec
from tflite_model_maker import image_classifier
from tflite_model_maker.config import ExportFormat
from tflite_model_maker.config import QuantizationConfig
from tflite_model_maker.image_classifier import DataLoader

import matplotlib.pyplot as plt
```

## 2、模型训练

1. 获取数据

   本实验先从较小的数据集开始训练，当然越多的数据，模型精度更高。这里从`storage.googleapis.com`中下载了本实验所需要的数据集。`image_path`可以定制，默认是在用户目录的`.keras\datasets`中。

   ```
   image_path = tf.keras.utils.get_file(
         'flower_photos.tgz',
         'https://storage.googleapis.com/download.tensorflow.org/example_images/flower_photos.tgz',
         extract=True)
   image_path = os.path.join(os.path.dirname(image_path), 'flower_photos')
   ```

   

2. 运行
   **第一步：加载数据集，并将数据集分为训练数据和测试数据。**

```
data = DataLoader.from_folder(image_path)
train_data, test_data = data.split(0.9)
print(train_data.size)
print(test_data.size)
```

```
INFO:tensorflow:Load image with size: 3670, num_label: 5, labels: daisy, dandelion, roses, sunflowers, tulips.
3303
367
```

**第二步：训练Tensorflow模型**

```
inception_v3_spec = image_classifier.ModelSpec(uri='D:\Workspace\JupyterNotebookFiles\E5\efficientnet_lite0_feature-vector_2')
# inception_v3_spec = image_classifier.ModelSpec(uri='https://storage.googleapis.com/tfhub-modules/tensorflow/efficientnet/lite0/feature-vector/2.tar.gz')
inception_v3_spec.input_image_shape = [240, 240]
model = image_classifier.create(train_data, model_spec=inception_v3_spec)
# 使用默认模型
# model = image_classifier.create(train_data)
```

```
INFO:tensorflow:Retraining the models...
Model: "sequential"
_________________________________________________________________
 Layer (type)                Output Shape              Param #   
=================================================================
 hub_keras_layer_v1v2 (HubKe  (None, 1280)             3413024   
 rasLayerV1V2)                                                   
                                                                 
 dropout (Dropout)           (None, 1280)              0         
                                                                 
 dense (Dense)               (None, 5)                 6405      
                                                                 
=================================================================
Total params: 3,419,429
Trainable params: 6,405
Non-trainable params: 3,413,024
_________________________________________________________________
None
Epoch 1/5


E:\Anaconda3\lib\site-packages\keras\optimizers\optimizer_v2\gradient_descent.py:108: UserWarning: The `lr` argument is deprecated, use `learning_rate` instead.
  super(SGD, self).__init__(name, **kwargs)


103/103 [==============================] - 62s 582ms/step - loss: 0.8861 - accuracy: 0.7609
Epoch 2/5
103/103 [==============================] - 61s 587ms/step - loss: 0.6605 - accuracy: 0.8908
Epoch 3/5
103/103 [==============================] - 79s 771ms/step - loss: 0.6231 - accuracy: 0.9135
Epoch 4/5
103/103 [==============================] - 64s 617ms/step - loss: 0.6060 - accuracy: 0.9235
Epoch 5/5
103/103 [==============================] - 60s 578ms/step - loss: 0.5943 - accuracy: 0.9320
```

**第三步：评估模型**

```
loss, accuracy = model.evaluate(test_data)
```

```
12/12 [==============================] - 8s 546ms/step - loss: 0.5981 - accuracy: 0.9210
```

**第四步，导出Tensorflow Lite模型**

```
model.export(export_dir='.')
```

```
INFO:tensorflow:Assets written to: C:\Users\ll\AppData\Local\Temp\tmpqryprhqv\assets


INFO:tensorflow:Assets written to: C:\Users\ll\AppData\Local\Temp\tmpqryprhqv\assets
d:\anaconda3\lib\site-packages\tensorflow\lite\python\convert.py:746: UserWarning: Statistics for quantized inputs were expected, but not specified; continuing anyway.
  warnings.warn("Statistics for quantized inputs were expected, but not "


INFO:tensorflow:Label file is inside the TFLite model with metadata.


INFO:tensorflow:Label file is inside the TFLite model with metadata.


INFO:tensorflow:Saving labels in C:\Users\ll\AppData\Local\Temp\tmpjyprgcyp\labels.txt


INFO:tensorflow:Saving labels in C:\Users\ll\AppData\Local\Temp\tmpjyprgcyp\labels.txt


INFO:tensorflow:TensorFlow Lite model exported successfully: .\model.tflite


INFO:tensorflow:TensorFlow Lite model exported successfully: .\model.tflite

```

这里导出的Tensorflow Lite模型包含了元数据(metadata),其能够提供标准的模型描述。导出的模型存放在Jupyter Notebook当前的工作目录中。

可以看到到出的文件已经出现。

![image-20220527181420630](%E5%AE%9E%E9%AA%8C5%E2%80%94%E2%80%94TensorFlow%20Lite%20%E6%A8%A1%E5%9E%8B%E7%94%9F%E6%88%90.assets/image-20220527181420630.png)