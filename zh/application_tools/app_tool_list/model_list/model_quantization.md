# 模型量化

## 1. 基础概念

模型量化是一种模型压缩技术，通过降低模型参数的数值精度（如从32位浮点数转换为8位整数）来减小模型体积、加快推理速度，同时尽量保持模型精度。

### 1.1. 什么是量化

量化是将模型中的浮点数（如FP32）转换为低精度数据类型（如INT8、INT4）的过程。
主要优势：
- **减小模型体积**：INT8量化可将模型大小压缩至原来的1/4
- **加速推理**：低精度运算在硬件上执行更快
- **降低功耗**：减少内存带宽和计算量

量化的核心公式：

```
Q(x) = round(x / scale + zero_point)
```

- `scale`：缩放因子
- `zero_point`：零点偏移
- `Q(x)`：量化后的整数值

若对量化原理感兴趣，可参考以下文章：

- https://www.maartengrootendorst.com/blog/quantization/
- https://mp.weixin.qq.com/s/8ABfKytTXp78ZTOWyoT0yw

### 1.2. 动态量化和静态量化

| 特性 | 动态量化 | 静态量化 |
|------|----------|----------|
| 校准数据 | 不需要 | 需要校准集 |
| 量化时机 | 推理时动态计算scale | 推理前预先计算scale |
| 精度 | 相对较高 | 略低于动态量化 |
| 推理速度 | 较慢（需实时计算） | 较快（scale已固定） |

**动态量化**：在模型推理时，根据每层输入数据的实际分布动态计算量化参数，无需额外校准数据集。

**静态量化**：使用一批校准数据（Calibration Data）提前计算每层的量化参数（scale和zero_point），推理时直接使用预先计算好的参数。

## 2. 模型量化说明

模型量化流程通常分为以下几个阶段：

1. **选择量化方式**：根据模型类型和部署场景选择动态量化或静态量化
2. **准备校准数据**：静态量化需要准备校准数据集（通常50~500张图片）
3. **执行量化**：使用量化工具对模型进行量化转换
4. **精度验证**：对比量化前后模型的精度损失，确保在可接受范围内
5. **部署推理**：将量化后的模型部署到目标硬件进行推理

**动态量化步骤简单，在性能满足需求的情况下可以优先选择动态量化**

### 2.1. 量化工具使用说明

XSlim是SpacemiT推出的PTQ（训练后量化）量化工具，集成了已经调整好的适配芯片的量化策略，使用Json配置文件调用统一接口实现模型量化。

#### 2.1.1 安装

为避免缺少各类依赖包，建议使用的Python版本为3.10至3.11。

1）创建Python虚拟环境
```
python3 -m venv .venv
source .venv/bin/activate
```

2）在Python虚拟环境中安装XSlim及相关依赖
```
pip install xslim
```

#### 2.1.2 使用

##### 精简模型

不量化仅模型精简
```
python -m xslim -i demo.onnx -o demo.q.onnx
```

##### 动态量化

在安装好XSlim的Python虚拟环境中执行以下命令即可完成动态量化：
```
python -m xslim -i demo.onnx -o demo.q.onnx --dynq
```

##### 静态量化

静态量化需准备校准数据集、预处理脚本、量化参数配置文件。

1）准备校准数据集

准备50~500张能代表真实推理场景的数据，建议放到同一目录下。写一个校准数据列表文件img_list.txt，img_list.txt每行表示一个校准数据文件路径，可以写相对于img_list.txt 所在目录的相对路径，也可以写绝对路径，如果模型是多输入的，请确保每个文件列表的顺序是对应的。

例：
```
QuantZoo/Data/Imagenet/Calib/n01440764/ILSVRC2012_val_00002138.JPEG
QuantZoo/Data/Imagenet/Calib/n01443537/ILSVRC2012_val_00000994.JPEG
QuantZoo/Data/Imagenet/Calib/n01484850/ILSVRC2012_val_00014467.JPEG
QuantZoo/Data/Imagenet/Calib/n01491361/ILSVRC2012_val_00003204.JPEG
QuantZoo/Data/Imagenet/Calib/n01494475/ILSVRC2012_val_00015545.JPEG
QuantZoo/Data/Imagenet/Calib/n01496331/ILSVRC2012_val_00008640.JPEG
```

