# AI 前沿进展与 Claude Code 实战指南

> 整理日期:2026/06/12
> 主题:从行业进展到团队 Agent 落地的完整知识地图
> 内容来源:技术问答整理

---

## 📚 目录

- [问题一:Claude 创始人提出的"只用写循环"是什么](#问题一claude-创始人提出的只用写循环是什么)
- [问题二:最前沿 AI 行业进展(RAG / MCP / Agent / Computer Use)](#问题二最前沿-ai-行业进展)
- [问题三:Claude Code 的 Agent 设计哲学](#问题三claude-code-的-agent-设计哲学)
- [问题四:如何用 MCP + Skills + Hooks 改造团队专属 Agent](#问题四如何用-mcp--skills--hooks-改造团队专属-agent)
- [附录:核心概念速查表](#附录核心概念速查表)

---

## 问题一:Claude 创始人提出的"只用写循环"是什么

### ❓ 问题
> claude 创始人提出了说自己只用写循环是啥

### ✅ 回答

#### 核心观点:Agent 就是一个 while 循环

> **"An agent is just an LLM in a loop calling tools."**
> (Agent 就是一个在循环里调用工具的 LLM)

来源:Anthropic 官方博客文章 **《Building Effective Agents》**(2024 年 12 月发布)。

#### 它真正想表达的是什么

很多人构建 Agent 喜欢上各种重型框架(LangChain、AutoGPT、CrewAI...),但 Anthropic 的观点是:

```python
# 这就是一个 Agent 的全部
while True:
    response = llm.call(messages, tools=tools)
    if response.is_done():
        break
    result = execute_tool(response.tool_call)
    messages.append(result)
```

**就这么简单。** 一个 `while` 循环 + LLM + 工具调用,本质上就构成了一个 Agent。

#### 为什么这个观点很重要

| 误区 | Anthropic 的主张 |
|---|---|
| Agent 需要复杂编排框架 | 大多数场景一个循环就够 |
| 多 Agent 系统更强 | 单 Agent + 好工具往往更可靠 |
| 需要复杂的 Planning 模块 | 让 LLM 自己规划,你只负责循环 |
| 抽象层越多越好 | 抽象越多,调试越难,延迟越高 |

#### 配套的两个概念区分

Anthropic 在那篇文章里还区分了:

- **Workflow(工作流)**:开发者预定义好步骤的流水线
- **Agent(智能体)**:LLM 自主决定下一步动作的循环

👉 **"只用写循环"对应的就是 Agent 模式**,Claude Code 本身就是这个理念的最佳实践。

---

## 问题二:最前沿 AI 行业进展

### ❓ 问题
> 讲讲最前沿 AI 行业进展;比如从 RAG、MCP、Agent、OpenClaw 这样历史性影响比较大的进展

### ✅ 回答

#### 一张全景图先看清脉络

```
2022.11  ChatGPT 引爆  →  让 AI 第一次"可对话"
   ↓
2023.03  RAG 大爆发    →  让 AI "有记忆/可查资料"
   ↓
2023.06  Function Call →  让 AI "能调用工具"
   ↓
2023-2024 Agent 元年   →  让 AI "能自主决策"
   ↓
2024.09  o1 推理模型   →  让 AI "会深度思考"
   ↓
2024.10  Computer Use →  让 AI "能操作电脑"
   ↓
2024.11  MCP 协议     →  让 AI "标准化连接万物"
   ↓
2025-2026 Coding Agent →  让 AI "能自主编程"
```

---

### 一、RAG(2020 提出 / 2023 大爆发)— 解决"幻觉与记忆"

#### 是什么
**Retrieval-Augmented Generation(检索增强生成)**:把外部知识库的内容,在 LLM 回答前**检索出来塞进 prompt**。

#### 谁提出的
Meta(Facebook AI Research)2020 年论文《Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks》。真正爆发是 2023 年 LangChain + LlamaIndex + 向量数据库(Pinecone、Milvus、Chroma)生态成熟之后。

#### 历史性意义

| 解决的问题 | 价值 |
|---|---|
| 模型知识截止日期问题 | 可接入实时数据 |
| 幻觉问题 | 答案有据可查 |
| 私域知识无法注入 | 企业终于敢用 LLM |
| 上下文长度限制 | 海量数据按需取用 |

#### 演进路线
```
朴素 RAG  →  Advanced RAG  →  GraphRAG(2024,微软)  →  Agentic RAG(2025)
向量检索    重排序+查询改写    图谱+向量混合           Agent自主决定检索策略
```

#### 当前状态(2026)
RAG 已"基础设施化"。重点转向 **GraphRAG**(知识图谱+向量)和 **Agentic RAG**(让 Agent 决定何时/检索什么)。

---

### 二、Function Calling(2023.06)— 解决"AI 只会聊天"

#### 是什么
OpenAI 在 2023 年 6 月发布,让 LLM **结构化地输出工具调用 JSON**。

#### 历史性意义
这是从"**语言模型**"到"**行动模型**"的关键一跃。

```
之前:LLM 输出文字 → 人去解析 → 人去操作
之后:LLM 输出 JSON → 程序解析 → 程序操作
```

这一步直接催生了后来的 Agent 和 MCP。

---

### 三、Agent(2023.03 - 至今)— 解决"AI 不会自主决策"

#### 引爆点
2023 年 3 月 **AutoGPT** 在 GitHub 一周斩获 10 万 star,人类第一次直观感受到"AI 能自己干活"。

#### 关键里程碑

| 时间 | 项目 | 突破 |
|---|---|---|
| 2023.03 | AutoGPT / BabyAGI | 让大众第一次见到自主 Agent |
| 2023.10 | LangChain Agents | Agent 工程化框架 |
| 2024.03 | Devin(Cognition) | 第一个"AI 软件工程师" |
| 2024.06 | CrewAI / AutoGen | 多 Agent 协作 |
| 2024.12 | Anthropic《Building Effective Agents》 | **范式回归**:Agent = LLM in a loop |
| 2025.03 | Manus(中国团队) | 通用 Agent 出圈 |

#### 历史性转折点
2024 年底 Anthropic 那篇文章是一个**冷水**——它指出:

> 多 Agent 编排框架带来的复杂度,往往超过收益。**单 Agent + 好工具 + 简单循环 = 最佳实践**。

这直接影响了 2025 年之后的工程方向,Claude Code 就是这个理念的产物。

---

### 四、推理模型(2024.09 起)— 解决"AI 不会深度思考"

#### 是什么
让模型在回答前**先输出长链思考过程**(Chain-of-Thought 内化),用**测试时计算(Test-Time Compute)** 换取性能提升。

#### 关键事件

| 时间 | 模型 | 突破 |
|---|---|---|
| 2024.09 | OpenAI **o1** | 首个推理模型,数学/代码能力暴涨 |
| 2025.01 | DeepSeek **R1** | **开源+成本暴跌**,引发"DeepSeek 时刻" |
| 2025.02 | Claude 3.7 Sonnet(Extended Thinking) | 推理与对话融合 |
| 2025-2026 | Fable / Opus 4.x 系列 | 推理成为默认能力 |

#### 历史性意义
**Scaling Law 进入新阶段**:从"训练时算力"扩展到"**推理时算力**"。这是过去 2 年最重要的范式转变之一。

DeepSeek R1 还带来一个副作用:**开源模型与闭源差距大幅缩小**,推动整个行业降价 90%。

---

### 五、Computer Use(2024.10)— 让 AI 操作真实软件

#### 是什么
**Anthropic 在 2024 年 10 月发布 Computer Use**,让 Claude 能像人类一样:
- 看屏幕(截图理解)
- 移动鼠标
- 点击/输入
- 操作任意软件

#### 后续跟进

| 时间 | 产品 |
|---|---|
| 2025.01 | OpenAI **Operator**(对标 Computer Use) |
| 2025.03 | Google **Project Mariner**(浏览器 Agent) |
| 2025-2026 | 各家逐步整合到旗舰产品 |

#### 历史性意义
这是 AI 从"**API 世界**"踏入"**真实软件世界**"的关键一步。

- **之前的 Agent**:只能调你提前接好的 API
- **Computer Use 之后**:**任何有界面的软件,AI 都能操作**

意味着遗留系统、私有 SaaS、本地软件全部都能被 AI 接管,影响力堪比"人手"长出来。

---

### 六、MCP(2024.11)— Anthropic 最被低估的贡献

#### 是什么
**Model Context Protocol(模型上下文协议)**:Anthropic 在 2024 年 11 月开源的"**AI 界的 USB-C**"。

#### 解决的问题
之前每个工具/数据源接入 LLM,都要单独适配:
```
N 个模型 × M 个工具 = N×M 套集成
```

MCP 之后:
```
N 个模型 + M 个工具 = N+M 套集成
```

#### 历史性意义

| 维度 | 影响 |
|---|---|
| 标准化 | 取代了 LangChain 各种私有抽象 |
| 生态 | 2025 年内 OpenAI、Google、Cursor 全部跟进采纳 |
| 工程意义 | 让 AI 工具开发从"项目级"变"组件级" |

**这是 Anthropic 用一个协议改写了 AI 工具生态的格局**,堪比 HTTP 之于 Web。

#### 当前状态(2026)
MCP 已成为事实标准,Cursor / Claude Code / Windsurf 等都原生支持,Marketplace 上已有数千个 MCP Server。

---

### 七、Coding Agent(2024-2026)— 当前最火的赛道

#### 关键时间点

| 时间 | 产品 | 意义 |
|---|---|---|
| 2024.03 | **Devin** | 首个 AI 软件工程师概念 |
| 2024.06 | **Cursor** 爆火 | AI Native IDE |
| 2024.12 | **Claude Code** | CLI 模式 + Agent 化 |
| 2025 | **Windsurf / Cline / Aider** | 百花齐放 |
| 2025-2026 | **Codex / Jules / Replit Agent** | 云端自主编程 |

#### 范式分化
现在编程 AI 分成三派:

```
1. Copilot 派 (GitHub Copilot)
   → 补全 + 行内建议,人主导

2. IDE Agent 派 (Cursor / Windsurf)
   → 多文件理解,人监督

3. CLI Agent 派 (Claude Code / Aider)
   → 终端原生,完全 Agent 化,人审查
```

#### 历史性意义
**软件开发方式正在被重写**。SWE-Bench 测试集从 2023 年的 1% 通过率涨到 2026 年的 80%+。

---

### 八、2026 年正在发生的 3 件大事

1. **Agent Computer Interface(ACI)成型** — 软件**为 AI 重新设计接口**(MCP 是雏形)。
2. **多模态原生化** — 从"语言模型加视觉模块"→ "**统一多模态模型**"。
3. **Long-Horizon Task(长程任务)突破** — Agent 能持续工作**数小时甚至数天**。

---

### 🎯 一句话总结历史脉络

> **2023 让 AI 会"说"**(ChatGPT/RAG)
> **2024 让 AI 会"做"**(Function Call/Agent/Computer Use)
> **2025 让 AI 会"想"**(o1/R1 推理模型)
> **2026 让 AI 会"协作"**(MCP/Agent 生态/Coding Agent)

每一次跨越都是**人机协作边界的扩展**:从问答 → 工具 → 行动 → 思考 → 自主。

---

## 问题三:Claude Code 的 Agent 设计哲学

### ❓ 问题
> Claude Code 的 Agent 设计哲学

### ✅ 回答

#### 一、思想根源:三篇文章读懂它的"道"

| 时间 | 文档 | 核心观点 |
|---|---|---|
| 2024.12 | **Building Effective Agents** | Agent = LLM + Tools + Loop,**别上框架** |
| 2025.04 | **Code Execution with MCP** | 把代码执行当作通用工具 |
| 2025-2026 | **Anthropic Engineering Blog 系列** | Context Engineering 取代 Prompt Engineering |

**一句话定调:** "Build the simplest thing that could possibly work, then make it work."

---

#### 二、六大核心设计原则(反直觉但极其有效)

##### 原则 1:**单 Agent > 多 Agent**(反潮流)

```
其他框架: Planner Agent → Coder Agent → Reviewer Agent → ... (复杂编排)
Claude Code: 一个 Claude + 一堆工具 + while 循环 (够用就好)
```

**为什么?**
- 多 Agent 之间通信全靠"传话",上下文损耗严重
- 协调成本 > 协作收益
- 调试复杂度指数级上升

**让步:** 确实需要分工时,用 **Sub-agent 机制**(Task tool)按需 spawn。

---

##### 原则 2:**CLI > GUI**(回归本源)

| 维度 | 终端 | IDE/浏览器 |
|---|---|---|
| 工具生态 | 几十年积累的 Unix 工具 | 受限于插件 |
| 可组合性 | 管道、重定向天然支持 | 需要 API 适配 |
| 远程能力 | SSH 一行搞定 | 需要专门部署 |
| 开发者习惯 | "Meet developers where they are" | 强迫切换环境 |
| Agent 友好度 | 输入输出都是纯文本 | 需要解析 DOM |

**深层逻辑:** 终端本身就是为"**命令-反馈循环**"设计的,和 Agent 的 ReAct 模式天然契合。

---

##### 原则 3:**Tools 简单 > Framework 复杂**

Claude Code 内置工具就那么十几个,但每一个都遵循 **"小而锐利"** 原则:

```
Read     - 读文件
Write    - 写文件
Edit     - 改文件
Glob     - 找文件
Grep     - 搜内容
Bash     - 执行命令
Task     - 派 sub-agent
WebFetch - 抓网页
```

**核心洞察:** **工具是给模型用的,不是给开发者用的**。模型不需要 fluent API,它只需要清晰的输入输出。

---

##### 原则 4:**Human-in-the-Loop > 完全自主**(关键克制)

```
默认权限模式:
├── 读文件 → 自动放行
├── 写文件 → 询问用户
├── 执行命令 → 询问用户
└── 删除/危险操作 → 强制确认
```

**为什么这是哲学层面的选择?**

> "**信任 = 透明度 × 可逆性**"

- **Plan Mode**:复杂任务先出方案,等用户批准
- **Diff 展示**:每次修改都给 diff
- **Permission Mode**:读/写/执行分级授权
- **Hooks**:用户可拦截任何工具调用

---

##### 原则 5:**文件系统 = 通用内存**(优雅的"作弊")

```
长期记忆 → Markdown 文件 (CLAUDE.md, MEMORY.md)
工作记忆 → 当前 context window
项目知识 → 直接读源码
任务状态 → TodoList 工具
```

**为什么?**

| 方案 | 优点 | 缺点 |
|---|---|---|
| 向量数据库 | 语义检索强 | 需部署、有损、不可读 |
| 图数据库 | 关系强 | 重、慢、复杂 |
| **Markdown 文件** | **零依赖、人可读、git 可追溯、模型原生擅长** | 检索靠 grep |

**核心洞察:**
> **"模型最擅长处理的就是文本,何必给它套一层数据库?"**

---

##### 原则 6:**Context Engineering > Prompt Engineering**

```
Prompt Engineering (旧):
精雕细琢一段话 → 期待模型表现

Context Engineering (新):
设计上下文的"信息架构" → 让模型在每一步都拿到对的信息
```

**Claude Code 的具体体现:**

1. **CLAUDE.md** — 项目级永久上下文
2. **MEMORY.md** — 用户级长期记忆
3. **System Reminders** — 隐式注入的运行时上下文
4. **Tool Results** — 即时的状态反馈
5. **Compaction** — 上下文压缩而非截断

---

#### 三、工程实现:几个"教科书级"的设计选择

##### 1. **Bash 作为终极工具**
**Bash 已经是 50 年累积的"通用工具协议"**。LLM 训练数据里有海量 Bash 案例,直接用就行。

##### 2. **Sub-agent 隔离机制**
```
主 Agent (context: 500K)
  └── Task: "搜索整个代码库的鉴权逻辑"
        └── Sub-agent (独立 context: 200K)
              └── 返回浓缩结论 (5K)
主 Agent 只看到 5K 结论,不被中间过程污染
```

##### 3. **MCP 即插即用**
任何 MCP Server 即接即用。Anthropic 不做"全家桶",而是做"标准接口"。

##### 4. **Hooks 系统**
```
PreToolUse  → 拦截/修改/拒绝
PostToolUse → 校验/记录/触发
Stop        → 收尾/通知
```

##### 5. **Skills 系统**
```
/security-review → 临时加载安全审查规则
/run            → 临时加载启动应用流程
```

**精髓:** **模型保持通用,能力按需注入**。

---

#### 四、对比看清独特性

| 维度 | Claude Code | Cursor | Devin | LangChain Agent |
|---|---|---|---|---|
| 形态 | CLI | IDE | 云端浏览器 | SDK |
| 自主度 | 中(需审批) | 低(辅助) | 高(全自动) | 看你写 |
| 哲学 | Unix + Human-in-loop | IDE Native | 全自动工程师 | 万物皆 Chain |
| 核心抽象 | 工具 + 循环 | 文件 + 补全 | 任务 + 计划 | Chain + Agent + Memory |
| 失败模式 | 用户能接管 | 退回手写 | 难以介入 | 框架黑盒 |

---

#### 五、最反直觉的几个选择(也是最聪明的)

- 🎯 **不做"AI IDE"** — 终端才是工程任务的家
- 🎯 **不做"全自动 Agent"** — 让用户始终在驾驶位
- 🎯 **不做"Agent 编排框架"** — 言行一致
- 🎯 **Markdown 优于一切结构化** — LLM 最擅长 Markdown
- 🎯 **透明度高于"魔法感"** — 信任比惊艳更重要

---

#### 六、一句话总结这套哲学

> **"用最朴素的工程,服务最智能的模型,服务最专业的人。"**

- **最朴素的工程** → CLI、文件系统、Bash、Markdown、while 循环
- **最智能的模型** → 深度耦合 Claude,信任模型决策能力
- **最专业的人** → 给开发者保留控制权和审查权

---

#### 七、对做 AI 应用的启发

1. **先用循环 + 工具跑通**,**再考虑要不要上框架**(80% 情况不需要)
2. **上下文设计 > 模型调优**:把信息架构想清楚,比换模型有用
3. **留出人类介入点**:永远不要假设 AI 100% 正确

---

## 问题四:如何用 MCP + Skills + Hooks 改造团队专属 Agent

### ❓ 问题
> 如何用 MCP + Skills + Hooks 把 Claude Code 改造成你团队的专属 Agent

### ✅ 回答

#### 一、架构思想:三件套的精准分工

> **MCP 让 Agent "够得着外部世界",Skills 让 Agent "懂你团队的活",Hooks 让 Agent "不能乱来"。**

```
┌─────────────────────────────────────────────────┐
│              团队专属 Claude Code               │
├─────────────────────────────────────────────────┤
│  CLAUDE.md  ← 团队大脑(规范、约定、上下文)     │
├─────────────────────────────────────────────────┤
│  Skills     ← 领域能力(评审、发布、设计规范)   │
├─────────────────────────────────────────────────┤
│  MCP        ← 外部连接(Jira / DB / K8s / 内部API)│
├─────────────────────────────────────────────────┤
│  Hooks      ← 安全护栏(强制规范、防止越界)     │
└─────────────────────────────────────────────────┘
                    Claude 模型
```

| 组件 | 解决什么问题 | 类比 |
|---|---|---|
| **CLAUDE.md** | "你应该知道什么" | 员工手册 |
| **Skills** | "你应该会做什么" | 岗位技能培训 |
| **MCP** | "你可以接触什么" | 公司账号权限 |
| **Hooks** | "你不能做什么" | 公司红线制度 |

---

#### 二、MCP 改造:接通你的内部世界

##### 1. 团队最该接入的 5 类 MCP

| 类别 | 用途 | 推荐方案 |
|---|---|---|
| **任务管理** | 自动读 Jira/Linear 工单 | 官方 MCP Server |
| **代码托管** | 操作 GitHub/GitLab | 官方 MCP Server |
| **数据库** | 查询业务数据 | 自研或 PostgreSQL MCP |
| **可观测性** | 查日志/监控/告警 | 自研对接 Grafana/Datadog |
| **内部 API** | 调用公司中台 | **必须自研** |

##### 2. 自己写一个团队 MCP Server(Node.js 版)

```javascript
// team-mcp-server/index.js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "team-internal-api",
  version: "1.0.0",
}, {
  capabilities: { tools: {} }
});

// 注册工具:查询公司用户中心
server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "query_user_by_id",
    description: "查询公司用户中心的用户信息(走内部鉴权)",
    inputSchema: {
      type: "object",
      properties: {
        user_id: { type: "string", description: "用户ID" }
      },
      required: ["user_id"]
    }
  }]
}));

server.setRequestHandler("tools/call", async (req) => {
  if (req.params.name === "query_user_by_id") {
    const result = await fetch(`https://internal-api.company.com/users/${req.params.arguments.user_id}`, {
      headers: { "Authorization": `Bearer ${process.env.INTERNAL_TOKEN}` }
    });
    return { content: [{ type: "text", text: JSON.stringify(await result.json()) }] };
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

##### 3. 团队级 MCP 配置(`.claude/settings.json`)

```json
{
  "mcpServers": {
    "team-api": {
      "command": "node",
      "args": ["./tools/team-mcp-server/index.js"],
      "env": {
        "INTERNAL_TOKEN": "${TEAM_INTERNAL_TOKEN}"
      }
    },
    "company-db": {
      "command": "uvx",
      "args": ["mcp-server-postgres", "postgresql://readonly@db.internal/prod"]
    },
    "jira": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-jira"],
      "env": { "JIRA_TOKEN": "${JIRA_TOKEN}" }
    }
  }
}
```

##### 4. 关键建议
- ✅ 内部 API **只暴露只读 + 关键写操作**
- ✅ 用 **环境变量** 传 Token,**绝不写在 settings.json 里**
- ✅ 给每个团队成员**独立的 MCP 凭证**,便于审计

---

#### 三、Skills 改造:把团队 SOP 变成 AI 能力

##### 1. Skill 的本质

一个 Skill = **一个文件夹**:

```
.claude/skills/code-review-cn/
├── SKILL.md          ← 主入口,描述何时触发、怎么做
├── checklist.md      ← 引用资源
├── templates/        ← 模板文件
│   ├── bug-template.md
│   └── refactor-template.md
└── scripts/          ← 可执行脚本(可选)
    └── auto-check.sh
```

##### 2. 团队最值得做的 7 个 Skill

| Skill 名 | 触发时机 | 价值 |
|---|---|---|
| `code-review` | 写完代码后 | 强制按团队 checklist 自检 |
| `api-design` | 新增/改接口 | 自动套用 RESTful 规范 |
| `db-migration` | 改数据库 | 强制走变更流程 |
| `release-checklist` | 准备上线 | 12 项发布前自检 |
| `incident-handle` | 线上故障 | 故障应急 SOP |
| `commit-style` | 写 commit | 强制 Conventional Commits |
| `bugfix-flow` | 修 bug | 强制根因分析 + 单测 |

##### 3. 实战:写一个"团队代码评审" Skill

**`.claude/skills/team-review/SKILL.md`**:

```markdown
---
name: team-review
description: 团队代码评审专用 Skill,执行 13 项评审清单。在用户完成代码修改后,或主动输入 /team-review 时触发。
---

# 团队代码评审 Skill

## 触发时机
- 用户写完代码,准备提交前
- 用户主动调用 /team-review
- PR 描述中包含 "请评审"

## 执行步骤

### 第一步:读取上下文
1. 读取 @doc/接口文档.md(了解接口约束)
2. 读取 @doc/设计文档.md(了解架构约束)
3. 运行 `git diff --staged` 获取本次变更

### 第二步:执行 13 项评审
按 @checklist.md 中的清单逐项检查,**每项必须给出结论**:
- ✅ 通过
- ⚠️ 警告(可上线但需关注)
- ❌ 阻塞(必须修复)

### 第三步:输出报告
按 @templates/review-report.md 的模板输出。

## 红线规则(发现立即标红)
- 硬编码密钥/Token/密码
- 直接吞掉 error(`if err != nil { return nil }`)
- 跨层调用(api 直连 DB)
- 没有参数校验的接口
- 循环里查数据库

## 重要约定
- **不要顺手重构无关代码**(违反最小变更)
- **不要新增依赖**(除非用户明示)
- 评审完必须输出 @templates/review-report.md 中的"自检结论"段落
```

**`.claude/skills/team-review/checklist.md`**(被 SKILL.md 引用):

```markdown
# 13 项评审清单

1. **功能正确性** - 是否覆盖正常/异常/边界
2. **影响范围** - 是否动了公共结构
3. **代码分层** - api/service/repo 是否越界
4. **最小变更** - 是否只改了必要代码
5. **接口规范** - 路径/返回结构/错误码
6. **参数校验** - 是否信任前端
7. **安全检查** - 硬编码/敏感信息/权限
8. **性能** - 循环查询/N+1
9. **错误处理** - 是否吞错/暴露内部
10. **日志** - 关键路径/敏感信息
11. **依赖** - 是否新增
12. **文档同步** - 接口/设计/README
13. **自检结论** - 影响范围/兼容性/联动/文档
```

##### 4. Skills 的精妙之处

- **按需加载**:不污染主上下文,触发时才加载
- **可引用资源**:`@checklist.md` 这种引用 Claude 会自动展开
- **可执行脚本**:Skill 里可以放 `.sh` `.py`,被 Claude 调用
- **可继承**:个人 Skill 覆盖团队 Skill,团队 Skill 覆盖默认

---

#### 四、Hooks 改造:守住团队红线

##### 1. Hooks 是什么

Hooks 是配置在 `settings.json` 里的**运行时拦截器**,在工具调用前后触发**真实的 shell 脚本**——这是唯一能"**强制**"Claude 行为的机制(Skills 和 CLAUDE.md 只是"建议")。

##### 2. 关键 Hook 时机

| Hook 时机 | 拦截能力 | 典型用途 |
|---|---|---|
| `PreToolUse` | **可阻止** 工具执行 | 防止危险操作 |
| `PostToolUse` | 工具执行完后触发 | 自动校验/格式化 |
| `UserPromptSubmit` | 用户提交前注入上下文 | 自动补充背景 |
| `Stop` | Claude 回答完成 | 自动跑测试/通知 |
| `SessionStart` | 会话启动 | 加载团队配置 |

##### 3. 实战:5 个团队级 Hook

**`.claude/settings.json`**:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{
          "type": "command",
          "command": "node .claude/hooks/check-secrets.js"
        }]
      },
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "node .claude/hooks/block-dangerous.js"
        }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{
          "type": "command",
          "command": "node .claude/hooks/auto-format.js"
        }]
      }
    ],
    "UserPromptSubmit": [{
      "hooks": [{
        "type": "command",
        "command": "node .claude/hooks/inject-context.js"
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "node .claude/hooks/run-tests.js"
      }]
    }]
  }
}
```

##### 4. 拦截器示例:**禁止提交密钥**

**`.claude/hooks/check-secrets.js`**:

```javascript
#!/usr/bin/env node
const input = JSON.parse(require('fs').readFileSync(0, 'utf-8'));
const content = input.tool_input?.content || input.tool_input?.new_string || '';

