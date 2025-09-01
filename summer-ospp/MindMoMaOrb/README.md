
# 模型名称

> Orb

## 介绍

> 材料科学中，设计新型功能材料一直是新兴技术的关键部分。然而，传统的从头算计算方法在设计新型无机材料时速度慢且难以扩展到实际规模的系统。近年来，深度学习方法在多个领域展示了其强大的能力，能够通过并行架构高效运行。ORB模型的核心创新在于将这种深度学习方法应用于材料建模，通过可扩展的图神经网络架构学习原子间相互作用的复杂性。ORB模型是一个基于图神经网络（GNN）的机器学习力场（MLFF），设计为通用的原子间势能模型，适用于多种模拟任务（几何优化、蒙特卡洛模拟和分子动力学模拟）。该模型的输入是一个图结构，包含原子的位置、类型以及系统配置（如晶胞尺寸和边界条件）；输出包括系统的总能量、每个原子的力向量以及单元格应力。与现有的开源神经网络势能模型（如MACE）相比，ORB模型在大系统规模下的速度提高了3-6倍。在Matbench Discovery基准测试中，ORB模型的误差比其他方法降低了31%，并且在发布时成为该基准测试的最新最佳模型。ORB模型在零样本评估中表现出色，即使在没有针对特定任务进行微调的情况下，也能在高温度非周期分子的分子动力学模拟中保持稳定。

## 环境要求

> 1. 安装`mindspore（2.5.0）`
> 2. 安装依赖包：`pip install -r requirement.txt`

## 快速入门

