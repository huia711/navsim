# 理解与创建 Agent

定义一个新的 agent，需要从 `navsim.agents.abstract_agent.AbstractAgent` 继承并实现相应接口。

让我们更深入地看看这个基类。你需要实现以下方法：

- `__init__()`：

    agent 的构造函数。

- `name()`：

    返回该 agent 的名称。  
    该名称将被用于生成评估结果 csv 的文件名。  
    你可以将其设置为任意字符串。

- `initialize()`：

    在第一次对 agent 做推理之前会调用该方法。  
    如果使用多进程 / 多 worker，每个 worker 都会为自己持有的 agent 实例调用一次该方法。  
    如果你需要加载 state dict 等模型参数，应当在这里完成，而不是在 `__init__` 中进行。

- `get_sensor_config()`：

    必须返回一个 `SensorConfig`（参见 `navsim.common.dataclasses.SensorConfig`），用于定义在每一帧中，agent 所需加载的传感器模态。  
    `SensorConfig` 是一个 dataclass：对每种传感器，你可以指定一个历史帧索引列表，表示需要加载哪些时间步的数据；或者也可以直接用布尔值表示是否加载该传感器的所有可用帧。  
    如果你希望访问所有可用的传感器，可以返回 `SensorConfig.build_all_sensors()`。  
    关于可用传感器的详细信息见下文。

    **加载传感器会对运行时开销产生很大影响。如果某个传感器并不需要，建议将其设置为 `False` 以节省计算。**

- `compute_trajectory()`：

    这是 agent 的核心函数。给定包含自车状态和各类传感器模态的 `AgentInput`，该函数需要计算并返回 agent 的未来轨迹。  
    关于输出格式的详情见下文。

    **未来轨迹必须返回为 `navsim.common.dataclasses.Trajectory` 类型的对象。示例可参考 Constant Velocity agent 或 Human agent。**

# 基于学习的 Agent

大多数情况下，你的 agent 会包含学习型组件。  
Navsim 提供了一个轻量、易用的训练接口。要使用它，你的 agent 需要额外实现一些功能。  
除上面提到的方法外，还需要实现以下方法。  
可以参考 `navsim.agents.ego_status_mlp_agent.EgoStatusMLPAgent` 的实现示例。