const patterns = [
  /sk-[a-zA-Z0-9]{20,}/,          // OpenAI key
  /AKIA[A-Z0-9]{16}/,              // AWS key
  /ghp_[a-zA-Z0-9]{36}/,           // GitHub token
  /password\s*=\s*["'][^"']{6,}/i, // 硬编码密码
  /Bearer\s+[a-zA-Z0-9]{30,}/,     // Bearer token
];

for (const p of patterns) {
  if (p.test(content)) {
    console.log(JSON.stringify({
      decision: "block",
      reason: `检测到疑似密钥/Token,违反团队安全红线。请使用环境变量或配置中心。匹配模式:${p}`
    }));
    process.exit(0);
  }
}
process.exit(0);
```

##### 5. 拦截器示例:**禁止操作生产环境**

**`.claude/hooks/block-dangerous.js`**:

```javascript
#!/usr/bin/env node
const input = JSON.parse(require('fs').readFileSync(0, 'utf-8'));
const cmd = input.tool_input?.command || '';

const blacklist = [
  /kubectl.*--context\s*=?\s*prod/,         // 生产 K8s
  /rm\s+-rf\s+\//,                          // 删除根目录
  /DROP\s+(TABLE|DATABASE)/i,               // 删表删库
  /git\s+push.*--force.*main/,              // 强推主分支
  /mysql.*-h.*prod/,                         // 连生产 DB
];

for (const p of blacklist) {
  if (p.test(cmd)) {
    console.log(JSON.stringify({
      decision: "block",
      reason: `命令命中团队禁止规则:${p}\n如需操作,请走变更流程或人工执行。`
    }));
    process.exit(0);
  }
}
```

##### 6. 增强器示例:**自动注入团队上下文**

**`.claude/hooks/inject-context.js`**:

```javascript
#!/usr/bin/env node
const input = JSON.parse(require('fs').readFileSync(0, 'utf-8'));
const prompt = input.prompt || '';

const additions = [];

// 检测到 Jira 单号自动注入
const jiraMatch = prompt.match(/[A-Z]+-\d+/);
if (jiraMatch) {
  additions.push(`📋 检测到 Jira 单号 ${jiraMatch[0]},建议先调用 jira MCP 工具获取需求详情。`);
}

// 检测到 "上线" 字样
if (/上线|发布|deploy/i.test(prompt)) {
  additions.push(`🚨 检测到发布意图,请先执行 /release-checklist Skill 完成自检。`);
}

if (additions.length) {
  console.log(JSON.stringify({
    additionalContext: additions.join('\n')
  }));
}
```

---

#### 五、CLAUDE.md 的三级分层(最容易被忽略的地基)

```
~/.claude/CLAUDE.md            ← 个人偏好(只你自己有)
   ↓
<team-repo>/CLAUDE.md          ← 团队约定(全员共享,进 git)
   ↓
<project>/CLAUDE.md            ← 项目特化(覆盖团队约定)
```

**团队级 CLAUDE.md 应该写什么**(进 git 的那份):

```markdown
# 团队工程规范

## 技术栈
- 后端:Go 1.22 + Gin + GORM + MySQL 8
- 前端:Vue 3 + TypeScript + Vite
- 部署:K8s + ArgoCD

## 代码规范
- 命名:snake_case 文件名,CamelCase 结构体
- 分层:api / service / repo,严禁跨层
- 错误:返回 `errors.Wrap`,不允许吞错

## 接口规范
- 路径:`/api/v{n}/<resource>`
- 返回:`{ code, message, data }`
- 错误码:见 `doc/error-codes.md`

## 必读文档
- 设计文档:`doc/design.md`
- 接口文档:`doc/api.md`
- 部署文档:`doc/deploy.md`

## 红线(违反将被 Hook 拦截)
- 不写硬编码密钥
- 不直连生产数据库
- 不强推主分支
- 不顺手重构无关代码
```

---

#### 六、完整案例:30 分钟搭一个"后端团队 Agent"

##### 目录结构

```
your-backend-repo/
├── CLAUDE.md                          ← 团队规范
├── .claude/
│   ├── settings.json                  ← MCP + Hooks 配置
│   ├── skills/
│   │   ├── api-design/SKILL.md        ← 接口设计 Skill
│   │   ├── db-migration/SKILL.md      ← 数据库变更 Skill
│   │   ├── release/SKILL.md           ← 发布流程 Skill
│   │   └── team-review/               ← 代码评审 Skill
│   │       ├── SKILL.md
│   │       ├── checklist.md
│   │       └── templates/
│   ├── hooks/
│   │   ├── check-secrets.js           ← 密钥拦截
│   │   ├── block-dangerous.js         ← 危险命令拦截
│   │   ├── auto-format.js             ← 自动 gofmt
│   │   ├── inject-context.js          ← 上下文注入
│   │   └── run-tests.js               ← 完成后跑测试
│   └── mcp-servers/
│       └── team-api/                  ← 自研 MCP Server
│           └── index.js
└── doc/
    ├── design.md
    └── api.md
```

##### 实际效果

团队成员只需 `git clone` 然后启动 Claude Code,就会自动获得:

- ✅ **自动加载团队规范**(CLAUDE.md)
- ✅ **接通 Jira / 内部 API / 数据库**(MCP)
- ✅ **触发 /team-review 执行 13 项评审**(Skills)
- ✅ **写代码时自动拦截密钥**(Hooks)
- ✅ **写完自动 gofmt + 跑单测**(Hooks)
- ✅ **提到 Jira 单号自动拉取详情**(Hooks)
- ✅ **试图操作生产环境立即被阻止**(Hooks)

---

#### 七、落地路线图(分四周推进)

##### Week 1:地基
- [ ] 写团队级 `CLAUDE.md`(进 git)
- [ ] 安装 1-2 个官方 MCP(GitHub、Jira)
- [ ] 加 1 个 Hook:密钥拦截

##### Week 2:核心 Skills
- [ ] 写 `team-review` Skill
- [ ] 写 `api-design` Skill
- [ ] 写 `commit-style` Skill

##### Week 3:内部 MCP
- [ ] 写自研 MCP Server 接入 1 个内部 API
- [ ] 接入只读数据库 MCP
- [ ] 接入日志/监控系统

##### Week 4:护栏与自动化
- [ ] 添加 5+ 个 Hooks(危险命令/自动格式化/自动测试/上下文注入)
- [ ] 全员培训 + 收集反馈
- [ ] 形成团队 Agent 改进 Backlog

---

#### 八、避坑清单

| 坑 | 后果 | 对策 |
|---|---|---|
| Skill 写太详细 | 上下文爆炸 | 主 SKILL.md 精简,细节放引用文件 |
| Hook 写成同步阻塞 | 体验卡顿 | 重操作改异步,Hook 只做轻校验 |
| MCP Server 暴露写权限 | 安全风险 | 默认只读,写操作走审批 |
| settings.json 提交 Token | 泄露 | 用 `${ENV_VAR}` + `.env.local` |
| CLAUDE.md 太啰嗦 | Claude 不读 | 保持<500 行,分模块拆 |
| Hooks 用错 matcher | 不触发 | matcher 是正则,用 `Write\|Edit` 而非 `Write,Edit` |
| 团队不培训 | 没人用 | 写 onboarding 文档 + Demo 视频 |

---

#### 九、终极心法

> **MCP 是"长出手脚",Skills 是"学会规矩",Hooks 是"戴上紧箍咒",CLAUDE.md 是"心法口诀"。**
>
> **四者合一,Claude Code 就从"通用 AI 助手"变成"你团队的资深工程师"。**

而且这个"资深工程师"有几个超能力:
- ⏰ 7×24 在线
- 📚 永远记得最新规范
- 🚫 不会违反红线
- 👥 给团队每个人提供**一致的标准**

---

## 附录:核心概念速查表

### 关键术语对照

| 英文 | 中文 | 解释 |
|---|---|---|
| RAG | 检索增强生成 | 检索外部知识后再生成回答 |
| MCP | 模型上下文协议 | AI 工具集成标准 |
| Agent | 智能体 | LLM + Tools + Loop |
| Function Calling | 函数调用 | LLM 结构化输出工具调用 |
| Chain-of-Thought | 思维链 | 模型先思考再回答 |
| Test-Time Compute | 推理时计算 | 用推理算力换性能 |
| Context Engineering | 上下文工程 | 设计信息架构而非雕琢 prompt |
| Computer Use | 电脑使用 | AI 操作真实软件界面 |
| Sub-agent | 子智能体 | 派出隔离上下文的子 Agent |
| ACI | 智能体计算机接口 | 为 AI 设计的软件接口 |

### Claude Code 关键文件

| 路径 | 作用 |
|---|---|
| `~/.claude/CLAUDE.md` | 全局个人偏好 |
| `~/.claude/settings.json` | 全局配置(MCP/Hooks) |
| `<project>/CLAUDE.md` | 项目规范 |
| `<project>/.claude/settings.json` | 项目配置 |
| `<project>/.claude/skills/*/SKILL.md` | 项目 Skills |
| `<project>/.claude/hooks/*.js` | 项目 Hooks |

### 重要外部资源

| 资源 | 链接关键词 |
|---|---|
| Anthropic 官方博客 | "Building Effective Agents" |
| MCP 协议官网 | modelcontextprotocol.io |
| Claude Code 文档 | docs.anthropic.com/claude-code |
| MCP Server 市场 | 搜索 "MCP Server registry" |

---

**📌 文档维护说明**

- 本文档由 4 次问答整理而成,反映 2026 年 6 月时点的行业认知
- AI 行业演进极快,半年后部分细节可能需要更新
- 建议每季度回顾一次,补充新进展

---

*文档结束 — 持续学习,持续构建。*
