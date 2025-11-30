# 交通 Agent 策略

NAVSIM v2 新增了对**可反应交通 agent**（reactive traffic agents）的支持，使周围车辆能够对自车（ego-vehicle）的行为做出响应。  
然而，自车本身依旧是**非反应式**的：环境的状态更新不会反馈到规划器中。与 NAVSIM v1 相同，自车 agent 需要在一次规划中提交一条完整的轨迹，该轨迹会在整个仿真区间内被执行。

## 可用的交通 Agent 策略

1. **Log-Replay（日志回放，非反应式）**

   - 与 NAVSIM v1 中的行为完全一致，交通参与者严格按照记录好的轨迹进行回放，不会对自车做出反应。

2. **Constant-Velocity（恒速，仅用于调试）**

   - 交通参与者以固定速度沿直线运动，提供一个简单的调试基线。

3. **IDM（Intelligent Driver Model，智能驾驶员模型，反应式）**

   - 与 nuPlan 中的做法类似，该模型为交通参与者提供更真实的驾驶行为，会根据道路情况调整速度和车距。  
   - 行人、静止物体以及其他非车辆 agent 仍将按照记录的日志数据运动。

## 选择交通 Agent 策略

对于单阶段（single-stage）仿真，你可以在运行评估脚本 `navsim/planning/script/run_pdm_score_one_stage.py` 时通过命令行 override 指定交通 agent 策略。

具体示例可以在脚本 `run_cv_pdm_score_evaluation.sh` 的注释部分中找到。比如：

```text
traffic_agents=non_reactive
```
或
```text
traffic_agents=reactive
```

这样你就可以根据评估需求在不同交通 agent 策略之间方便地切换。

在双阶段（two-stage）仿真中（例如面向 Hugging Face 的提交），默认会启用**反应式**交通 agent。

所有可用的交通 agent 策略文档可以在这里找到：  
[navsim/planning/script/config/common/traffic_agents_policy.md](navsim/planning/script/config/common/traffic_agents_policy.md)

## 基于学习的交通仿真

我们计划在 NAVSIM v2 中扩展更多的交通策略，特别是基于学习的交通仿真模型。