校准数据列表文件需要配置到量化参数配置文件的`data_list_path`字段。
```
"data_list_path": "path/to/img_list.txt"
```

2）预处理脚本文件

校准数据集需要进行预处理，且预处理后的结果应与模型推理时的输入完全一致。

预处理脚本文件custom_preprocess.py示例:

```
from typing import Sequence
import torch
import cv2
import numpy as np

def preprocess_impl(path_list: Sequence[str], input_parametr: dict) -> torch.Tensor:
    """
    读取path_list, 并依据input_parametr中的参数预处理, 返回一个torch.Tensor

    Args:
        path_list (Sequence[str]): 一个校准batch的文件列表
        input_parametr (dict): 等同于配置中的calibration_parameters.input_parametres[idx]

    Returns:
        torch.Tensor: 一个batch的校准数据
    """
    batch_list = []
    mean_value = input_parametr["mean_value"]
    std_value = input_parametr["std_value"]
    input_shape = input_parametr["input_shape"]
    for file_path in path_list:
        img = cv2.imread(file_path)
        img = cv2.resize(img, (input_shape[-1], input_shape[-2]), interpolation=cv2.INTER_AREA)
        img = img.astype(np.float32)
        img = (img - mean_value) / std_value
        img = np.transpose(img, (2, 0, 1))
        img = torch.from_numpy(img)
        img = torch.unsqueeze(img, 0)
        batch_list.append(img)
    return torch.cat(batch_list, dim=0)
```

预处理脚本文件和function需要配置到量化参数配置文件的`preprocess_file`字段。
```
"preprocess_file": "custom_preprocess.py:preprocess_impl"
```

3）配置量化参数

- JSON配置说明

```
{
    "model_parameters" : {
        "onnx_model": "", "待量化onnx模型"
        "output_prefix": "", "可缺省，输出量化模型文件的前缀"
        "working_dir": "" "可缺省，输出目录以及量化时产生文件的存放目录"
        "skip_onnxsim": false "跳过使用onnxslim化简模型，默认为false"
    },
    "calibration_parameters" : {
        "calibration_step": 100, "可缺省，限制最大的校准文件个数，默认为100"
        "calibration_device": "cpu", "可缺省，默认为cuda，自动检测，否则为cpu"
        "calibration_type": "default",  "可缺省，默认为default，可选kl、minmax、percentile、mse"
        "input_parametres": [
            {
                "input_name": "data", "可缺省，由工具从模型中读取",
                "input_shape": [1, 3, 224, 224], "可缺省，输入的shape，由工具从模型中读取"
                "dtype": "float32", "可缺省，当前输入的数据类型，由工具从模型中读取"
                "file_type": "img", "可缺省，默认为img，校准文件的类型，可以有img，npy，raw"
                "color_format": "bgr", "可缺省，默认为bgr"
                "mean_value": [103.94, 116.78, 123.68], "可缺省，默认为空"
                "std_value": [57, 57, 57], "可缺省，默认为空"
                "preprocess_file": "", "自定义预处理方法的py脚本文件，
                                        内部预设了PT_IMAGENET,IMAGENET字段直接使用"
                "data_list_path": "" "不可缺省，校准数据列表的路径"
            },
            {
                "input_name": "data1",
                "input_shape": [1, 3, 224, 224],
                "dtype": "float32",
                "file_type": "img",
                "mean_value": [103.94, 116.78, 123.68],
                "std_value": [57, 57, 57],
                "preprocess_file": "",
                "data_list_path": ""
            }
        ]
    },
    "以下均可缺省"
    "quantization_parameters": {
        "analysis_enable": true, "开启量化后分析，默认为true"
        "precision_level": 0
        "finetune_level": 1 "默认为1， 可选0， 1， 2，3"
        "max_percentile": 0.9999 "指定percentile量化时的占比"
        "custom_setting": [
            "出入边包围的全部量化算子，入边可不写常量"
            {
                "input_names": ["aaa", "bbb"],
                "output_names": ["ccc"],
                "max_percentile": 0.999,
                "precision_level": 2,
                "calibration_type": "default"
            }
        ],
        "truncate_var_names": ["/Concat_5_output_0", "/Transpose_6_output_0"] "截断模型"
    }
}
```

