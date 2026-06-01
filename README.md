# Skills 技能仓库

Claude Code 技能集，包含以下独立模块：

| 技能 | 目录 | 功能 |
|---|---|---|
| project-learner | project-learner/ | 全自主项目学习与架构分析 |
| go-project-init | go-project-init/ | 初始化 Go 项目结构 |
| architect-review | architect-review/ | 架构评审与建议 |
| spec-checker | spec-checker/ | 代码规范检查 |

---

## 安装方法

### 全局安装（所有项目可用）

将技能目录复制到 Claude Code 全局技能目录：

```bash
# 克隆或复制 skills 仓库到全局技能目录
cp -r G:\go\src\skills ~/.claude/skills/
```

### 项目私有安装（仅当前项目可用）

在项目根目录下创建 `.claude/skills/` 目录，将技能放入：

```bash
mkdir -p .claude/skills
cp -r G:\go\src\skills/* .claude/skills/
```

---

## 使用方法

### 自动触发

Claude Code 会根据用户输入的关键词自动匹配对应技能：

| 用户输入 | 触发技能 | 说明 |
|---|---|---|
| `/project-learner` | project-learner | 全自主项目学习与架构分析 |
| `/go-project-init` | go-project-init | 初始化新 Go 项目结构 |
| `/architect-review` | architect-review | 架构评审与建议 |
| `/spec-checker` | spec-checker | 代码规范检查 |

### 手动调用

在对话中直接描述需求，Claude Code 会自动识别并调用相应技能。例如：

- "分析一下当前项目架构" → 触发 project-learner
- "初始化一个新 Go 项目" → 触发 go-project-init
- "评审一下这个接口设计" → 触发 architect-review
- "检查代码是否符合规范" → 触发 spec-checker

---

## 目录结构

```
skills/
├── README.md              # 本文档
├── project-learner/       # 项目学习技能
├── go-project-init/       # Go 项目初始化技能
├── architect-review/      # 架构评审技能
└── spec-checker/          # 规范检查技能
```