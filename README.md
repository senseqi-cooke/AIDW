# AIDW (AI Development Workspace)

AI 开发工作空间，一个用于协调 AI Agent 进行软件开发的框架。
我使用的是 CodeX 为主控，你也可以用 cc 或者其他的。
** 重点说明 gxd-subagent-shim 这里是没有内容的，后面如果取得原开发者：AI Ghost Lab 的许可后再说。**
---

## 项目结构

```
AIDW/
├── README.md              # 项目说明文档（入口文件）
├── AGENTS.md              # Agent 模式说明文档
├── agent-workflow.md      # 工作流编排核心文档
├── roles/                 # 角色定义目录
│   ├── backend-dev.json   # 后端开发工程师角色
│   ├── frontend-dev.json  # 前端开发工程师角色
│   ├── tester.json        # 测试工程师角色
│   └── ba.json            # 需求分析师角色
├── docs/                  # 需求文档目录（按需求创建子目录）
├── experience/            # 经验总结目录（存放开发过程中的知识沉淀）
└── gxd-subagent-shim/     # 子 agent 调用脚本（用于审计追溯）
```

---

## 核心文件说明

### 1. AGENTS.md
**作用**：这是我使用的Codex 初始化文件。重点增加了定义 AI 的工作模式和使用规则。

**内容**：
- 两种开发模式：默认模式 / 工作流模式
- 语言规范：使用中文沟通
- 经验总结规则：超过 3 分钟的查找/思考过程需记录到 `experience/` 目录

**引用关系**：
- 被 Agent 读取以确定当前工作模式
- 用户以 `需求开发` 或 `需求调整` 开头时，Agent 会读取 `agent-workflow.md`

---

### 2. agent-workflow.md
**作用**：工作流模式的核心编排文档，定义 Subagent Orchestrator 的行为规范。

**核心概念**：
- **Orchestrator（编排器）**：不直接执行任务，只调度 Subagent
- **Subagent（子代理）**：实际执行代码/文档工作的 Agent
- **gxd-subagent-shim**：调用 Subagent 的工具，同时负责审计追溯

**工作流程**（每个任务必走）：
```
A0) Clarify（需求澄清）→ A) Extract（提取）→ B) Plan（计划）
→ C) Trace & Audit（审计）→ D) Dispatch（调度）→ 验收
```

**引用关系**：
- 定义了 `roles/` 目录下角色文件的使用方式
- 通过 `gxd-subagent-shim` 调用 Subagent
- 任务产物存储到 `.artifacts/agent_runs/<RUN_ID>/`

---

### 3. roles/ 目录
**作用**：定义不同角色的技能、约束和输出标准。

**文件关系**：

| 文件 | 角色 | 技能栈 | 主要职责 |
|------|------|--------|----------|
| `backend-dev.json` | 后端开发 | Java 8, Spring Boot, MySQL | 后端接口开发、数据库设计 |
| `frontend-dev.json` | 前端开发 | Vue 3, Element Plus | 前端页面开发、组件封装 |
| `tester.json` | 测试工程师 | JUnit 5, Postman | 单元测试、接口测试、回归验证 |
| `ba.json` | 需求分析师 | UML, 流程图 | 需求分析、验收标准定义 |

**引用关系**：
- 被 `agent-workflow.md` 引用，定义在 `## ROLE LIBRARY` 章节
- 任务中通过 `roles_needed` 和 `role_overrides` 指定所需角色
- Subagent 读取对应 JSON 文件获取详细约束

**示例用法**（在任务 JSON 中）：
```json
{
  "roles_needed": ["backend-dev", "frontend-dev"],
  "role_overrides": {
    "backend-dev": {
      "focus_areas": ["用户 CRUD", "权限校验"],
      "output_standards": "roles/backend-dev.json"
    }
  }
}
```

---

### 4. docs/ 目录
**作用**：存储需求文档，按模块/日期组织。

**命名规范**（来自 `AGENTS.md`）：
- 文件名格式：`docs/{年月}+{处理类型}+{中文描述}.md`
- 例如：`docs/202501+新增需求+用户管理模块.md`

**引用关系**：
- 由 Orchestrator 在需求确认阶段创建
- 需求变更时同步更新
- 被 Subagent 读取以获取上下文

---

### 5. experience/ 目录
**作用**：沉淀开发过程中的经验和知识，用于 AI 的持续学习。

**记录规则**（来自 `AGENTS.md`）：
- 查找/思考超过 3 分钟的过程需要总结
- 内容包含：简要业务逻辑、关键代码位置
- 文件名使用中文

**引用关系**：
- 由 Subagent 或 Orchestrator 写入
- 可作为后续任务的上下文参考

---

### 6. gxd-subagent-shim/ 目录
**作用**：Subagent 调用的 shim 脚本，负责审计追溯。这是AI Ghost Lab 大神用 Python 写的，没有经过他的同意我就不开源了。群友们应该都有的。

**核心功能**：
- 通过 `gxd-subagent-shim create/resume` 调用 Subagent
- 自动写入追溯数据到 `.artifacts/agent_runs/<RUN_ID>/`
- 维护 `meta.json`、`events.jsonl`、`index.md`

**追溯结构**：
```
.artifacts/agent_runs/<RUN_ID>/
├── meta.json          # 元数据
├── events.jsonl       # 事件日志（追加写入）
├── index.md           # 可读索引
└── steps/<step_id>/
    └── rounds/R<k>/
        ├── request.json
        └── response.json
```

**引用关系**：
- 被 `agent-workflow.md` 中的 Orchestrator 调用
- 是审计追溯的**唯一可信来源**

---

## 文件引用关系图

```
┌─────────────────────────────────────────────────────────────┐
│                      用户输入                                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  AGENTS.md                                                │
│  → 确定工作模式（默认/工作流）                               │
└───────────────────────────┬─────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
        ┌──────────────┐      ┌──────────────────────┐
        │ 默认模式      │      │ 工作流模式           │
        │ 直接执行      │      │ 读取 agent-workflow  │
        └──────────────┘      └──────────┬───────────┘
                                          │
                                          ▼
                        ┌──────────────────────────────────┐
                        │  agent-workflow.md (Orchestrator) │
                        │  → 解析任务 → Plan → Dispatch     │
                        └──────────────┬───────────────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                  ▼                  ▼
            ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
            │ roles/       │  │ gxd-subagent │  │ docs/            │
            │ 读取角色定义  │  │ -shim 调用   │  │ 创建需求文档      │
            └──────────────┘  │ Subagent     │  └──────────────────┘
                              └──────┬───────┘
                                     │
                                     ▼
                        ┌──────────────────────────────┐
                        │ Subagent 执行 → 产物写入      │
                        │ .artifacts/agent_runs/       │
                        └──────────────┬───────────────┘
                                       │
                                       ▼
                        ┌──────────────────────────────┐
                        │ experience/                  │
                        │ 沉淀经验（超过3分钟的过程）   │
                        └──────────────────────────────┘
```

---

## 使用流程

### 模式1：默认模式
直接沟通，AI 自己规划并执行任务。需求文档输出到 `/docs` 目录。

### 模式2：工作流模式
1. 用户输入 `需求开发` 或 `需求调整` 开头
2. Orchestrator 读取 `agent-workflow.md`
3. 完成需求澄清 → 提取 → 计划 → 调度 → 验收
4. Subagent 通过 `gxd-subagent-shim` 调用
5. 产物和审计数据存入 `.artifacts/agent_runs/`

---

## 开发语言
- 文档和沟通：**中文**
- 代码注释：**中文**