- 支持省略的字段

| 字段名 | 默认值 | 可选值 | 备注 |
|--------|--------|--------|------|
| output_prefix | onnx_model 的去后缀文件名，输出以 .q.onnx 结尾 | / | |
| working_dir | onnx_model 所在的目录 | / | |
| calibration_step | 100 | | 一般建议设置为 100-1000 范围 |
| calibration_device | cuda | cuda、cpu | 系统自动检测 |
| calibration_type | default | default、kl、minmax、percentile、mse | 推荐先使用 default，而后是 percentile 或 minmax |
| input_name | 从 onnx 模型中读取 | | |
| input_shape | 从 onnx 模型中读取 | | 需要 shape 为全 int，支持 batch 为符号，并默认填为 1 |
| dtype | 从 onnx 模型中读取 | float32、int8、uint8、int16 | 当前仅支持 float32 |
| file_type | img | img、npy、raw | 只支持读取与 dtype 一致的 raw 数据，即默认为 float32 |
| preprocess_file | None | PT_IMAGENET、IMAGENET | 系统预设了两种 IMAGENET 标准预处理 |
| finetune_level | 1 | 0、1、2、3 | 0：不进行任何激进的参数校准；1：可能进行一些静态量化参数校准；2+：将根据逐块量化的损失情况进行量化参数校准 |
| precision_level | 0 | 0、1、2、3、4 | 0：为全 int8 量化，即使调优也限制在 int8；1-2：只将部分算子量化为 int8，可适用于一般 Transformer 模型；3：动态量化；4：FP16 |
| max_percentile | 0.9999 | | percentile 量化时的截断范围，限制最小值为 0.99 |
| custom_setting | None | | |
| truncate_var_names | [] | | 只支持依据截断 Tensor 名将计算图二分，并且将检查二分结果，否则报错 |

- custom_setting的规则

```
"custom_setting": [
    {
        "input_names": ["input_0"],
        "output_names": ["/features/features.1/block/block.0/block.0.0/Conv_output_0"],
        "precision_level": 2
     }
  ]
```

工具采用计算图出入边包围子图的方式捕获一个子图并设置自定义量化参数，如下图示例，我们希望将计算图红框中的算子调整为precision_level=2的精度模式，则需要先确定当前子图的所有非常量出入边，即入边为子图内首个Conv的输入n2，Add的旁支输入n7，出边为Add的输出y

![truncate_var_names示例](image/custom_setting1.png)
![truncate_var_names示例](image/custom_setting2.png)

即产生如下的配置
```
"custom_setting": [
     {
         "input_names": ["n2", "n7"],
         "output_names": ["y"],
         "precision_level": 2
     }
  ]
```

- truncate_var_names的规则

truncate_var_names支持将完整的带有后处理层的ONNX模型送入量化工具，量化产出模型也会保留后处理结构，但需要使用者指定模型主结构与后处理结构的分离点，即truncate_var_names

![truncate_var_names示例](image/truncate_var_names.png)

例如yolov6p5_n模型，只需要指定Sigmoid、Concat（红框）算子的输出即可将模型二分，只量化上半部分。

- 精简配置示例

quant.json
```
{
    "model_parameters": {
        "onnx_model": "yolov11n_320x320.onnx"
    },
    "calibration_parameters": {
        "input_parametres": [
            {
                "color_format": "bgr",
                "preprocess_file": "custom_preprocess.py:preprocess_impl"
                "data_list_path": "/home/user/code/Coco/calib_img_list.txt"
            }
        ]
    },
    "quantization_parameters": {
        "truncate_var_names": [
            # 需要netron找算子，把计算图截断成上下两部分，后半部分不量量化softmax不建议量化
            "/model.23/Mul_2_output_0",
            "/model.23/Sigmoid_output_0"
        ]
    }
}
```

