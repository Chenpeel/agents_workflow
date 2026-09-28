---
name: agents-workflow
description: 当用户要求建立任务工作流, 编写计划和职责边界, 维护任务进度或继续已有工作流时使用.
owner: Chenpeel
---

# 维护任务工作流

## 使用边界

- 仅为明确需要工作流档案的任务启用, 不为普通单步任务强行建档.
- 默认将工作流放在任务所属项目根目录的 `.agents_workflow/`; 用户指定其他目录时以指定位置为准. 不要把任务档案写进本 skill 的安装目录.
- 遵守目标项目的权限, Git 和验证规则; 不用模板中的提示替代真实的需求与验收结果.

## 建立与继续

1. 阅读本目录的 [README.md](README.md) 和需要使用的 [plan.md](template_workflow/plan.md), [responsibilities.md](template_workflow/responsibilities.md), [current.md](template_workflow/current.md).
2. 根据任务确定具体的 kebab-case `TARGET`, 以任务开始当日的本地日期确定 `DATE` (`YYYYMMDD`). 先查找同一任务已有的工作流; 已存在就继续维护, 不因日期变化重建.
3. 在上述工作流目录下新建 `{TARGET}_{DATE}_workflow/`, 从 `template_workflow/` 复制三个必需的 Markdown 模板并替换 `{TARGET}` 与 `{DATE}`; `research/`, `workflow_result/`, `dialogue_context/`, `tmp/` 仅在需要时创建.
4. 填写实际目标, 职责与基线; 保留每个模板顶部的文档定位, 读者, 更新规则和配套文档说明. 不确定的目标与边界先向用户确认, 不编造已落地事实.

## 推进与收尾

| 文件 | 维护规则 |
| --- | --- |
| `plan.md` | 一次性建立长期实施蓝本; 目标变更前说明并取得用户授权. |
| `responsibilities.md` | 一次性明确职责与边界; 边界变更前说明并取得用户授权. |
| `current.md` | 每轮实现或提交后更新已落地, 未落地, 进度与阶段结论; 蓝本获批修改时记录变更. |

- 每轮开工前先读三个文件, 核对当前事实与长期蓝本; 不在 `current.md` 中改写蓝本.
- 只有实现与验收全部完成才将进度写为 100%; 保留工作流目录作为档案, 再按 `plan.md` 的文档要求完成模块 `README.md`.
- 核对三个文件均存在, 模板占位符已替换, 进度百分比与 20 格进度条一致, 记录与实际结果相符.

例如: 为 API 限流建档时默认使用 `项目根目录/.agents_workflow/api-rate-limiting_YYYYMMDD_workflow/`, 其中只复制三个必需文件; 完成实际验收之前, 不把 `current.md` 预写成 100%.
