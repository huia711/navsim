# 数据集划分与过滤后的训练 / 测试划分

NAVSIM 框架使用多种数据集划分来标准化训练与评估 agent。  
所有划分均基于 OpenScene 数据集，该数据集被划分为 `mini`、`trainval`、`test` 三个子集，它们可以分别下载。

你可以直接在这些标准划分上进行训练与评估（见下表中的 `OpenScene` 一行）。  
或者，你也可以在**经过筛选的训练 / 验证划分**上训练和评估（表中的 `NAVSIM` 行），这些划分筛选出了更具挑战性的场景，是**更推荐**用于高效产出可比且具有竞争力结果的选项。

与“数据集划分”直接对应于可下载的日志集合不同，“训练 / 测试划分”是通过场景过滤器（scene filters）实现的，用于定义从这些日志中如何提取具体的场景。

NAVSIM 的训练 / 测试划分是对 OpenScene 标准划分的子采样（subsample）。  
此外，NAVSIM 的划分之间是可以重叠的，而标准划分（`trainval`、`test`、`mini`）之间是不重叠的。  
具体来说：

- `navtrain` 基于 `trainval` 数据；
- `navtest` 与 `navhard_two_stage` 基于 `test` 数据。

由于 `trainval` 的传感器数据体积非常大，我们提供了单独的下载链接，仅包含 `navtrain` 所需的帧。  
这方便只想运行 `navtrain` 训练划分的用户。如果你已经下载了完整的 `trainval` 传感器数据，则**无需**重复下载 `navtrain` 的传感器帧。  
日志（logs）始终对应完整的数据集划分。

## 总览

下表给出了 NAVSIM 支持的训练与测试划分概览。

<table border="0">
    <tr>
        <th></th>
        <th>名称</th>
        <th>描述</th>
        <th>日志大小</th>
        <th>传感器数据大小</th>
        <th>配置参数</th>
    </tr>
    <tr>
        <td rowspan="3">OpenScene</td>
        <td>trainval</td>
        <td>用于训练和验证 agent 的大规模划分，包含常规驾驶记录。对应 nuPlan，并下采样至 2Hz。</td>
        <td>14GB</td>
        <td>>2000GB</td>
        <td>
        train_test_split=trainval
        </td>
    </tr>
    <tr>
        <td>test</td>
        <td>用于测试 agent 的小规模划分，包含常规驾驶记录。对应 nuPlan，并下采样至 2Hz。</td>
        <td>1GB</td>
        <td>217GB</td>
        <td>
        train_test_split=test
        </td>
    </tr>
    <tr>
        <td>mini</td>
        <td>用于演示的较小划分，包含常规驾驶记录。对应 nuPlan，并下采样至 2Hz。</td>
        <td>1GB</td>
        <td>151GB</td>
        <td>
        train_test_split=mini
        </td>
    </tr>
    <tr>
        <td rowspan="3">NAVSIM</td>
        <td>navtrain</td>
        <td>标准 NAVSIM 训练划分，包含非平凡驾驶场景。传感器数据可通过 <a href="https://github.com/autonomousvision/navsim/blob/main/download/download_navtrain_aws.sh">download_navtrain_aws.sh</a>（AWS 下载）或 <a href="https://github.com/autonomousvision/navsim/blob/main/download/download_navtrain_hf.sh">download_navtrain_hf.sh</a>（HuggingFace 下载）单独获取。</td>
        <td>14GB</td>
        <td>445GB*</td>
        <td>
        train_test_split=navtrain
        </td>
    </tr>
    <tr>
        <td>navtest</td>
        <td>标准 NAVSIM 测试划分，包含非平凡驾驶场景。作为对 `test` 划分的过滤子集提供。</td>
        <td>983MB</td>
        <td>223GB</td>
        <td>
        train_test_split=navtest
        </td>
    </tr>
    <tr>
        <td>navhard_two_stage</td>
        <td>NAVSIM v2 中用于测试的标准划分，包含真实与合成驾驶场景。合成帧可通过 <a href="https://github.com/autonomousvision/navsim/blob/main/download/download_navhard_two_stage.sh">download_navhard_two_stage.sh</a> 下载。</td>
        <td>892MB</td>
        <td>31GB</td>
        <td>
        train_test_split=navhard_two_stage
        </td>
    </tr>
    <tr>
        <td rowspan="2">Competition</td>
        <td>warmup_two_stage</td>
        <td>用于在 Hugging Face 上验证提交流水是否正确的预热测试划分。合成帧可通过 <a href="https://github.com/autonomousvision/navsim/blob/main/download/download_warmup_two_stage.sh">download_warmup_two_stage.sh</a> 下载。</td>
        <td>27MB</td>
        <td>1.2G</td>
        <td>
        train_test_split=warmup_two_stage
        </td>
    </tr>
    <tr>
        <td>private_test_hard_two_stage</td>
        <td>用于挑战榜单（Hugging Face）上的私有测试划分。原始与合成帧可通过 <a href="https://github.com/autonomousvision/navsim/blob/main/download/download_private_test_hard_two_stage.sh">download_private_test_hard_two_stage.sh</a> 下载。</td>
        <td>14MB</td>
        <td>11GB</td>
        <td>
        train_test_split=private_test_hard_two_stage
        </td>
    </tr>