4）执行静态量化命令

```
python -m xslim --config ./quant.json
```

5）量化输出

- 量化后的模型

.q.onnx后缀的量化模型

- 量化分析文件

开启analysis_enable后，将在输出目录下生成量化分析文件，以markdown文件形式呈现

![q_report示例](image/q_report.png)

SNR高于0.1、Cosine小于0.99的输出将被标记，如果某个模型标记输出过多，则可能产生量化误差，Cosine低并不一定产生量化误差，SNR的可信度更高；Constant标记的节点可以无视，这些节点被数值优化过，可能存在明显差异。

### 2.2. 量化损失测试

量化精度损失可通过以下两种方式确认：

#### 方式一：查看量化分析文件（快速确认）

执行量化时默认开启 `analysis_enable`，量化完成后会在输出目录生成量化分析报告（.md文件）。

通过报告中的 SNR 和 Cosine 两个指标初步判断量化质量：

| 指标 | 异常阈值 | 说明 |
|------|---------|------|
| SNR（噪声/信号比） | **> 0.1** | 噪声超过信号的10%，值越小越好 |
| Cosine（余弦相似度） | **< 0.99** | 输出方向偏差过大，值越接近1越好 |

> - 标记层过多 → 说明整体量化误差较大，需要调整量化参数
> - 仅 Cosine 低、SNR 正常 → 通常可接受，**SNR 可信度更高**

#### 方式二：业务指标对比（精确验证）

通过对比量化前后模型在验证集上的业务指标，确认精度损失是否在可接受范围内。

以目标检测为例，通过对比量化前后 mAP50（mean Average Precision，IoU 阈值为 0.5 时所有类别的平均精度）确认精度损失。

## 3. 实战举例

TBD

### 3.1. yolo模型量化

静态量化yolov11n_320x320.onnx。

1）安装量化环境

参考“模型量化说明”。

2）下载模型

```
wget https://archive.spacemit.com/spacemit-ai/BRDK/Model_Zoo/CV/YOLOv11/yolov11n_320x320.onnx
```

3）准备校准数据集

```
wget https://archive.spacemit.com/spacemit-ai/BRDK/Model_Zoo/Datasets/Coco/Coco.tar.gz
tar -zxvf Coco.tar.gz
```

4）准备预处理文件

preprocess.py
```
from typing import Sequence
import torch
import cv2
import numpy as np

def preprocess_impl(path_list: Sequence[str], input_parametr: dict) -> torch.Tensor:
    batch_list = []
    mean_value = input_parametr["mean_value"]
    std_value = input_parametr["std_value"]
    input_shape = input_parametr["input_shape"]


    for file_path in path_list:
        img = cv2.imread(file_path)
        img = cv2.resize(img, (input_shape[-1], input_shape[-2]), interpolation=cv2.INTER_AREA)
        img = img / 255
        img = img.astype(np.float32)
        img = np.transpose(img, (2, 0, 1))
        img = torch.from_numpy(img)
        img = torch.unsqueeze(img, 0)
        batch_list.append(img)
```

5）配置量化参数

quant.json
```
{
    "model_parameters": {
        "onnx_model": "yolov11n_320x320.onnx"
    },
    "calibration_parameters": {
        "input_parametres": [
            {
                "color_format": "bgr",
                "data_list_path": "/home/user/code/Coco/calib_img_list.txt"
            }
        ]
    },
    "quantization_parameters": {
        "truncate_var_names": [
            # 需要netron找算子，把计算图截断成上下两部分，后半部分不量量化softmax不建议量化
            "/model.23/Mul_2_output_0",
            "/model.23/Sigmoid_output_0"
        ]
    }
}
```

6）执行量化命令

```
python -m xslim --config ./quant.json
```

### 3.2. 大模型量化

TBD

### 3.3. TTS量化

TBD

## 4. FAQ

TBD
