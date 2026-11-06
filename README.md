# CD-Agent-Baseline

这是 [CD-Agent](https://github.com/ethan-404-sys/cd-agent) 中 **Agent baseline**（LEVIR-CD 上由 Agent 调度原子工具的变化检测 baseline）的精简副本，只放代码、任务设置和设计文档，方便 Gemini 等工具直接阅读。`results/agent_baseline/` 只带文字和数据（md/json/txt），轨迹展示页（每个 1.6–3.8 MB 的 HTML）、旧实验和大文件都留在原仓库。

- 来源：`cd-agent` 分支 `claude/project-thread-2tuaib`，提交 `434b3bd`（2026-11-06 同步）。
- 目录结构与原仓库一致，所以 `code/agent_baseline/__init__.py` 能按相对路径找到 `code/ovcd_v0/`。

## 从哪里读

| 路径 | 内容 |
| --- | --- |
| `doc/agent_baseline/README_ai.md` | **先读这个**：给 AI 的开发文档（背景、规范、最新决定、工具、轨迹格式、优化记录） |
| `doc/agent_baseline/index.html` | 同一内容给人看的版本 |
| `doc/agent_baseline/task_system.html` | 任务设置：任务场景 × 附加约束 |
| `doc/cd_agent_task_design.html` | 变化检测智能体任务设置（T1.1 × C0、T5.1 等编号的定义） |
| `code/agent_baseline/IMPL_NOTES.md` | 实现与维护说明（模块边界、相对任务书的偏离） |
| `code/agent_baseline/README_generalization.md` | 泛化重构说明：目标类别与变化方向参数化（`scopes/`、`tasks.py`）、JSONL 任务清单、可选的沙箱 python 工具 |
| `code/agent_baseline/` | baseline 全部代码：`loop_json.py` 主循环、`tools.py` 工具与 schema、`planner.py`、`backends.py`、`recorder.py`、`prompts/` 系统提示、`skills/`、`workers/`、`analysis/`、`tests/` |
| `code/ovcd_v0/` | 只保留 agent_baseline 直接 import 的依赖模块：`common` `prompts` `mllm` `render` `sam_tools` `sam3_t7` `items` |
| `code/agent_baseline/README_second.md` | SECOND 数据集 C1 实验说明 |
| `results/agent_baseline/` | 结果：`levir_analysis.md`、`data/` 下各轮对比与 SECOND 指标（`second_metrics.md` 等）、展示页的 README |
| `code/probe/sam3_tools.py` | SAM3 构建函数（worker 和 `sam3_soft.py` 用到） |

## 运行测试

```bash
cd code && python -m unittest discover -s agent_baseline/tests -t .
```

只需 numpy、Pillow、scipy；102 个测试在这份副本里通过，其中 1 个（G1 回放）需要服务器上冻结的 `code_g` 源码，这里会自动跳过。真正跑实验需要服务器上的模型、数据和 `results/` 目录，这里没有。

## 没有拷贝的内容

- `results/agent_baseline/demo*/T*.html` 轨迹展示页（共 8 个，约 20 MB），以及 `results/` 下其他旧实验结果
- `code/ovcd_v0/` 中其余的旧实验脚本（`rounds_r*.py`、`figs_*`、`page/` 等）、`code/probe/` 其余脚本
- `previous_work/`、`audit/`、`figures/`、`interim/` 以及其他旧实验文档