</table>

（*不含历史帧时约 300GB）

## Splits 说明

标准划分 `trainval`、`test` 和 `mini` 来自 OpenScene 数据集。  
需要注意的是，这些数据对应 nuPlan 数据集，但频率降低为 2Hz。  
你可以通过 [download](../download) 目录下的脚本在 Hugging Face 上下载所有标准划分。

NAVSIM 还提供了 `trainval` 划分的子集 / 过滤版本 `navtrain`。  
`navtrain` 使训练方案更加标准化，同时所需的传感器数据存储远小于完整 `trainval`（445GB vs. 2100GB）。  
如果你的 agent 不需要历史传感器输入，可以下载不含历史的 `navtrain`，此时只需要约 300GB 的存储。  
需要注意的是，`navtrain` 的传感器数据可以通过 <a href="https://github.com/autonomousvision/navsim/blob/main/download/download_navtrain_aws.sh">download_navtrain_aws.sh</a> 或 <a href="https://github.com/autonomousvision/navsim/blob/main/download/download_navtrain_hf.sh">download_navtrain_hf.sh</a> 单独下载，但依然需要配合 `trainval` 的日志数据使用。

`navtest` 划分为 NAVSIM v1 提供标准化测试场景，并附带场景过滤器。  
类似地，`navhard_two_stage` 划分则是 NAVSIM v2 中用于伪闭环评估的测试划分。  
`navtrain`、`navtest` 和 `navhard_two_stage` 均经过过滤，以提高划分中场景的“有趣程度”（例如更加具有挑战性）。

在 Hugging Face 挑战中，我们提供了 `warmup_two_stage` 与 `private_test_hard_two_stage`，分别用于预热赛道与正式挑战赛道。  
**不允许** 使用 `test` / `navtest` / `navhard_two_stage` / `warmup_two_stage` / `private_test_hard_two_stage` 中的数据来训练你的挑战赛提交模型。  
你可以使用其他任何公开数据集或预训练权重。  
此外，为了符合奖项资格，你在提交的技术报告中必须**明确说明**使用了哪些数据。

## 故障排查

由于部分用户在下载 `navtrain` 时遇到缺失文件的问题，我们提供了 `.tgz` 文件的 MD5 校验和，以帮助识别损坏的下载内容。  
建议在不删除已有 `.tgz` 文件的前提下重新下载 `navtrain`（即删除 [download_navtrain_aws.sh](https://github.com/autonomousvision/navsim/blob/main/download/download_navtrain_aws.sh) 中的第 12 与 22 行），并运行：

```bash
echo "6f92f38d5f03ed852da7872a7122bdd2  navtrain_current_1.tgz" | md5sum -c -
echo "7a72f0a758b5df6cbe4c565920a4869f  navtrain_current_2.tgz" | md5sum -c -
echo "b083fce1428308abb5682a1a150cc1af  navtrain_current_3.tgz" | md5sum -c -
echo "68354ac2c993aa1ebbfac59547fdb840  navtrain_current_4.tgz" | md5sum -c -
echo "dc46ed34d92d5ab9cc1464d67b72fbf6  navtrain_history_1.tgz" | md5sum -c -
echo "fab177bdb79c0c9536da1566d13e5995  navtrain_history_2.tgz" | md5sum -c -
echo "71ed9a2387edc3849921861d7873c7f0  navtrain_history_3.tgz" | md5sum -c -
echo "2cc13aced2f458e50fe4cc2f26d18e07  navtrain_history_4.tgz" | md5sum -c -
```

<details>
<summary>期望输出：</summary>

```bash
navtrain_current_1.tgz: OK
navtrain_current_2.tgz: OK
navtrain_current_3.tgz: OK
navtrain_current_4.tgz: OK
navtrain_history_1.tgz: OK
navtrain_history_2.tgz: OK
navtrain_history_3.tgz: OK
navtrain_history_4.tgz: OK
```

</details>
</br>


