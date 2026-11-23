---
id: "cb6bd015-c45f-4de5-9dc2-7aba240248a2"
title: "Mnemon 记忆空间配置：存储范围 vs 记忆空间范围 与三空间方案"
description: "源码核对的 storageScope 与 memorySpacesScope 区别、长度限制、面向通信工程研究生的三个记忆空间命名/描述方案与数据落点；供后续记忆空间配置与路由类问题复用。"
status: "active"
created_at: "2026-11-23T14:52:30.576Z"
updated_at: "2026-11-23T14:52:30.576Z"
content_hash: "ce3deb1c38278342f65b0b9721b3e204c6597446ddf6f53c09a5ce1e6850e170"
source_paths:
  - "lib/client.js"
  - "lib/index.js"
  - "lib/types/contracts.d.ts"
  - "lib/contracts.d.ts"
session_ids:
  - "mnemon-review-v1-547892a9-931d-4b15-bb6a-80a1a0036e35"
memory_body_ids:
  []
---

# Mnemon 记忆空间配置说明

## 背景
用户（通信工程专业研究生，涉及课题组项目与学校课程）咨询如何为"个人身份 / 课题组科研 / 课程学习"配置记忆空间，过程中对"全局"一词产生混淆。本记录固化源码核对结论与最终推荐方案，供后续同类问题复用。以下为推荐方案，用户尚未明确确认已实际应用。

## 两个互不相干的"范围"（源码核对）
- `storageScope`（存储范围）：运行时记忆、项目档案（Documents）存哪里、是否按工作区分。用户当前为 `workspaces`（集中存储·按工作区隔离）。
- `memorySpacesScope`（记忆空间范围）：记忆空间本身和 Provider 连接是所有工作区共享，还是跟随各工作区的存储根。
  - 取值 `storage` | `global`，默认 `storage`。源码：`memorySpacesScope: z.union(["storage", "global"]).default("storage")`，config 默认亦为 `storage`。
  - `storage`（默认）："与 MEMORY.md 保存在同一存储根目录" → 跟随存储范围，按工作区各一套。
  - `global`："所有工作区共用全局记忆空间与 Provider 连接；MEMORY.md 仍按存储范围隔离" → 一套空间所有工作区共用。
  - 源码位置：`lib/client.js`（config schema、默认值、目录解析）、`lib/index.js`、`lib/types/contracts.d.ts`；中文文案见 locale 中 `config.memorySpacesScope*`（记忆空间范围 / 记忆空间范围存储提示 / 全局提示）。

## 关键澄清（避免混淆）
- "记忆空间是全局整体记忆"指的是**跨会话持久**（相对于每轮注入的运行时记忆），不等于**跨工作区共享**。
- 把空间命名为 `global` 不产生任何共享效果；共享只由 `memorySpacesScope` 设置项决定。
- 改范围不搬移已有数据 → 应先改设置，再建空间（用户当时尚未建空间，故为零成本时机）。

## 长度限制（源码核对）
- 名称上限 100 字符：`MEMORY_SPACE_NAME_MAX_LENGTH = 100`
- 描述上限 1000 字符：`MEMORY_SPACE_DESCRIPTION_MAX_LENGTH = 1000`（`lib/client.js` 中为 `1e3`）
- 位置：`lib/types/contracts.d.ts`、`lib/contracts.d.ts`、`lib/client.js`。

## 最终推荐方案
设置：存储范围保持"集中存储·按工作区隔离"**不动**；`memorySpacesScope` 改为 `global`（所有工作区共用）；Provider 选 Mnemon Native（本地），建完逐一**激活**（激活后才参与召回并允许写入）。

描述是给 AI 的路由表（"这里存什么"+"什么问题时来这里找"），要含具体领域触发词（组会、导师反馈、投稿 deadline、公式推导、实验数据等），而非空话。

三个空间（名称 + 描述，可直接复制）：

1. `个人与通用`
   存放用户身份与跨场景的长期事实：通信工程专业研究生、长期目标、个人偏好与工作习惯、协作方式约定，以及不属于科研或课程的通用信息。当问题涉及"我是谁"、个人习惯、通用偏好，或无法归入科研/课程的事项时，来这里召回。

2. `课题组科研`
   存放课题组项目与科研相关的一切：项目背景与进度、导师布置的任务与反馈、组会讨论与决议、论文写作与投稿状态、实验设计与结果数据、算法与仿真实现细节、技术方案选型、踩过的坑与解决方法、会议与投稿 deadline。当问题涉及某个课题或项目、实验数据、论文、导师要求、技术细节、科研进展时，来这里召回。

3. `课程学习`
   存放学校课程相关内容：课程名称与授课老师、知识点与公式推导、作业题目与解法、考试安排与复习重点、参考教材、学习计划与成绩。当问题涉及某门课程、作业、考试复习、课程知识点或学习成绩时，来这里召回。

可选第 4 个 `文献阅读`（读论文多、需要回溯方法出处时才拆；否则并入课题组科研）：论文标题、作者、会议/期刊、核心贡献、方法论、关键结论、与自己研究的关系、阅读笔记。

## 数据落点（改为 global 后）
```
~/.mnemon/
├── data/
│   ├── 个人与通用/mnemon.db     ← 身份、偏好、跨项目通用事实
│   ├── 课题组科研/mnemon.db     ← 组会、导师、论文、实验
│   └── 课程学习/mnemon.db       ← 课程、作业、考试
└── workspaces/
    ├── <课题组工作区>/  runtime/(USER.md/MEMORY.md) + documents/
    └── <课程工作区>/    runtime/ + documents/
```

## 路由三原则
- 长期事实 → 三个记忆空间（全局共享，Agent 按描述自动路由）
- 项目资料/长结构化材料（实验方案、技术调研、论文大纲）→ 项目档案 Documents（按工作区隔离，不串味）
- 临时约定/当前指令 → MEMORY.md（按工作区隔离）
- 激活后基本无需手动操作；可正常对话让受监督子代理写入，或用 🧠 图标手动存，或开 `auto-capture`；用 `mnemon status` 验证条数。有内容后可用"AI 维护记忆空间元信息"自动优化描述。
- 起步也可只建前两个空间，积累一两周再决定是否加第三/第四个；空空间多且描述接近时反而易选错。
