# CD-Agent：Agent 与 Harness 核心架构

> 技术背景速读；源于 CD-Agent 工作分支 `claude/project-thread-2tuaib`（2026-11-13，同步基准 `25bc69acc948817c287922735579dcab81917bf0`）。具体实现以源仓库为准。

## 核心定位

CD-Agent 研究由 Agent 调度工具完成遥感双时相变化检测，而非只把一个检测器的预测交给模型点评。输入通常包含 T1/T2 两期影像、自然语言任务指令，以及部分任务中的初始变化掩码、空间范围或用户反馈；输出为变化掩码、摘要和完整调用轨迹。

早期 `ovcd_v0` 将固定的检测/修正步骤程序化串联；后来的 `agent_baseline` 把分割、比较、核验、区域编辑拆成原子工具，允许规划器根据中间结果决定下一步。

## 执行闭环

```text
双时相图像 + 指令 + 可选掩码/约束
              ↓
         Planner（LLM）
              ↓ 选择工具 / 参数
   Harness：预算、校验、状态与工具调度
              ↓
 看图 / SAM3 分割 / 实例匹配 / 视觉核验
 / 掩码组合 / 删除、添加、修边界 / 其他任务工具
              ↓
     中间句柄 + 统计 + 可视化
              ↺ 返回 Planner
              ↓
      验证与提交（submit）
              ↓
       输出掩码 + 轨迹 + 指标
```

## 关键机制

- **Planner**：通过工具调用选择动作；规划器和视觉核验器在部分配置中是两个独立角色。
- **Toolbox**：例如 `view`、`inspect`、`segment_text`、`match_instances`、`ask`、`select`、`to_mask`、`combine`、`morph`、`remove_region`、`add_region`、`refine_region`、`submit`。并非全部工具对每项任务同时开放。
- **Session / 状态仓库**：图像、掩码和实例集以句柄管理；模型看到受控的工具结果，而不是直接获得所有大型张量。
- **Harness**：限制规划轮次、调用次数和参数；处理错误、状态更新、强制回退与提交；支持对照配置和任务扩展。历史实现中典型预算为每图最多 30 次成功规划器响应，但具体版本以实验配置为准。
- **Verifier**：用于区域层面视觉核验及提交前验证；避免把核验能力与规划策略混为一谈。
- **Recorder / Replay**：记录模型消息、工具调用、图像、状态变化、最终掩码和评测数据。真值只用于记录/评价，不应回传给 Agent 决策。
- **任务泛化**：可按目标类别、变化方向、区域约束、初始结果和反馈类型调整输入与工具，而无需为每个任务重写整个循环。

## 与固定路线、规则方法的区别

固定路线按预先规定的步骤调用基础模型；规则方法基于固定条件选择或过滤；Agent 允许根据图像和工具返回结果动态行动。对比时需要查看相同任务、相同基础工具与对应实验配置。

## 代码及原始展示

- 历史基线代码：`code/agent_baseline/loop_json.py`、`tools.py`、`state.py`、`planner.py`、`recorder.py`、`backends.py`。
- 开发与实现说明：`code/agent_baseline/IMPL_NOTES.md`、`README_generalization.md`。
- 原始仓库的架构/回放材料：`results/ovcd_v0_review.html`、`results/pipeline.html`、`doc/agent_baseline/index.html`，以及多任务回放 `results/agent_baseline/demo_g3/`（请到源仓库对应分支核对）。
- 本精简仓库的代码可能落后于工作分支；**不要以此副本判断最新实现细节**。