- `get_feature_builders()`  
  该方法需要返回一个特征构建器（Feature Builder）列表，每个元素的类型为 `navsim.planning.training.abstract_feature_target_builder.AbstractFeatureBuilder`。  
  FeatureBuilder 会接收 `AgentInput` 对象并计算训练 / 推理所需的特征张量。一个 FeatureBuilder 可以输出多个特征张量，并以字典形式返回；之后这些特征字典会在模型的 `forward` 中被使用。  
  目前我们提供以下特征构建器：
    - [EgoStatusFeatureBuilder](https://github.com/autonomousvision/navsim/blob/main/navsim/agents/ego_status_mlp_agent.py#L18)：返回包含当前速度、加速度和驾驶指令的张量。
    - [TransfuserFeatureBuilder](https://github.com/autonomousvision/navsim/blob/main/navsim/agents/transfuser/transfuser_features.py#L28)：返回一个字典，包含当前前向相机图像、LiDAR BEV 图，以及自车状态。

- `get_target_builders()`  
  与 `get_feature_builders()` 类似，但返回的是用于训练的 target builder，类型为 `navsim.planning.training.abstract_feature_target_builder.AbstractTargetBuilder`。  
  不同于特征构建器，target builder 可以访问包含真值信息的 `Scene` 对象（而不仅是 `AgentInput`）。

- `forward()`  
  模型的前向传播。该函数接收一个特征字典，包含由所有特征构建器生成的特征张量。  
  所有张量已经按 batch 维度拼接，并且位于与模型相同的设备上。  
  前向传播需要返回一个字典，其中必须包含键 `"trajectory"`，其对应的张量描述未来轨迹，形状为 \[B, T, 3]，其中 B 为 batch size，T 为未来时间步数，3 依次表示 \(x, y, heading\)。

- `compute_loss()`  
  给定特征、目标以及模型预测结果，该函数需要计算用于训练的损失，并以单个 Tensor 的形式返回。

- `get_optimizers()`  
  用于定义训练时使用的优化器。  
  如果不使用学习率调度器，该方法直接返回一个 `torch.optim.Optimizer`。  
  若需要学习率调度器，则应返回一个字典，其中 `"optimizer"` 对应 `torch.optim.Optimizer`，`"lr_scheduler"` 对应 `torch.optim.lr_scheduler.LRScheduler`。

- `get_training_callbacks()`  
  该函数返回一个 `pl.Callback` 列表，用于在训练过程中监控或可视化模型表现。  
  我们在 `navsim.agents.transfuser.transfuser_callback.TransfuserCallback` 中实现了一个用于 Transfuser 的回调，可作为起点进行扩展。

- `compute_trajectory()`（学习型 Agent）  
  与非学习型 agent 不同，**对于学习型 Agent，你不需要手动实现该函数**。  
  在推理阶段，轨迹会通过特征构建器与 `forward` 方法自动计算得到。

## 输入

你可以重写 `get_sensor_config()` 来决定 agent 可以访问哪些传感器。

可用的传感器依赖于具体数据集。对于 OpenScene，目前包含 9 种传感器模态：8 个摄像头和一个融合点云（由 5 个 LiDAR 合成）。  
每种模态都提供过去 2 秒、2Hz 频率（共 4 帧）的历史数据。  
在测试集上，我们只会公开这些数据（不包括地图 / 轨迹 / 占据栅格等信息）。你可以在训练时使用这些额外信息，但在提交到排行榜时将无法访问。

你可以通过 `navsim.common.dataclasses.SensorConfig` dataclass 配置要使用的传感器模态以及每一帧所需的历史长度。

**为什么要用 LiDAR？**  
近期的开放式规划文献里，许多方法逐渐弱化了 LiDAR 的作用，而更多地依赖多视角高分辨率摄像头。这显著提高了训练和测试 SOTA 规划器的算力开销。  
我们希望通过提供 LiDAR 模态，鼓励提交更加高效的方案，例如减少摄像头数量或使用低分辨率输入。

**自车状态（Ego Status）。**  
除了传感器数据之外，agent 还会接收到自车在局部坐标系下的位姿、速度和加速度信息。  
为了解歧驾驶意图，我们还提供一个离散驾驶指令，指示期望路线是向左、直行还是向右；另外还有一个 “unknown” 指令，可用于在训练中过滤样本。  
需要注意的是，NAVSIM 中的驾驶指令**仅基于目标路线**，不会与障碍物或交通标志信息纠缠（这在早期如 nuScenes 等基准中较为常见）。  
左转和右转指令不仅覆盖转弯，也包括变道和急弯等情况。

## 输出

基于上述输入，你需要在 `compute_trajectory()` 中输出一个 `Trajectory`。  
这是一个 BEV 轨迹序列（以局部坐标系下的 \(x, y, heading\) 表示），并附带一个 `TrajectorySampling` 配置对象，指明轨迹的时间长度和采样频率。  
PDM 分数在 4 秒时间范围、10Hz 频率上进行评估。  
当输出的轨迹频率与评估频率不同时，`TrajectorySampling` 配置会被用于插值。

你可以参考基线实现来了解如何编写 agent！

## 基线 Agent

NAVSIM 提供了多个基线模型，可作为对比或新端到端驾驶模型的起点。  
我们在 [Hugging Face](https://huggingface.co/autonomousvision/navsim_baselines) 上提供了所有学习型基线的模型权重。

### `ConstantVelocityAgent`

`ConstantVelocityAgent` 是一个非常朴素的基线，只执行最简单的驾驶逻辑：  
agent 保持恒定速度和恒定航向角，输出一条直线轨迹。  
你可以使用该 agent 熟悉 `AbstractAgent` 接口，或分析那些简单即可获得较高 PDM 分数的样本。

实现见：[代码链接](https://github.com/autonomousvision/navsim/blob/main/navsim/agents/constant_velocity_agent.py)。

### `EgoStatusMLPAgent`

`EgoStatusMLPAgent` 是一个 “盲” 的基线，它完全忽略环境感知相关的传感器，只使用自车状态（速度、加速度与驾驶指令），并通过多层感知机（MLP）生成轨迹。  
因此，EgoStatusMLP 提供了一个仅依赖自车运动学状态即可达到的性能上界。  
它是一个轻量的学习示例，展示了如何创建特征缓存并在 NAVSIM 中训练一个 agent。

实现见：[代码链接](https://github.com/autonomousvision/navsim/blob/main/navsim/agents/ego_status_mlp_agent.py)。

### `TransfuserAgent`

[Transfuser](https://arxiv.org/abs/2205.15997) 是一个示例传感器 agent，同时使用相机与 LiDAR 输入。  
其骨干网络在前向摄像头图像和离散化 LiDAR BEV 网格上分别使用 CNN 提取特征，然后通过 Transformer 在多个卷积阶段融合两路特征。  
Transfuser 结合了多种辅助任务和模仿学习，并在 CARLA 仿真器的端到端驾驶任务中取得了强劲的闭环表现。

在 NAVSIM 中，我们复现了来自 [CARLA Garage](https://github.com/autonomousvision/carla_garage) 的 Transfuser 骨干，并将 BEV 语义分割和基于 DETR 的目标检测作为辅助任务。  
为适配 Transfuser 所需的广角视场，我们将三个前向摄像头的图像拼接在一起。  
Transfuser 是构建传感器 agent 的良好起点，提供了图像与 LiDAR 传感器的预处理、训练可视化回调以及更高级的损失函数（如用于检测任务的匈牙利匹配）。

实现见：[代码链接](https://github.com/autonomousvision/navsim/blob/main/navsim/agents/transfuser)。

### `LatentTransfuserAgent`

[Latent Transfuser (LTF)](https://arxiv.org/abs/2205.15997) 是 TransFuser 架构的一个变体，通过显式位置编码来替代 LiDAR 数据。  
这使得模型即便在推理阶段没有 LiDAR 输入时也能正常工作，更具灵活性。  
在 CARLA 榜单上的实验表明，LTF 显著优于其他仅使用图像的方案，是一个强有力的图像专用自动驾驶基线。  
它将基于 Transformer 的注意力机制与无需 LiDAR 的灵活性很好地结合在一起。

在 NAVSIM 中，你只需在现有的 `TransfuserAgent` 配置中设置 `agent.config.latent=True` 即可启用该特性。

实现见：[代码链接](https://github.com/autonomousvision/navsim/blob/main/navsim/agents/transfuser)。


