# 扩展预测驾驶员模型分数（EPDMS）

在开放式规划（open-loop planning）相关研究中，由于评估指标关注范围较窄，或不同项目间对同一指标的定义不一致，想要进行公平比较是很有挑战的。  
为了解决这一问题，NAVSIM v1 提出了 PDM Score（PDMS），它由若干子指标和乘法惩罚项组合而成。

在 NAVSIM v2 中，EPDMS 在 PDMS 的基础上进行了扩展，引入了：

- 2 个新的加权指标（LK 和 EC）
- 2 个新的乘法类指标（DDC 和 TLC）
- 假阳性惩罚过滤（False-positive penalty filtering）

其中，车道保持子指标（Lane Keeping, LK）用于惩罚长时间偏离车道中心线的轨迹。  
在十字路口区域，由于中心线标注常常与传感器感知到的真实车道线不完全一致，LK 指标会被禁用。  
此外，NAVSIM v2 更加重视驾驶舒适度。  
在 NAVSIM v1 中的舒适度指标（Comfort, C）被改进为历史舒适度（History Comfort, HC），不仅评估轨迹本身的动态特性，也衡量轨迹与车辆历史运动的一致性。  
同时，新引入的扩展舒适度（Extended Comfort, EC）会比较相邻帧输出轨迹及其动态状态（加速度、加加速度等）的差异。帧间变化过大将被视为不舒适行为，从而降低得分。

新的驾驶方向遵守（Driving Direction Compliance, DDC）和交通灯遵守（Traffic Light Compliance, TLC）子指标扩展了在评估中可检测和惩罚的违规行为范围。  
另外，为减少假阳性惩罚，当人类驾驶员也对某次违规行为负责时，我们会禁用对该情形的惩罚。  
这可以避免在某些必须暂时违规才能完成合理驾驶目标的场景中不公平地惩罚规划器，例如为了绕过静止障碍物而短暂驶入对向车道。

EPDMS 的完整组成如下所示（新指标用粗体标出）：

Metric | Weight | Range |
|---|---|---|
No at-fault Collisions (NC) | multiplier | {0, 1/2, 1} |
Drivable Area Compliance (DAC) | multiplier | {0, 1} |
**Driving Direction Compliance** (DDC) | multiplier | {0, 1/2, 1} |
**Traffic Light Compliance** (TLC) | multiplier | {0, 1} |
Ego Progress (EP) | 5 | [0, 1] |
Time to Collision (TTC) within bound | 5 | {0, 1} |
**Lane Keeping** (LK)  | 2 | {0, 1} |
**History Comfort** (HC) | 2 | {0, 1} |
**Extended Comfort** (EC) | 2 | {0, 1} |

EPDMS 定义为：

<br>

$$\text{EPDMS} = \left(\prod_{m\in\\{NC, DAC, DDC, TLC\\}} \text{filter}\_m(\text{agent}, \text{human})\right) \cdot  \left( \frac{\sum_{m \in \\{TTC, EP, HC, LK, EC\\}} w_m \cdot \text{filter}\_m(\text{agent}, \text{human}) }{\sum_{m\in \\{TTC, EP, HC, LK, EC\\}} w_m}\right)$$

$$\text{with}\quad \text{filter}_m(\text{agent}, \text{human}) = \begin{cases}
1.0 & \text{if } m(\text{human}) = 0 \\
m(\text{agent}) & \text{otherwise.}
\end{cases}$$

> 直观理解：若人类驾驶员在某个指标上的得分为 0（即没有违规或问题），则该指标不对规划器产生惩罚；否则，使用规划器自身的得分。

<br>

供参考，NAVSIM v1 中使用的 PDMS（其中 HC 为旧版 Comfort, C）定义为：

<br>

$$ \text{PDMS} = \left(\prod_{m\in\\{NC, DAC\\}} m({\text{agent}})\right) \cdot \left(\frac{\sum_{m\in \\{TTC, EP, C\\}} w_m\cdot m(agent)}{\sum_{m\in \\{TTC, EP, C\\}} w_m}\right)$$

<br>
<br>

# 伪闭环聚合（Pseudo Closed-Loop Aggregation）

在 NAVSIM v1 中，所有指标都是在对规划器输出进行 4 秒非反应式仿真后计算得到的：  
背景交通参与者沿着记录轨迹回放，自车由 LQR 控制器根据规划轨迹执行运动。

在新的 NAVSIM v2 评估流程中，我们采用两阶段聚合的方式，在保持开放式设置的同时近似闭环行为。整体流程如下：

1. **第一阶段打分：**  
   - 对初始场景在固定 4 秒时间范围内进行 EPDMS 评估；  
   - 此阶段的仿真与 NAVSIM v1 的流程类似。

2. **第二阶段打分：**  
   - 除初始场景外，测试集中还包含了多个从该初始场景出发的“后继场景”（follow-up scenes）；  
   - 这些后继场景是通过从同一初始场景开始、采用不同 4 秒规划轨迹滚动仿真预先生成的；  
   - 因此，每个后继场景的起始状态会与第一阶段末端状态不同，例如具有横向偏移或不同的速度；  
   - 在每个后继场景上，同样在 4 秒时间范围内计算 EPDMS。

3. **加权与聚合：**  
   - 为了模拟闭环仿真效应，每个后继场景在总体得分中的权重取决于其起始状态与提交规划器在第一阶段结束状态的接近程度；  
   - 对更接近规划器终点状态的后继场景赋予更高权重，具体采用高斯核进行权重计算；  
   - 首先对所有第二阶段得分进行加权聚合，随后将第一阶段与第二阶段的结果做乘法组合，得到最终的聚合指标。

# 运行评估

要为某个 agent 评估 PDM/EPDMS 指标，可以运行：

```bash
cd $NAVSIM_DEVKIT_ROOT/scripts/evaluation/
./run_cv_pdm_score_evaluation.sh
```

默认情况下，该脚本会为一个简单的恒速 [规划基线](https://github.com/autonomousvision/navsim/blob/main/docs/agents.md#output) 生成评估 csv。  
你可以修改脚本以评估自己的规划 agent。

例如，可以在 `$NAVSIM_DEVKIT_ROOT/navsim/navsim/planning/script/config/common/agent/my_new_agent.yaml` 下为你的 agent 添加一个新的配置文件。  
此时，只需在脚本中添加 `agent=my_new_agent` 的 override，即可对自己实现的 agent 进行评估。  
你可以在 `run_human_agent_pdm_score_evaluation.sh` 中找到示例。


