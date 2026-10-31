# 边缘设备上的 MNIST 模型对比（Raspberry Pi / AI Hardware 课程项目）

同一个任务（手写数字识别）上，把四个卷积网络跑一遍，记录**准确率、推理延迟、
CPU/内存占用**，用来回答课程里那个问题：在边缘设备上，模型容量和延迟该怎么权衡。

代码分两套框架，因为组里各人环境不同：

| 脚本 | 框架 | 模型 | 输入 |
| --- | --- | --- | --- |
| `ResNet_train.py` / `ResNet_test.py` / `rnet_3.py` | TensorFlow / Keras | ResNet-50（ImageNet 权重迁移） | 28×28×3 |
| `EfficientNet-B0_test.py` | PyTorch | EfficientNet-B0 | 32×32×3 |
| `mobilenet_v2_test.py` | PyTorch | MobileNetV2 | 32×32×3 |
| `mnet_test.py` | PyTorch（较早版本） | MobileNetV2 | 32×32×3 |
| `dataloader.py` | TensorFlow | —— | 把 MNIST 展开成 `mnist_images/<split>/<label>/<i>.png`，供非 TF 脚本读取 |
| `predict.py` | TensorFlow | 读 `best_model.h5` | 单张 32×32 图片 |

测试脚本里的计时段落用 `time.perf_counter()` 包住前向推理，并用 `psutil`
采样 CPU 与内存；混淆矩阵和分类报告由 `scikit-learn`、`seaborn` 输出。

## 怎么跑

```console
$ pip install -r requirements.txt

# 1. 先把 MNIST 落成图片目录（TF 脚本会自己下载数据集）
$ python dataloader.py

# 2. 训练 + 测试：任选一个
$ python ResNet_train.py
$ python mobilenet_v2_test.py
$ python EfficientNet-B0_test.py
```

`requirements.txt` 里两套框架都给上了；实际只需要装你跑的那一套（TF 或 PyTorch）。

## 关于本 README 的旧版本

旧 README 描述的是「FPGA/ASIC 加速平台、Verilog 数据流并行、相对 CPU 延迟降低
60%、功耗下降 35%」，还带了三个 `https://drive.google.com/your_model_link`
占位链接。**这些内容与仓库里的代码不符**，本仓库里没有任何 RTL、没有功耗测量、
也没有可下载的模型文件。现在这份 README 只描述仓库里真实存在的脚本；课程报告里
的硬件设计与性能数字不在这里，也不在本仓库的复现范围内。

## 限制

- MNIST 上跑 ResNet-50 / EfficientNet-B0，参数量远超任务需要，延迟对比受输入
  尺寸影响很大（28×28 与 32×32 之间的重采样本身就带误差）。
- 没有固定随机种子，两次运行的准确率会有小波动；脚本里也没有保存训练好的权重
  （`predict.py` 期望的 `best_model.h5` 需要自己先训出来）。
- 计时是 wall-clock，包含数据搬运，不等于纯算子耗时。

## 许可

MIT，见 [LICENSE](LICENSE)。
