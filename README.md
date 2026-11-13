# CD-Agent — Gemini 阅读版项目背景

本仓库是 [CD-Agent](https://github.com/ethan-404-sys/CD-Agent) 的**精简阅读副本**，面向 Gemini 等外部模型。目的：快速了解研究背景、Agent/Harness 架构、任务设置和已有实验结果；**不是**最新可运行代码的完整镜像。

**内容来源**：CD-Agent 工作分支 `claude/project-thread-2tuaib`，参考提交 `25bc69acc948817c287922735579dcab81917bf0`；2026-11-13 整理。任务页面已同步，架构和结果采用面向阅读的简表。原有大量代码与旧数据文件作为历史参考保留，**不要把旧文件当成最新研究版本**。

## 先读这四段（即使只能读取 README）

### 1. 背景

研究对象是**遥感双时相变化检测智能体（Change Detection Agent）**：给定同一区域两个时相的遥感图像与自然语言要求，智能体调用视觉基础模型、图像处理和判断工具，生成或更新目标变化掩码，并记录全过程。与“固定检测流程 + 最后调用语言模型”不同，Agent 可以依据中间观察和工具结果动态选择动作；不同任务可以要求类别、变化方向、空间限制、视觉示例或用户反馈。

### 2. 核心 Agent / Harness 架构

```text
T1/T2 影像 + 自然语言指令 + 可选初始掩码/约束
                      ↓
                  Planner
                      ↓ tool call
    Harness：工具 schema/预算/状态管理/失败处理
                      ↓
    view / segment_text(SAM3) / match_instances
    ask(视觉核验) / combine / refine / add / remove ...
                      ↓
                状态句柄与反馈
                      ↺
               验证、submit
                      ↓
          变化掩码 + 摘要 + 轨迹
```

规划器与核验器可分工；中间掩码与实例通常存放在 Session 中，通过句柄传递。记录器保留工具链、状态和结果；真值只用于评测，不反馈给 Agent。可在 `AGENT_ARCHITECTURE.md` 查看更详细的角色分工。

### 3. 任务体系

任务按照 **T1–T5 × C0–C4** 构建（并非所有组合都适用）：

- **T1** 常见目标类别变化；**T2** 稀缺目标类别变化；**T3** 根据视觉示例定义变化；
- **T4** 根据文字描述、指代或空间关系定义变化；**T5** 更新已有检测结果（掩码、反馈、地图等）。
- **C0–C4** 表示额外条件或约束，如变化方向、范围等；具体组合与编号以任务配置页面为准。

代表性任务：T1.1 × C0（指定类别从零检测）、T1.1 × C1（新建/拆除等方向）、T5.1（已有掩码修正）、T5.2/T5.3（点框/语言反馈）。详细定义见 `TASK_SYSTEM.md` 与 `doc/agent_baseline/task_system.html`（可视化原件）。

### 4. 已有实验（只陈列数据，留给读者自行判断）

以下为工作分支 2026-11-10 左右汇总的**代表性** F1（%），数据集和任务不可交叉比较：

| 任务 | Agent | 对照 |
| --- | ---: | ---: |
| T1.1 × C0，LEVIR 验证 | 85.76（G3） | 规则 86.35 |
| T1.1 × C0，LEVIR 测试 | 82.63（G1） | 规则 80.97 |
| T1.1 × C1，SECOND 新建 | 71.31（G3） | 相对规则 +7.57 |
| T1.1 × C1，SECOND 拆除 | 48.45（G3） | 相对规则 +10.49 |
| T1.2，SECOND | 41.22 | 相对固定路线 −5.04 |
| T1.3，SECOND | 46.53 | 相对固定路线 −14.35 |
| T3.1，SECOND | 20.34 | 相对固定路线 +0.95 |
| T5.2，LEVIR 验证，k=1 / 3 | 84.19 / 84.59 | 相对给定掩码 +1.03 / +1.43 |

其他任务（T1.1 × C2/C4、T5.1/T5.3/T5.4、WHU 等）与对照见 `EXPERIMENT_RESULTS.md`。这些是特定配置的历史结果，若需最新详细实验请回原仓库核对。

## 重点阅读文件

| 文件 | 内容 |
| --- | --- |
| [TASK_SYSTEM.md](TASK_SYSTEM.md) | 任务分类与示例的简明文字版 |
| [AGENT_ARCHITECTURE.md](AGENT_ARCHITECTURE.md) | Planner、Harness、Toolbox、Verifier 与 Recorder |
| [EXPERIMENT_RESULTS.md](EXPERIMENT_RESULTS.md) | 多任务代表性数字 |
| [doc/agent_baseline/task_system.html](doc/agent_baseline/task_system.html) | **任务设置可视化 HTML** |
| [doc/cd_agent_task_design.html](doc/cd_agent_task_design.html) | 任务设计背景 HTML |
| [code/agent_baseline/](code/agent_baseline/) | 旧版基线实现，供查阅，不作为最新代码 |
| [results/agent_baseline/](results/agent_baseline/) | 旧版实验数据与分析快照 |

**相关可视化（源仓库）**：[OVCD 复核页](https://github.com/ethan-404-sys/CD-Agent/blob/main/results/ovcd_v0_review.html)、[流水线页面](https://github.com/ethan-404-sys/CD-Agent/blob/main/results/pipeline.html)。这些主要是背景与历史流程图，不能替代当前 Agent 架构文字。

> 给 Gemini：如果你实际只读到了 README，请直接说明；不要声称已经阅读了链接中的文件。用户会单独提供当前研究问题和需要你完成的任务。
