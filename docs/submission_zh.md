# 提交到排行榜

NAVSIM 在 Hugging Face 上提供了官方排行榜（[Leaderboard 2024](https://huggingface.co/spaces/AGC2024-P/e2e-driving-navsim)、[Leaderboard 2025 Warmup](https://huggingface.co/spaces/AGC2025/e2e-driving-warmup-iccv)、[Leaderboard 2025](https://huggingface.co/spaces/AGC2025/e2e-driving-2025)）。  
所有评估均在服务器上使用官方评估脚本完成，从而避免了不同项目之间因指标定义略有差异而产生的歧义。  
本页面主要介绍 **2025 榜单** 相关的提交流程；若你希望向 2024 榜单（NAVSIM v1, 使用 `navtest` 评估划分）提交结果，请使用 [NAVSIM v1.1 分支](https://github.com/autonomousvision/navsim/tree/v1.1)。

在 [NAVSIM v2 End-to-End Driving Challenge 2025](https://huggingface.co/spaces/AGC2025/e2e-driving-2025) 中，我们将使用 `private_test_hard_two_stage` 划分作为正式排行榜的测试集。  
下面将介绍如何生成一个**有效的提交文件**。

## 规则

- [**通用规则（Leaderboard 2025）**](https://opendrivelab.com/challenge2025/#rule)
- **开源代码与模型（Leaderboard 2024）**：

  - 我们将定期（约每 6 个月）移除排行榜上**未提供对应开源训练 / 推理代码与预训练权重**的条目。  
    即使条目因缺少信息被移除，一旦补充好相应代码后，仍可重新提交。
  - 需要通过设置提交文件中的 `TEAM_NAME` 变量来提供代码仓库链接，例如：`"<a href=Link/to/repository>Method name</a>"`。  
    若仓库在首次提交后才创建（或更新），也可以在排行榜界面中编辑已有条目的 `TEAM_NAME`。

## [Leaderboard 2025](https://huggingface.co/spaces/AGC2025/e2e-driving-2025)

⏰ **注意：** 为保证公平性，我们不会提供带有标注的评估数据集。  
因此，**只有通过提交到排行榜才能获取你的得分**。  
受评估数据量所限，从成功提交到结果出现在排行榜上，预计需要 **约 2 小时**。

### 生成排行榜提交文件

要向排行榜提交结果，你需要生成一个 pickle 文件，其中包含每个测试场景的预测轨迹。  
NAVSIM 提供了帮助你生成该 pickle 文件的脚本。

你可以查看 NAVSIM 仓库中的 `run_cv_create_submission_pickle.sh`（位于 [NAVSIM](https://github.com/autonomousvision/navsim/blob/main/docs/install.md) 仓库）：  
该脚本会为 ConstantVelocity agent 生成提交用的 pickle 文件。  
若要为你自己的 agent 生成提交文件，可以将脚本中的 `agent` override 改为 `TRAIN_TEST_SPLIT=private_test_hard_two_stage`。  
**注意：你必须在 `run_create_submission_pickle.sh` 中设置 `TEAM_NAME`、`AUTHORS`、`EMAIL`、`INSTITUTION` 和 `COUNTRY`，否则生成的提交文件无效。**

## 提交描述（Submission Description）

之后，你需要将生成的结果作为 **一个 HuggingFace 模型** 上传。  
排行榜同样接受私有模型，因此你可以选择不公开你的提交文件。

具体步骤如下：

1. 在 Hugging Face 网站右上角点击头像，选择 `+New Model`；
2. 填写表单信息并上传你生成的 `submission.pkl` 文件。

## 提交流程（Submission Process）

- 在比赛空间左侧面板中选择 `New Submission`；  
- 在表单中粘贴你刚刚创建的 Hugging Face 模型链接；  
- 点击 `Submit` 完成一次新的提交。

**注意：你每天只能提交一次。**

---

## [Leaderboard 2025 Warmup](https://huggingface.co/spaces/AGC2025/e2e-driving-warmup-iccv)

Warmup 榜单会在一小部分场景上测试你的模型，帮助你在正式提交到主榜单前排查技术问题。  
推荐流程如下：

### 本地复现 Warmup 分数

你可以在本地使用 [NAVSIM](https://github.com/autonomousvision/navsim/blob/main/docs/install.md) 仓库复现 Warmup 得分，并与 Hugging Face 上的结果进行对比。  
建议步骤为：

1. **下载数据集** —— 参见 [dataset](install.md) 页面中的说明；
2. **缓存数据** —— 运行脚本 `scripts/evaluation/run_metric_caching.sh`，并设置 `TRAIN_TEST_SPLIT=warmup_two_stage`；
3. **运行评估** —— 使用脚本 `scripts/evaluation/run_cv_pdm_score_evaluation.sh` 以及你自己的模型。  
   - 若在缓存数据时指定了 `metric_cache_path`，请在评估时使用相同路径；  
   - 将 `TRAIN_TEST_SPLIT=warmup_two_stage`，以确保与 Hugging Face 返回的得分一致。

### Warmup 榜单提交

与正式榜单相同，你需要生成一个包含所有测试场景轨迹的 pickle 文件。  
NAVSIM 提供了生成该 pickle 的脚本。

可以查看仓库中的 `run_cv_create_submission_pickle_warmup.sh`（位于 [NAVSIM](https://github.com/autonomousvision/navsim/blob/main/docs/install.md) 仓库）：  
该脚本会为 ConstantVelocity agent 创建提交文件。  
你可以将脚本中的 `agent` override 改为 `TRAIN_TEST_SPLIT=warmup_two_stage`，以在 Warmup 划分上生成你自己的提交文件。  
**同样地，你需要在 `run_create_submission_pickle.sh` 中设置 `TEAM_NAME`、`AUTHORS`、`EMAIL`、`INSTITUTION` 和 `COUNTRY` 才能生成有效的提交文件。**

你应当能够在本地评估中获得与服务器评估结果一致的 Warmup 得分。

---

# FAQ

## 如何查看我的提交？

你可以在比赛空间的 **My Submissions** 标签页中查看提交状态。  
选中某条提交并点击底部的 **Update Selected Submissions**，即可刷新该提交在公共排行榜上的评估状态。

## 我的评估结果会对外公开吗？

**公共排行榜** 会持续展示所有队伍在该 Warmup 赛道上的最佳成绩。  
你可以在比赛结束后更改队伍名称。

若你希望在比赛期间保持匿名，可以先使用一个临时队名参赛，比赛结束后再将队伍名称切换为真实队名。

## 我可以在不公开提交文件的前提下进行提交吗？

可以。比赛空间接受 Hugging Face 私有模型。  
事实上，我们**推荐**你以私有模型的形式提交，以保持提交文件的私密性。

## 我的评估状态显示 “Failed”，如何获取错误信息？

首先，确保你的提交文件格式正确（参考“提交准备”部分），并且在“New Submission”页面中填入了正确的 Hugging Face **model** 链接（格式类似 `用户名/模型名`）。  
请确认你上传的 pickle 文件名为 `submission.pkl`。

如果你已经确认格式无误，请联系 [Wei Cao](mailto:dave.caowei@gmail.com)。  
邮件中应包含对应提交的 **Submission ID**（可在 **My Submissions** 标签页找到）。

邮件模板示例：

```text
Email Subject: [CVPR E2E] Failed submission - {Submission ID}
Body:
    Your Name: {}
    Team Name: {}
    Institution / Company: {}
    Email: {}
```

## 提交页面显示 “Invalid Token”，该怎么办？

这通常意味着你当前没有登录比赛空间，或者由于长时间未操作（>1 天）而被自动登出。  
请刷新页面，在左侧面板底部点击「Login with Hugging Face」，重新登录后再次提交。

## 无法访问 “My Submissions” 页面怎么办？

大概率是你尚未登录当前比赛空间。  
请刷新页面，在左侧面板底部点击「Login with Hugging Face」，重新登录后重试。