> 1. 在[数据集链接](https://download-mindspore.osinfra.cn/mindscience/mindchemistry/orb/dataset/)下载相应的数据集并放在`dataset`目录下
> 2. 在[模型链接](https://download-mindspore.osinfra.cn/mindscience/mindchemistry/orb/orb_ckpts/)下载orb预训练模型ckpt并放在`orb_ckpts`目录下
> 3. 安装依赖包：`pip install -r requirement.txt`
> 4. 单卡训练命令： `bash run.sh`
> 5. 多卡训练命令： `bash run_parallel.sh`
> 6. 评估命令： `python evaluate.py`
> 7. 模型预测结果会存在`results`目录下

### 代码目录结构

```text
代码主要模块在src文件夹下，其中dataset文件夹下是数据集，orb_ckpts文件夹下是预训练模型和训练好的模型权重文件，configs文件夹下是各代码的参数配置文件。

orb_models                                           # 模型名
├── dataset
  ├── train_mptrj_ase.db                             # 微调阶段训练数据集
  └── val_mptrj_ase.db                               # 微调阶段测试数据集
├── orb_ckpts
  └── orb-mptraj-only-v2.ckpt               # 预训练模型checkpoint
├── configs
  ├── config.yaml                                    # 单卡训练参数配置文件
  ├── config_parallel.yaml                           # 多卡并行训练参数配置文件
  └── config_eval.yaml                               # 推理参数配置文件
├── src
  ├── __init__.py
  ├── atomic_system.py                               # 定义原子系统的数据结构
  ├── base.py                                        # 基础类定义
  ├── ase_dataset.py                                 # 处理和加载数据集
  ├── calculator.py                                  # 用于计算原子系统的能量、力或其他物理性质
  ├── featurization_utilities.py                     # 提供将原子系统转换为特征向量的工具
  ├── gns.py                                         # 图神经网络相关
  ├── graph_regressor.py                             # 图回归模型
  ├── nn_util.py                                     # 神经网络工具
  ├── pretrained.py                                  # 预训练模型相关函数
  ├── property_definitions.py                        # 定义原子系统中各种物理性质的计算方式和命名规则
  ├── rbf.py                                         # 实现径向基函数的计算，可能用于将原子间距离转换为特征向量
  ├── reference_energies.py                          # 用于模型能量计算的基准
  ├── segment_ops.py                                 # 提供对数据进行分段处理的工具
  └── utils.py                                       # 工具模块
├── finetune.py                                      # 模型单卡微调
├── finetune_prallel.py                              # 模型并行微调
├── evaluate.py                                      # 模型推理
├── run.sh                                           # 单卡训练启动脚本
├── run_parallel.sh                                  # 多卡并行训练启动脚本
└── requirement.txt                                  # 环境
```  

## 下载数据集

在[数据集链接](https://download-mindspore.osinfra.cn/mindscience/mindchemistry/orb/dataset/)下载训练和测试数据集放置于当前路径的dataset文件夹下（如果没有需要自己手动创建）；在[模型链接](https://download-mindspore.osinfra.cn/mindscience/mindchemistry/orb/orb_ckpts/)下载orb预训练模型`orb-mptraj-only-v2.ckpt`放置于当前路径的orb_ckpts文件夹下（如果没有需要自己手动创建）；文件路径参考[代码目录结构](#代码目录结构)

## 训练过程

### 单卡训练

更改`configs/config.yaml`文件中训练参数:

> 1. 设置微调阶段的训练和测试数据集，见`data_path`字段
> 2. 设置训练加载的预训练模型权重文件，更改`checkpoint_path`路径字段
> 3. 其它训练设置见Training Configuration部分

```bash
pip install -r requirement.txt
bash run.sh
```

代码运行结果如下所示：

```log
==============================================================================================================
Please run the script as:
bash run.sh
==============================================================================================================
2025-06-02 01:09:34,543 - INFO - Loading datasets: dataset/train_mptrj_ase.dbTotal train dataset size: 800 samples
2025-06-02 01:10:02,390 - INFO - Loading datasets: dataset/val_mptrj_ase.dbTotal train dataset size: 200 samples
2025-06-02 01:10:07,079 - INFO - Model has 25213610 trainable parameters.
Epoch: 0/100,
 train_metrics: {'data_time': 0.00010895108183224995, 'train_time': 386.58018293464556, 'energy_reference_mae': 5.598883946736653, 'energy_mae': 3.3611322244008384, 'energy_mae_raw': 103.14391835530598, 'stress_mae': 41.36046473185221, 'stress_mae_raw': 12.710869789123535, 'node_mae': 0.02808943825463454, 'node_mae_raw': 0.0228044210622708, 'node_cosine_sim': 0.7026202281316122, 'fwt_0.03': 0.23958333333333334, 'loss': 44.74968592325846}
 val_metrics: {'energy_reference_mae': 5.316623687744141, 'energy_mae': 3.594848871231079, 'energy_mae_raw': 101.00129699707031, 'stress_mae': 30.630516052246094, 'stress_mae_raw': 9.707925796508789, 'node_mae': 0.017718862742185593, 'node_mae_raw': 0.014386476017534733, 'node_cosine_sim': 0.5506304502487183, 'fwt_0.03': 0.375, 'loss': 34.24308395385742}

...

Epoch: 99/100,
 train_metrics: {'data_time': 7.802306208759546e-05, 'train_time': 59.67856075416785, 'energy_reference_mae': 5.5912095705668134, 'energy_mae': 0.007512244085470836, 'energy_mae_raw': 0.21813046435515085, 'stress_mae': 0.7020445863405863, 'stress_mae_raw': 2.222463607788086, 'node_mae': 0.04725319395462672, 'node_mae_raw': 0.042800972859064736, 'node_cosine_sim': 0.3720853428045909, 'fwt_0.03': 0.09895833333333333, 'loss': 0.7568100094795227}
 val_metrics: {'energy_reference_mae': 5.308632850646973, 'energy_mae': 0.27756747603416443, 'energy_mae_raw': 3.251189708709717, 'stress_mae': 2.8720269203186035, 'stress_mae_raw': 9.094478607177734, 'node_mae': 0.05565642938017845, 'node_mae_raw': 0.05041291564702988, 'node_cosine_sim': 0.212838813662529, 'fwt_0.03': 0.19499999284744263, 'loss': 3.2052507400512695}
2025-06-02 03:12:22,942 - INFO - Checkpoint saved to orb_ckpts/
2025-06-02 03:12:22,942 - INFO - Training time: 7333.08717 seconds
```

### 多卡并行训练

更改`configs/config_parallel.yaml`和`run_parallel.sh`文件中训练参数:

> 1. 设置微调阶段的训练和测试数据集，见`data_path`字段
> 2. 设置训练加载的预训练模型权重文件，更改`checkpoint_path`路径字段
> 3. 其它训练设置见Training Configuration部分
> 4. 修改`run_parallel.sh`文件中`--worker_num=4 --local_worker_num=4`来设置调用的卡的数量

```bash
pip install -r requirement.txt
bash run_parallel.sh
```

代码运行结果如下所示：

```log
2025-05-22 00:30:45,548 - INFO - Loading datasets: dataset/train_mptrj_ase.dbTotal train dataset size: 800 samples
2025-05-22 00:30:45,728 - INFO - Loading datasets: dataset/train_mptrj_ase.dbTotal train dataset size: 800 samples
2025-05-22 00:30:45,686 - INFO - Loading datasets: dataset/train_mptrj_ase.dbTotal train dataset size: 800 samples
2025-05-22 00:30:45,681 - INFO - Loading datasets: dataset/train_mptrj_ase.dbTotal train dataset size: 800 samples
2025-05-22 00:31:08,282 - INFO - Loading datasets: dataset/val_mptrj_ase.dbTotal train dataset size: 200 samples
2025-05-22 00:31:08,584 - INFO - Loading datasets: dataset/val_mptrj_ase.dbTotal train dataset size: 200 samples
2025-05-22 00:31:08,495 - INFO - Loading datasets: dataset/val_mptrj_ase.dbTotal train dataset size: 200 samples
2025-05-22 00:31:08,096 - INFO - Loading datasets: dataset/val_mptrj_ase.dbTotal train dataset size: 200 samples
2025-05-22 00:31:12,594 - INFO - Model has 25213607 trainable parameters.
2025-05-22 00:31:13,056 - INFO - Model has 25213607 trainable parameters.
2025-05-22 00:31:13,408 - INFO - Model has 25213607 trainable parameters.
2025-05-22 00:31:13,666 - INFO - Model has 25213607 trainable parameters.

...

2025-05-22 01:10:51,992 - INFO - Training time: 2375.89474 seconds
2025-05-22 01:10:52,005 - INFO - Training time: 2377.02413 seconds
2025-05-22 01:10:52,675 - INFO - Training time: 2377.22778 seconds
2025-05-22 01:10:52,476 - INFO - Training time: 2376.63176 seconds
[INFO] PS(2744253,ffff137ef120,python):2025-05-22-01:11:03.754.142 [mindspore/ccsrc/ps/core/communicator/tcp_client.cc:318] Start] Event base dispatch success!
[INFO] PS(2744253,ffff13fff120,python):2025-05-22-01:11:03.754.184 [mindspore/ccsrc/ps/core/communicator/tcp_server.cc:220] Start] Event base dispatch success!
[INFO] PS(2744259,ffff1ffff120,python):2025-05-22-01:11:03.529.843 [mindspore/ccsrc/ps/core/communicator/tcp_client.cc:318] Start] Event base dispatch success!
[INFO] PS(2744259,ffff3495a120,python):2025-05-22-01:11:03.529.844 [mindspore/ccsrc/ps/core/communicator/tcp_server.cc:220] Start] Event base dispatch success!
[INFO] PS(2744247,ffff19fbf120,python):2025-05-22-01:11:06.926.027 [mindspore/ccsrc/ps/core/communicator/tcp_client.cc:318] Start] Event base dispatch success!
[INFO] PS(2744247,ffff1a7cf120,python):2025-05-22-01:11:06.926.027 [mindspore/ccsrc/ps/core/communicator/tcp_server.cc:220] Start] Event base dispatch success!
[INFO] PS(2744241,ffff2cf0c120,python):2025-05-22-01:11:10.634.471 [mindspore/ccsrc/ps/core/communicator/tcp_client.cc:318] Start] Event base dispatch success!
[INFO] PS(2744241,ffff2d71c120,python):2025-05-22-01:11:10.634.471 [mindspore/ccsrc/ps/core/communicator/tcp_server.cc:220] Start] Event base dispatch success!
```

### 推理

更改`configs/config_eval.yaml`文件中推理参数:

> 1. 设置测试数据集，见`val_data_path`字段
> 2. 设置推理加载的预训练模型权重文件，更改`checkpoint_path`路径字段
> 3. 其它训练设置见Evaluating Configuration部分

```bash
python evaluate.py
```

代码运行结果如下所示：

```log
2025-05-22 00:18:51,054 - INFO - Loading datasets: dataset/val_mptrj_ase.dbTotal train dataset size: 200 samples
2025-05-22 00:19:02,033 - INFO - Model has 25213607 trainable parameters.
.Validation loss: 0.89507836
    energy_reference_mae: 5.3159098625183105
    energy_mae: 0.541229784488678
    energy_mae_raw: 4.244375228881836
    stress_mae: 0.22862032055854797
    stress_mae_raw: 10.575761795043945
    node_mae: 0.12522821128368378
    node_mae_raw: 0.04024107754230499
    node_cosine_sim: 0.38037967681884766
    fwt_0.03: 0.22499999403953552
    loss: 0.8950783610343933
```
