****# TensorRT 菜谱

## 总体介绍

+ 本仓库面向 NVIDIA TensorRT 初学者和开发者，提供了 TensorRT 相关学习资料和参考资料、丰富的代码范例

+ 创建 docker container 之后可以参考 requirements.txt 自选安装需要的库 `pip install -r requirement.txt`

+ 有用的链接
  + [Link](https://developer.nvidia.com/nvidia-tensorrt-download) 下载

  + [Link](https://docs.nvidia.com/deeplearning/tensorrt/release-notes/index.html) 版本特性介绍

  + [Link](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html) 文档

  + [Link](https://docs.nvidia.com/deeplearning/tensorrt/archives/index.html) 归档文档

  + [Link](https://docs.nvidia.com/deeplearning/tensorrt/support-matrix/index.html) 支持特性列表

  + [Link](https://docs.nvidia.com/deeplearning/tensorrt/api/c_api) C++ 接口文档

  + [Link](https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/) python 接口文档

  + [Link](https://developer.nvidia.com/zh-cn/blog/author/ken-he/) 何琨的博客，包含 TensorRT 的介绍（中文）

## The update of the repository 仓库更新变化

+ **2022年9月15日**，开始引入 TensorRT 8.5 的新特性范例代码，预计 TensorRT 8.5 发布后 cookbook 切换到 trt8.5 分支

+ **2022年9月7日**，添加 *testAll.sh* 用于运行所有范例代码并生成相应的输出结果

+ **2022年8月21日**，添加本仓库用到的资源链接（MNIST 数据集和 Hackathon2022 的初赛赛题模型 ONNX 文件），链接：[Link](https://pan.baidu.com/s/14HNCFbySLXndumicFPD-Ww) ，提取码：gpq2

+ **2022年8月4日** 常用的 docker image 表格。注意 pyTorch 和 TensorFlow1 包含 NVIDIA 的改动（尤其是 QAT 相关内容），与单独使用 *pip* 安装的版本有差异

|            Name of Docker Image             | python |  CUDA   |  cuDNN   | TensorRT |        Framework         |           Comment           |
| :-----------------------------------------: | :----: | :-----: | :------: | :------: | :----------------------: | :-------------------------: |
|    **nvcr.io/nvidia/tensorrt:19.12-py3**    |  3.6.9   | 10.2.89 |  7.6.5   |  6.0.1   |        TensorRT 6        | Last version of TensorRT 6  |
|    **nvcr.io/nvidia/tensorrt:21.06-py3**    | 3.8.5  | 11.3.1  |  8.2.1   | 7.2.3.4  |        TensorRT 7        | Last version of TensorRT 7  |
|    **nvcr.io/nvidia/tensorrt:22.04-py3**    | 3.8.10 | 11.6.2  | 8.4.0.27 | 8.2.4.2  |       TensorRT 8.2       | Last version of TensorRT8.2 |
|    **nvcr.io/nvidia/tensorrt:22.07-py3**    | 3.8.13 | 11.7U1  |  8.4.1   |  8.4.1   |       TensorRT 8.4       |       TensorRT8.4-GA        |
| **nvcr.io/nvidia/tensorflow:22.07-tf1-py3** | 3.8.13 | 11.7U1  |  8.4.1   |  8.4.1   |    TensorFlow 1.15.5     |      TensorFlow 1 LTS       |
| **nvcr.io/nvidia/tensorflow:22.07-tf2-py3** | 3.8.13 | 11.7U1  |  8.4.1   |  8.4.1   |     TensorFlow 2.9.1     |                             |
|    **nvcr.io/nvidia/pytorch:22.07-py3**     | 3.8.13 | 11.7U1  |  8.4.1   |  8.4.1   | pyTorch 1.13.0a0+08820cb |                             |
|  **nvcr.io/nvidia/paddlepaddle:22.07-py3**  | 3.8.13 | 11.7U1  |  8.4.1   |  8.4.1   |    PaddlePaddle 2.3.0    |                             |

+ **2022年7月15日** Cookbook 内容随 TensorRT 更新到 8.4 GA 版本，同学们使用较旧版本的 TensorRT 运行 Cookbook 中的范例代码时，可能需要修改部分代码才能运行，例如：
  + 调整 `config.set_memory_pool_limit(trt.MemoryPoolType.WORKSPACE, 1 << 30)` 为 `config.max_workspace_size = 1 << 30`

---

## 各部分内容介绍

## 00-MNISTData -- 用到的数据集

+ Cookbook 用到的 MNIST 数据集，运行其他范例代码前，需要下载到本地并做一些预处理

+ 数据集可以从这里 [Link](http://yann.lecun.com/exdb/mnist/) 或这里 [Link](https://storage.googleapis.com/cvdf-datasets/mnist/) 或百度网盘 [Link](https://pan.baidu.com/s/14HNCFbySLXndumicFPD-Ww)(提取码：gpq2) 下载得到

+ 数据集下载后放进本目录，形如 *00-MNISTData/*.gz*，一共 4 个文件

+ 运行下列命令，提取 XXX 张训练图片到 ./train，YYY 张图片到 ./test（数据及包含 60000 张训练图片和 10000 张测试图片，无参数时默认提取 3000 张训练图片和 500 张测试图片，提取的图片为 JPEG 格式）

```shell
python3 extractMnistData.py XXX YYY
```

+ 该目录下还有一张从数据集的测试集中提取的 *8.png*，单独作为 TensorRT 的推理输入数据使用

---

## 01-SimpleDemo -- 使用 TensorRT 的一个完整程序的范例

+ 使用 TensorRT 的基本步骤，包括网络搭建、引擎构建、序列化与反序列化、推理计算等

+ 范例涵盖不同版本的 TensorRT，均包含 C++ 和 python 的等价实现

### TensorRT6

+ 采用 **TensorRT6** + **Implicit Batch** 模式 + **Static Shape** 模式 + **Builder API** + **pycuda** 库

+ **Implicit Batch** 模式仅提供前向兼容性支持，可能缺少一些新特性和性能优化方面的支持，在较新版本的 TensorRT 请尽量使用 Explicit Batch 模式

+ **Builder API** 已被弃用，在较新版本 TensorRT 中请使用 BuilderConfig API

+ **pycuda** 库比较旧，可能存在一些线程安全以及与其他机器学习框架中 CUDA Context 交互方面的问题，已经不推荐使用，较新版本 TensorRT 推荐使用 cuda-python 库

### TensorRT7

+ 采用 **TensorRT7** + **Implicit Batch** 模式 + **Static Shape** 模式 + **Builder API** + **cuda-python** 库的 Runtime API

### TensorRT8

+ 采用 **TensorRT8** + **Explicit Batch** 模式 + **Dynamic Shape** 模式 + **BuilderConfig API** + **cuda-python** 库的 Driver API / Runtime API 双版本

### TensorRT8.4

+ 基本同 TensorRT8，使用了 TensorRT8.4 中新引入的 API，python 代码仅保存了 cuda-python 库的 Runtime API 版本

### TensorRT8.5

+ 基本同 TensorRT8.4，使用了 TensorRT8.5 中新引入的 API，主要是 CudaEngine 和 ExecutionContext 的 Binding 相关的 API 有较大改动

---

## 02-API -- TesnorRT 核心类和 API 用法范例

+ TensorRT 一些比较重要的对象的 API 的用法，主要是展示其作用及其参数的效果，在实际工作中按需取用就好

### Builder

+ **Builder** 类的通用成员和方法

### BuilderConfig

+ **uilderConfig** 类的通用成员和方法

### CudaEngine

+ **CudaEngine** 类的通用成员和方法

### ExecutionContext

+ **ExecutionContext** 类的通用成员和方法

### Int8-PTQ

+ 使用 TensorRT INT8-PTQ 模式的一般流程，以及涉及到的各种类的相关方法

+ 实际模型中使用 INT8-PTQ 模式的范例见 03-APIModel/MNISTExample-\*，各种模型导出到 TensorRT 时要做的工作基本没有差别

### Int8-QDQ

+ TensorRT 中包含 Quantize 和 Dequantize 层的网络时范例

+ 实际模型中使用 INT8-QAT 模式的范例见 04-API/*-QAT，涉及到模型训练框架中的修改，但是 TensorRT 中要做的工作是差不多的

### Layer

+ **Layer** 类的通用成员和方法

+ 各 Layer 的用法及其参数的范例，无特殊情况均采用 TensorRT8 + Explicit Batch 模式

+ 各 \*Layer/\*.md 详细记录了所有 Layer 参数的用法，配合各 \*Layer/\*.py 范例代码，展示各参数作用下的输出结果和算法解释

+ GitLab/Github 的 markdown 不支持插入 Latex，所以各 \*Layer/\*.md 在线预览时公式部分无法渲染，可以下载后使用支持 Latex 的 markdown 编辑软件（如 Typora）来查看。同时，目录中也提供了各 .md 的 PDF 文件版本（位于 50-Resource/Layer）可以直接查看

+ 各 Layer 维度支持列表 [Link](https://docs.nvidia.com/deeplearning/tensorrt/support-matrix/index.html#layers-matrix)

+ 各 Layer 精度支持列表 [Link](https://docs.nvidia.com/deeplearning/tensorrt/support-matrix/index.html#layers-precision-matrix)

+ 各 Layer 流控制支持列表 [Link](https://docs.nvidia.com/deeplearning/tensorrt/support-matrix/index.html#layers-flow-control-constructs)

### Network

+ **Network** 类的通用成员和方法

### PrintNetwork

+ 打印 TensorRT 自动优化前的网络的逐层信息

+ 想要打印 TensorRT 自动优化后的 Serialized Network（Engine）逐层信息可参考 09-Advance/EngineInspector

### Tensor

+ **Tensor** 类的通用成员和方法

---

## 03-APIModel -- 使用 API 搭建模型范例

+ 以一个完整的、基于 MNIST 数据集的、手写数字识别模型为例，展示了使用 API 逐层搭建的方式在 TensorRT 中重建来自各种机器学习框架中已训练好模型的过程，包括原模型权重提取，TensorRT 中相应模型的搭建和权重加载

+ 所有基于 MNIST 数据集的范例均可在 TensorRT 中开启 FP16 模式或 INT8-PTQ 模式

### MNISTExample-Paddlepaddle

+ 使用 Paddlepaddle 框架训练手写数字识别模型，并在 TensorRT 中重建模型和进行推理

### MNISTExample-pyTorch

+ 使用 pyTorch 框架训练手写数字识别模型，并在 TensorRT 中重建模型和进行推理

+ 本例包含使用 TensorRT C++ API 重新搭建模型并运行推理过程的范例

### MNISTExample-TensorFlow1

+ 使用 TensorFlow1 框架训练手写数字识别模型，并在 TensorRT 中重建模型和进行推理

### MNISTExample-TensorFlow2

+ 使用 TensorFlow2 框架训练手写数字识别模型，并在 TensorRT 中重建模型和进行推理

### TypicalaPI-Paddlepaddle[TODO]

+ Paddlepaddle 中各种典型结构的不同实现在 TensorRT 中重建的范例

### TypicalaPI-pyTorch[TODO]

+ pyTorch 中各种典型结构的不同实现在 TensorRT 中重建的范例

### TensorFlow1

+ TensorFlow1 中各种典型结构的不同实现在 TensorRT 中重建的范例

+ 目前包含卷积、全连接、LSTM 结构

### TensorFlow2[TODO]

+ TensorFlow2 中各种典型结构的不同实现在 TensorRT 中重建的范例

---

## 04-Parser -- 使用 Parser 转换模型范例

+ 以一个完整的、基于 MNIST 数据集的、手写数字识别模型为例，展示了使用 Parser 转化方式在 TensorRT 中重建来自各种机器学习框架中已训练好模型的过程，包括原模型转为 ONNX 中间格式（.onnx 文件）和在 TensorRT 中解析

+ Onnx 的算子说明 [Link](https://github.com/onnx/onnx/blob/main/docs/Operators.md)

+ TensorRT 对 Onnx 算子的支持列表 [Link](https://github.com/onnx/onnx-tensorrt/blob/main/docs/operators.md)

+ 所有基于 MNIST 数据集的范例均可在 TensorRT 中开启 FP16 模式或 INT8-PTQ 模式（已启用 INT8-QAT 模式的除外）

### Paddlepaddle-ONNX-TensorRT

+ 在 Paddlepaddle 中训练模型并导出为 .onnx，然后 TensorRT 解析该 .onnx、构建引擎和执行推理

### pyTorch-ONNX-TensorRT

+ 在 pyTorch 中训练模型并导出为 .onnx，然后 TensorRT 解析该 .onnx、构建引擎和执行推理

### pyTorch-ONNX-TensorRT-QAT

+ 在 pyTorch 中用量化感知训练（Quantization Aware Training，QAT）训练模型并导出为 .onnx，然后 TensorRT 解析该 .onnx、构建引擎和执行推理

+ 参考 [Link](https://github.com/NVIDIA/TensorRT/tree/main/tools/pytorch-quantization)

+ 原始例子中，模型的校正和精调过程要跑在镜像 *nvcr.io/nvidia/pytorch:20.08-py3* 上，而导出 ONNX 的部分却要跑在镜像 *nvcr.io/nvidia/pytorch:21.12-py3* 及之后的版本上。原因是模型运行依赖于较旧镜像中的文件 */opt/pytorch/vision/references/classification/train.py*，该文件在较新镜像中发生了很大的变动，同时较旧镜像中的 *torch.onnx* 模块不支持 QAT 导出

+ 本例子使用了完全本地化的模型，移除了上述依赖，可以独立运行

+ 需要安装 pytorch-quantization 库

### TensorFlow1-Caffe-TensorRT

+ **该 Workflow 已废弃，本范例仅做参考**

+ 在 TensorFlow1 中训练模型并导出为 .ckpt，接着转为 .prototxt 和 .caffemodel，然后 TensorRT 解析该文件、构建引擎和执行推理

### TensorFlow1-ONNX-TensorRT

+ 在 TensorFlow1 中训练模型并导出为 .pb，接着转为 .onnx，然后 TensorRT 解析该 .onnx、构建引擎和执行推理

### TensorFlow1-ONNX-TensorRT-QAT

+ 在 TensorFlow1 中用量化感知训练模型并导出为 .pb，接着转为 .onnx，然后 TensorRT 解析该 .onnx、构建引擎和执行推理

+ 参考 [Link](https://github.com/shiyongming/QAT_demo)

### TensorFlowF1-UFF-TensorRT

+ **该 Workflow 已废弃，本范例仅做参考**

+ 在 TensorFlow1 中训练模型并导出为 .pb，接着转为 .uff，然后 TensorRT 解析该 .uff、构建引擎和执行推理

### TensorFlow2-ONNX-TensorRT

+ 在 TensorFlow2 中训练模型并导出为 .pb 或多个文件，接着转为 .onnx，然后 TensorRT 解析该 .onnx、构建引擎和执行推理

### TensorFlow2-ONNX-TensorRT-QAT[TODO]

+ 在 TensorFlow2 中用量化感知训练模型并导出为 .pb 或多个文件，接着转为 .onnx，然后 TensorRT 解析该 .onnx、构建引擎和执行推理

---

## 05-Plugin -- 自定义插件书写

+ 实现自定义插件并运用到 TensorRT 推理中的范例

+ 最核心的范例是 **usePluginV2DynamicExt**，它代表了 TensorRT 8 中最主流 Plugin 的最基础用法，所有其他范例均以该范例为基础展开，同学们学习 Plugin 可以先看这个范例

+ 每个范例下均有 **test\*Plugin.py** 单元测试文件，推荐同学们在完成 Plugin 的书写后一定要进行单元测试，再集成到整个模型中

### loadNpz

+ 从 .npz 文件中读取数据并用于 Plugin 中

+ 范例展示了在 C++ 中使用 cnpy 库读取 .npz 文件的方法

+ cnpy 的原始仓库 [Link](https://github.com/rogersce/cnpy)

### multipleVersion

+ 书写和使用同一个 Plugin 的不同版本

+ 使用 TensorRT 内建的 Plugin 时也需要如此确认 Plugin 的版本

### PluginPrecess

+ 使用多 Optimization Profile 的情境下，一个含有 Plugin 的网络中 Plugin 的各成员函数调用顺序

+ 该示例告诉我们在每个步骤（构建步骤或运行时步骤）上调用成员方法的顺序

### PluginReposity

+ 常见 Plugin 小仓库

+ 各 Plugin 仅保证计算结果正确，不保证性能最优化

+ 部分 Plugin 的不同版本包含不同的功能，这里都in行了保留

+ 后缀 "-TRT8" 的 Plugin 表示已经对齐 TensorRT 8 的 Plugin 格式要求，其他很多 Plugin 基于 TensorRT6 或 TensorRT7 书写，在更新版本的 TensorRT 上编译时需要修改部分成员函数（范例 *usePluginV2Ext/AddScalarPlugin.h* 和 *usePluginV2Ext/AddScalarPlugin.cu* 中写到了新旧 TensorRT Plugin 中成员方法的区别）.

### useCuBLAS

+ 在 Plugin 中使用 cuBLAS 计算矩阵乘法的范例

+ 内含一个 useCuBLASAlone.cu 范例，为单独使用 cuBLAS 计算 GEMM 的例子

### useFP16

+ 在 Plugin 中使用 float16 数据类型，功能同 usePluginV2DynamicExt

+ Steps to run

### useINT8-PTQ

+ 在 Plugin 中使用 int8 数据类型，功能同 usePluginV2DynamicExt

+ 注意输入输出的数据排布可能不是 Linear 型的

### useINT8-QDQ

+ 在 Plugin 中使用 int8 数据类型，功能同 usePluginV2DynamicExt

+ 注意输入输出的数据排布可能不是 Linear 型的

### usePluginV2DynamicExt

+ 使用 IPluginV2DynamicExt 类实现一个 Plugin，功能是给输入张量所有元素加上同一个标量值，然后输出

+ 特点：
  + 成员函数的声明对齐 TensorRT8 的要求
  + 使用 Explicit Batch 模式 + Dynamic Shape 模式
  + 输入张量形状可变（相同维度不同形状的输入共享同一个 TensorRT engine）


### usePluginV2Ext

+ 使用 IPluginV2Ext 类实现一个 Plugin，功能是给输入张量所有元素加上同一个标量值，然后输出
+ 特点：
  + 使用 Implicit Batch 模式
  + 输入张量形状不可变（不同形状的输入要建立不同的 TensorRT engine）
  + 标量加数在构建期确定（多次 inference 之间不能更改）
  + 支持序列化和反序列化
  + 支持 TensorRT7 和 TensorRT8 版本（但是需要修改 AddScalarPlugin.h 和 AddScalarPlugin.cu 中 enqueue 函数的声明和定义）

### usePluginV2IOExt

+ 使用 IPluginV2IOExt 类实现一个 Plugin，功能是给输入张量所有元素加上同一个标量值，然后输出
+ 特点：
  + 使用 Implicit Batch 模式
  + 支持各输出张量拥有不同的数据类型和数据排布（本范例中只有一个输出张量，没有体现）

---

## 06-PluginAndParser -- 结合使用 Parser 与 Plugin 的范例

+ 结合使用 Paser 和 Plugin 来转化模型并在 TensorRT 中推理

+ 需要用到 onnx-graphsurgeon，可参考 08-Tool/onnxGraphSurgeon

### pyTorch-FailConvertNonZero

+ 在 pyTorch 训练的模型中用到了 NonZero 算子（TensorRT 8.4 还不原生支持），其导出的 .onnx 用 TensorRT 解析时失败报错的例子

### pyTorch-LayerNorm

+ pyTorch 模型转 ONNX 转 TensorRT 的过程中，替换一个 Normalization Plugin

### TensorFlow1-AddScalar

+ TensorFlow1 模型转 ONNX 转 TensorRT 的过程中，替换一个 AddScalar Plugin

### TensorFlow1-LayerNorm

+ TensorFlow1 模型转 ONNX 转 TensorRT 的过程中，替换一个 Layer Normalization Plugin

### TensorFlow2-AddScalar

+ TensorFlow2 模型转 ONNX 转 TensorRT 的过程中，替换一个 AddScalar Plugin

### TensorFlow2-LayerNorm

+ TensorFlow2 模型转 ONNX 转 TensorRT 的过程中，替换一个 Layer Normalization Plugin

---

## 07-FameworkTRT -- 使用 ML 框架内置的接口来使用 TensorRT

### TensorFlow1-TFTRT

+ 使用 TensorFlow1 的 TFTRT 来运行一个训练好的 TF 模型

### TensorFlow2-TFTRT

+ 使用 TensorFlow2 的 TFTRT 来运行一个训练好的 TF 模型

### Torch-TensorRT

+ 使用 Torch-TensorRT 来运行一个训练好的 pyTorch 模型

---

## 08-Tool -- 开发辅助工具

+ 一些开发辅助工及其使用范例的介绍

### Netron

+ ONNX 模型可视化工具，可用于查看模型的计算图元信息、网络结构、节点信息、张量信息

+ 下载地址：[Link](https://github.com/lutzroeder/Netron)

+ 子目录 Netron 中有一个 model.onnx 可供使用，可用 Netron 打开

+ 有一个比较有意思的工具 onnx-modifier [Link](https://github.com/ZhangGe6/onnx-modifier)。外观类似 Netron 并且可以对 ONNX 计算图做简单编辑，不过编辑大型模型时比较卡顿，能做的修改类型也比较有限，编辑大型模型还是推荐使用 onnx-graphsurgeon

### Nsight systems

+ 程序性能调试器（替代旧性能分析工具 nvprof 和 nvvp）

+ 随 CUDA 安装 [Link](https://developer.nvidia.com/cuda-zone) 或独立下载安装 [Link](https://developer.nvidia.com/nsight-systems)，位于 /usr/local/cuda/bin/ 下的 nsys 和 nsys-ui

+ 参考文档 [Link](https://docs.nvidia.com/nsight-systems/UserGuide/index.html)

+ 注意将 nsight systems 更新到最新版本，较老的 nsys-ui 可能打不开较新 nsight systems 生成的 .qdrep 或 .nsys-rep

+ 使用方法：命令行运行 `nsys profile XXX`，获得 .qdrep 或 .qdrep-nsys 文件，然后打开 nsys-ui，将上述文件拖入即可观察 timeline

+ TensorRT 运行分析建议
  + 只计量运行阶段，而不分析构建期
  + 构建期打开 ProfilingVerbosity 以便获得关于 Layer 的更多信息（见 09-Advance/ProfilingVerbosity）
  + 可以搭配 trtexec 使用，例如 `nsys profile -o myProfile -f true trtexec --loadEngine=model.plan --warmUp=0 --duration=0 --iterations=50`
  + 也可以配合自己的 script 使用，例如：`nsys profile -o myProfile -f true python3 myScript.py`

### OnnxGraphsurgeon

+ ONNX 计算图编辑库

+ 下载方法 `pip install nvidia-pyindex onnx-graphsurgeon`

+ 参考文档 [Link](https://docs.nvidia.com/deeplearning/tensorrt/onnx-graphsurgeon/docs/index.html)

+ 这部分代码参考了 NVIDIA 官方仓库关于 onnx-graphsurgeon 的范例代码 [Link](https://github.com/NVIDIA/TensorRT/tree/master/tools/onnx-graphsurgeon/examples)

+ 功能和 API 上有别于 onnx 库

+ 功能：
  + 修改计算图：计算图元信息 / 节点信息 / 张量信息 / 权重信息
  + 修改子图：添加 / 删除 / 替换 / 隔离
  + 优化计算图：常量折叠 / 拓扑排序 / 去除无用层

### Onnxruntime

+ 运行 ONNX 模型进行推理的 runtime 系统，在这里常用于检查 ML 框架导出模型的正确性

+ 下载方法 `pip install runtime-gpu -i https://pypi.ngc.nvidia.com`

+ 参考文档 [Link](https://onnxruntime.ai/)

### Polygraphy

+ 深度学习模型调试器

+ 下载方法 `pip install polygraphy`

+ 包含 python API 和命令行工具

+ 参考文档 [Link](https://docs.nvidia.com/deeplearning/tensorrt/polygraphy/docs/index.html) 和一个视频 [Link](https://www.nvidia.com/en-us/on-demand/session/gtcspring21-s31695/)

+ 功能：
  + 使用多种后端运行推理计算，包括 TensorRT，onnxruntime，TensorFlow
  + 比较不同后端的逐层计算结果
  + 由模型文件生成 TensorRT 引擎并序列化为 .plan
  + 查看模型网络的逐层信息
  + 修改 Onnx 模型，如提取子图，计算图化简
  + 分析 Onnx 转 TensorRT 失败原因，将原计算图中可以 / 不可以转 TensorRT 的子图分割保存

### trex

+ TensorRT engine 可视化和分析工具

+ 原仓库 [Link](https://github.com/NVIDIA/TensorRT/tree/main/tools/experimental/trt-engine-explorer)

+ 功能
  + 利用 TensorRT 构建起和运行期产生的 json 数据文件分析最终以后画好的 TensorRT Engine 的信息

### trtexec

+ TensorRT 命令行工具和推理性能测试工具

+ 随 TensorRT 安装，位于 tensorrt-XX/bin/trtexec

+ 功能
  + 由 .onnx 模型文件生成 TensorRT 引擎并序列化为 .plan
  + 查看 .onnx 或 .plan 文件的网络逐层信息
  + 模型性能测试（测试TensorRT 引擎基于随机输入或给定输入下的性能）

---

## 09-Advance -- TensorRT 高级用法

### AlgorithmSelector

+ 手工筛选 TensorRT 构建期自动优化阶段网络每一层可用的 tactic
  
### CreateExecutionContextWithoutDeviceMemory

+ Execution Context 对象的 CreateExecutionContextWithoutDeviceMemory 方法的使用范例（有点问题）

### CudaGraph

+ 结合使用 CUDA Graph 和 TensorRT

+ 有助于解决模型运行的 launch bound 问题

### EmptyTensor[TODO]

+ TensorRT 中 空张量的使用范例

### EngineInspector

+ 打印 TensorRT 构建起自动优化后的网络结构

### ErrorRecoder

+ 自定义 TensorRT 的报错日志

### GPUAllocator

+ 在 TensorRT 的构建期和运行期分别使用自定义的的显存管理器来管理 TensorRT 的显存使用

### LabeledDimension

+ 命名维度功能，指定每一个动态维度的名称，方便 TensorRT 检查输入输出张量形状和进行相关优化

+ since tensorRT 8.4

### Logger

+ 自定义 TensorRT 运行日志记录器

### MultiContext

+ 对 TensorRT 的一个推理引擎，使用多个 TensorRT 上下文进行并行推理

### MultiOptimizationProfile

+ 对 TensorRT 的一个推理引擎，使用多个动态数据范围进行推理

+ 可以优化动态范围较大的模型的推理性能

### MultiStream

+ 使用多个 CUDA stream 进行推理

+ 这个范例只说明了 python 中 CUDA stream 的使用，其中页锁定内存的使用（尤其是向页锁定内存中拷入考出数据）需参考 09-Advance/StreamAndAsync 的办法

+ 可以优化模型 Data Copy bound 的问题

### nvtx

+ 使用 NVIDIA Tools Extension (NVTX) 标记程序运行 timeline，以便 Nsight Systems 分析

### Profiler

+ 自定义 TensorRT 构建期和运行期各层网络的运行时间

+ 为后续手工优化提供数据支持

### ProfilingVerbosity

+ TensorRT 详细日志开关

+ 可用于打印运行时间数据或被 Nsight Sysytems 使用

### Refit

+ 可用于更新 TensorRT 推理引擎的权重

+ 强化学习必备工具

+ Refit-set_weights.py，使用 set_weights API 来进行改装操作

+ Refit-set_named_weights.py，使用 set_named_weights API 来进行改装操作（与 set_weights API稍有不同，单功能基本等价）

+ Refit-OnnxByParser.py，使用来自 ONNX 文件的新权重，并使用 TensorRT Parser 来进行改装操作

+ Refit-OnnxByWeight.py，使用来自 ONNX 文件的新权重，采用保存 ONNX 文件的权重并在 TensorRT 中重新喂给该层的方法来进行改装操作

### Safety

+ 使用 TensorRT 的 Safety 模块构建推理引擎并进行推理

+ Safety 模式仅适用于 Drive Platform（QNX）[Link](https://github.com/NVIDIA/TensorRT/issues/2156)

### Sparsity

+ 使用结构化稀疏特性加速计算

### StreamAndAsync

+ 使用 CUDA Stream 和异步 API 实现 TensorRT 异步推理

### StrictType

+ 手工指定网络每一层的计算精度（float32/float16/in8/...）

### TacticSource

+ 手工指定 TensorRT 构建期自动优化阶段的备选 kernel 范围

### TimingCache

+ 保存和复用 TensorRT 构建期的自动优化测试结果缓存，用于多次构建具有完全相同 tactic 的引擎

---

## 10-BestPractice -- 有趣的 TensorRT 优化范例

+ 在 TensorRT 模型优化工作中总结出来的、计算图手工优化的部分范例

+ 部分参考资料 [Link](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#optimize-layer)

### AdjustReduceLayer

+ 优化 Reduce 层，通常针对输入张量的末尾维度进行规约操作效率更高

+ 范例代码一共测试了三种情形
    1. 直接 Reduce：[B,256,T] -> [B,1,T]
    2. 添加一对 Transpose：[B,256,T] -> [B,T,256] -> [B,T,1] -> [B,1,T]
    3. 在 2. 中第一个 Transpose 后面再加一个 Identity：[B,256,T] -> [B,T,256] -> [B,T,256] -> [B,T,1] -> [B,1,T]
+ 2. 中的一对 Transpose 会被 TensorRT 融合掉（变成跟 1. 一样的网络），3. 中再添加一个 Identity 破坏了这种融合，强制 Reduce 对末尾维度进行规约计算

### AlignSize

+ 优化矩阵乘法相关的层，通常数据对齐到一定的倍数上会有较好的性能

+ 范例代码一共测试了四种情形
    1. [32,1] -> [32,256] -> [32,2048] -> [32,256] -> [32,2048] -> ... -> [32,256]
    2. [31,1] -> [31,256] -> [31,2048] -> [31,256] -> [31,2048] -> ... -> [31,256]
    3. [32,1] -> [32,255] -> [32,2048] -> [32,255] -> [32,2048] -> ... -> [32,255]
    4. [32,1] -> [32,256] -> [32,2047] -> [32,256] -> [32,2047] -> ... -> [32,256]
+ 使用 script 时情形 1. 和 2. 性能接近，高于情形 3. 和 4. 的性能

### ComputationInAdvance

+ 手动提前计算以减少运行期计算量

+ 范例代码一共测试了两种情形
    1. 使用源代码算法，运行期输入张量经切片后参与 12 个矩阵乘法，然后转置
    2. 在计算图中提前完成矩阵乘法和转置，运行期输入张量用于将结果进行切片
+ 优化后性能为原来 4 倍
+ TensorRT 中使用了 shape input（因为输出张量形状依赖于输入张量的值），不同于其他范例的 dynamic shape
+ 
### Convert3DMMTo2DMM

+ 优化矩阵乘法相关的层，二维矩阵参与矩阵乘能比三维矩阵参与矩阵乘获得更好的性能

+ 范例代码一共测试了两种情形
    1. [32,256,1] -> [32,256,256] -> [32,256,2048] -> [32,256,256] -> [32,256,2048] -> ... -> [32,256,256] -> [32,256,1]
    2. 在 1. 的最前和最后添加两个 Reshape 节点，在最前将输入张量的前两维合并，在最后还原输出张量的前两维。
        [32,256,1] -> [32*256,256] -> [32*256,2048] -> [32*256,256] -> [32*256,2048] -> ... -> [32*256,256] -> [32,256,1]
+ 使用二维矩阵乘法的性能优于三维矩阵乘法

### ConvertTranposeMultiplicationToConvolution

+ 将 Transpose + Matrix Multiplication 转化为 Convolution + Shuffle

+ 范例代码测试了在 GTX 1070 和 A30 上的效果
  + 在 GTX1070 上，小 BatchSize 上就较好的加速效果
  + 在 A30 上，生成的网络结构与 GTX1070 不同，在小 BatchSize 上没有显著加速，但在加大 BatchSize 后体现出了明显加速效果
+ 使用二维矩阵乘法的性能优于三维矩阵乘法
+ 感谢 TensorRT Hackathon 2022 的 “宏伟” 同学提供思路

### EliminateSqueezeUnsqueezeTranspose

+ 手动删除一些 Squeeze/Unsqueeze/Transpose 层，以便 TensorRT 做进一步层融合

+ 范例代码一共测试了两种情形
    1. Conv -> Conv -> Unsqueeze -> Add -> Squeeze -> ReLU -> ... -> Conv -> Transpose -> Add -> Transpose -> ReLU -> ... -> Conv
    2. 去掉了 1. 中所有 Squeeze/UnsqueezeTranspose，使得所有 Conv+Add+ReLU 可以被 TensorRT 融合成一个 kernel
        Conv -> ConvAddReLU -> ... -> ConvAddReLU -> Conv
+ 优化后性能几乎翻倍



### IncreaseBatchSize

+ 增大推理计算的 Batch Size 来提升总体吞吐量

+ 范例代码对同一模型尝试 BatchSize = 1 ~ 1024 尺寸进行推理计算
+ 随着 Batch Size 增大，计算延迟先基本不变后逐渐增大，而吞吐量持续上升

### UsingMultiOptimizationProfile

+ 范例代码对同一模型尝试两种动态范围策略，第一种（model-1.plan）采用一整个 Optimization Profile，第二种（model-2.plan）采用两个分割的 Optimization Profile，分别用于大形状和小形状，然后分别测试不同输入数据形状下的性能表现

+ 采用多个 Optimization Profile 情况下整体性能表现会更好一些

### UsingMultiStream

+ 同 09-Advance/MultiStream，使用多个 CUDA Stream 来重叠数据拷贝和数据计算的时间

---

## 11-ProblemSolving

+ 在将模型部署到 TensorRT 上时常见的报错信息及其解决办法

+ 这部分将会随着 TensorRT 的版本更新不断扩充和调整

### ParameterCheckFailed

### Slice Node With Bool IO

### Weights Are Not Permitted Since They Are Of Type Int32

### Weights Has Count X But Y Was Expected

---

## 50-Resource -- 文档资源

+ TensorRT 教程的幻灯片对应的 PDF 文件，以及一些其他有用的参考资料

+ 由于 Github 的 markdown 不能原生支持 Latex，cookbook 的文档中使用到 Latex 的 \*.md 会导出一个 PDF 文件放在本目录下，方便浏览

+ 目前收录的内容
  + 02-API/Layer/\*Layer/\*.md，导出自 cookbook/02-API/Layer/\*Layer/\*.md 的 PDF 版本，即各种层的带公式版本的说明文件
  + number.pdf，导出自 cookbook/51-Uncategorized/number.md，各浮点和整数类型的数据范围，目前包括 FP64、FP32、TF32、FP16、BF16、FP8e5m2、FP8e4m3，INT64、INT32、INT16、INT8、INT4
  + Hackathon2022-初赛总结-Wenet优化-V1.1.pdf，导出自 51-Uncategorized/Hackathon2022-初赛总结-Wenet优化-V1.1.pptx，2022 年 TensorRT Hackathon 初赛试题 Wenet 模型优化精讲
  + TensorRT教程-TRT8.2.3-V1.1.pdf，导出自 51-Uncategorized/TensorRT教程-TRT8.2.3-V1.1.pptx，TensorRT 8.2.3 视频教程幻灯片

---

## 51-Uncategorized

+ 未分类的一些东西

+ 目前收录的内容
  + getTensorRTVersion.sh，用于查看运行环境 CUDA、cuDNN、TensorRT 版本的脚本
  + Hackathon2022-初赛总结-Wenet优化-V1.1.pptx，2022 年 TensorRT Hackathon 初赛试题 Wenet 模型优化精讲
  + number.md，各浮点和整数类型的数据范围，目前包括 FP64、FP32、TF32、FP16、BF16、FP8e5m2、FP8e4m3，INT64、INT32、INT16、INT8、INT4
  + TensorRT教程-TRT8.2.3-V1.1.pptx，TensorRT 8.2.3 视频教程幻灯片

---

## 52-Deprecated

+ 旧版本 TensorRT 中的一些 API 和用法，他们在较新版本 TensorRT 中已被废弃，直接运行会报错退出

---

## 99-NotFinish

+ 没有完成的范例代码，以及同学们提出的新的范例代码需求
