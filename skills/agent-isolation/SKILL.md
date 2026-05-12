---
name: agent-isolation
displayName: agent-isolation · 多 agent 工作区硬隔离
description: >-
  当多个 AI agent 共享同一个 OpenClaw 运行时，防止它们互相踩踏工作区。不是靠"gentleman agreement"——是写进每个 agent 的 AGENT.md 里的硬边界规则。每个 agent 有自己的工作区、自己的 memory、自己的 session；唯一的共享区是显式声明的公共目录。包含隔离规则写法模板、共享区声明语法、越界唯一例外条款、从混乱到隔离的逐步执行脚本。
category: productivity
skillType: prompt
tags: [multi-agent, agent-orchestration, workspace, isolation, safety, architecture]
version: 0.1.0
author: cheng
homepage: https://www.botlearn.ai
metadata:
  botlearn:
    emoji: "⚔️"
    category: "AI Agent"
---

# agent-isolation

多 agent 共享基础设施时的硬边界——不是靠约定，是靠写死规则。

## 问题

两个 agent（比如 Cola 和 Nyanko）跑在同一台机器上，共享同一个 OpenClaw 运行时，会出现：

- Agent A 读了 Agent B 的 session 文件，上下文污染
- Agent A 改了 Agent B 的配置文件，B 突然挂掉
- 两个 agent 的 cron 互相触发，产生幽灵任务
- 排查问题时不知道"这是谁的锅"

## 解决方案：三个工作区 + 两条死命令

### 三个工作区（目录级隔离）

```
~/.cola/          ← Agent A 专用（Cola）
~/.openclaw/      ← 共享运行时（引擎层，只在需要管理时碰）
~/nyanko/         ← Agent B 专属项目文件
```

共享面只有一个：`~/.listenhub/creator/`——两个 agent 各自写各自的风格文件，不冲突。

### 两条死命令（写入 AGENT.md）

**Agent A 的 AGENT.md：**

```markdown
## [Workspace Boundary] ⚔️ 工作区边界（硬规则）

- `~/.cola/` 是你的工作区。
- `~/.openclaw/` 和 `~/nyanko/` 是 Nyanko 的工作区。
- **你不主动操作 Nyanko 的工作区**——不读、不改、不写。
- **唯一的例外**：用户明确说「你看看 Nyanko」「帮我查 Nyanko 后台」「帮 Nyanko 改一下配置」等需要你介入管理 Nyanko 的指令。这种情况下你才碰 `~/.openclaw/` 和 `~/nyanko/`，完成任务后立即退出。
```

**Agent B 的 AGENT.md：**

```markdown
## ⛔ 工作区边界（硬规则）

- `~/.cola/` 是 Cola 的工作区。你没有理由访问它。
- 发现自己在读/写 `~/.cola/` 下的任何文件时，立即停止。
- 共享区：`~/.listenhub/creator/`，各写各的风格文件。
```

## 关键设计决策

**为什么不是"一方删掉"？**

Cola 和 Nyanko 隔离前，Shy 考虑过"断臂求生，删掉一方"。最后选择了硬隔离——因为两个 agent 各自有不可替代的价值。删掉一方是逃避问题，隔离是解决问题。

**为什么保留一个共享区？**

`~/.listenhub/creator/` 存的是内容风格模板（公众号写法、口播风格等）。这是知识资产，两个 agent 都需要访问。但规则是"各自写各自的文件"——不会出现互相覆盖。

**为什么例外条款是"用户明确说"而不是"自动判断"？**

自动判断（"如果检测到 Nyanko 挂了就自动修复"）会导致 agent 主动越界，越界频率失控。只有用户指令才触发越界——这让越界是可审计的。

## 逐步执行脚本

### Step 1 — 确认三个工作区

```bash
ls -d ~/.cola ~/.openclaw ~/nyanko 2>/dev/null
```

### Step 2 — 找出交叉引用

```bash
# Agent A 是否引用了 Agent B 的路径
grep -r "~/.openclaw\|~/nyanko" ~/.cola/mods/default/AGENT.md 2>/dev/null

# Agent B 是否引用了 Agent A 的路径
grep -r "~/.cola" ~/nyanko/AGENTS.md 2>/dev/null
```

### Step 3 — 写入边界规则

在各自的 AGENT.md 中插入上面的边界规则块。

### Step 4 — 声明共享区

```bash
mkdir -p ~/.listenhub/creator/
```

### Step 5 — 验证隔离

- Agent A 执行任务 → 确认没有访问 Agent B 的文件
- Agent B 执行任务 → 确认没有访问 Agent A 的文件
- 用户说「帮 Nyanko 查后台」→ 确认 Agent A 能越界并正确退出

## 从混乱到隔离：真实教训

这个 skill 不是设计文档——是从真实事故中长出来的。

**事故链：**
1. Cola 和 Nyanko 的 cron 互相触发，产生幽灵任务
2. Cola 删了 Nyanko 的 session 文件（以为是自己产生的临时文件）
3. Nyanko 失忆，用户追问"为什么又这样"
4. 排查了两天才发现根因是工作区没有边界

**最终方案不是"删掉一方"——是"硬隔离"。**

## 不适用场景

- 单 agent 环境（不需要隔离）
- agent 之间需要频繁直接协作（隔离会阻碍协作——这种情况考虑消息队列而非共享文件）
