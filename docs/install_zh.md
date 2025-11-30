# 下载与安装指南

以下步骤可帮助你完成 NAVSIM 的下载与安装。

## 1. 克隆 navsim-devkit

克隆官方仓库并进入目录：

```bash
git clone https://github.com/autonomousvision/navsim.git
cd navsim
```

## 2. 下载数据集

你需要获取 OpenScene 的日志与传感器数据，以及 nuPlan 地图。本仓库提供了下载 nuPlan 地图、mini split 与 test split 的脚本。  

**注意：下载数据前务必阅读并同意 [LICENSE](https://motional-nuplan.s3-ap-northeast-1.amazonaws.com/LICENSE)。**

在 `download` 目录中运行：

```bash
cd download && ./download_maps
```

接着，根据需求下载不同的数据划分。  
这些划分与推荐的标准训练 / 测试划分并非完全对应，详情可参考 [splits.md](splits.md) 了解每个标准划分的大小以及需要下载哪些数据。

提供的下载脚本如下：

```bash
./download_mini
./download_trainval
./download_test
./download_warmup_two_stage
./download_navhard_two_stage
./download_private_test_hard_two_stage
```

此外，脚本 `./download_navtrain` 可用于下载 `trainval` 中的一小部分数据，以支持 `navtrain` 训练划分。

所有下载内容会先存放在 `download` 目录。完成后请整理为如下目录结构：

```angular2html
~/navsim_workspace
├── navsim (devkit)
├── exp
└── dataset
    ├── maps
    ├── navsim_logs
    │    ├── test
    │    ├── trainval
    │    ├── private_test_hard
    │    │         └── private_test_hard.pkl
    │    └── mini
    ├── sensor_blobs
    │    ├── test
    │    ├── trainval
    │    ├── private_test_hard
    │    │         ├── CAM_B0
    │    │         ├── CAM_F0
    │    │         ├── ...
    │    └── mini
    ├── navhard_two_stage
    │    ├── openscene_meta_datas
    │    ├── sensor_blobs
    │    ├── synthetic_scene_pickles
    │    └── synthetic_scenes_attributes.csv
    ├── warmup_two_stage
    │    ├── openscene_meta_datas
    │    ├── sensor_blobs
    │    ├── synthetic_scene_pickles
    │    └── synthetic_scenes_attributes.csv
    └── private_test_hard_two_stage
         ├── openscene_meta_datas
         └── sensor_blobs
```

按照上述结构，在 `~/.bashrc` 中添加环境变量：

```bash
export NUPLAN_MAP_VERSION="nuplan-maps-v1.0"
export NUPLAN_MAPS_ROOT="$HOME/navsim_workspace/dataset/maps"
export NAVSIM_EXP_ROOT="$HOME/navsim_workspace/exp"
export NAVSIM_DEVKIT_ROOT="$HOME/navsim_workspace/navsim"
export OPENSCENE_DATA_ROOT="$HOME/navsim_workspace/dataset"
```

⏰ **提示：**
- `navhard_two_stage` 用于在本地以双阶段伪闭环方式测试模型表现。
- `warmup_two_stage` 体量更小，适合在 [Hugging Face Warmup 榜单](https://huggingface.co/spaces/AGC2025/e2e-driving-warmup) 验证与测试。你在 `warmup_two_stage` 的本地结果应与提交到 Hugging Face 后的结果一致。
- `private_test_hard_two_stage` 含有挑战数据。要参与 [Hugging Face CPVR 2025 榜单](https://huggingface.co/spaces/AGC2025/e2e-driving-internal) 的正式挑战，需要用它生成 `submission.pkl`（详见 [Submission](submission.md)）。

## 3. 安装 navsim-devkit

最后，为 navsim 创建新环境并安装依赖：

```bash
conda env create --name navsim -f environment.yml
conda activate navsim
pip install -e .
```

至此，NAVSIM 开发套件的下载与安装就绪，可继续进行 ReCogDrive 相关实验。

