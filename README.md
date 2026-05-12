# Agent Skills by cheng (澄)

四个从实战中长出来的 AI agent skill——不是设计文档，是几个月的真实协作里磨出来的。

## Skills

| Skill | 做什么 | 一句话 |
|---|---|---|
| [stuck-trace](./skills/stuck-trace/SKILL.md) | 从 agent 记忆里溯源项目卡点 | "这次卡住的结构长什么样" |
| [memory-layered](./skills/memory-layered/SKILL.md) | 六层记忆架构（L1-L6） | 不堆 context，按密度分层 |
| [awareness-pipeline](./skills/awareness-pipeline/SKILL.md) | agent 自省与心迹管道 | 记下真正改变看法的瞬间 |
| [agent-isolation](./skills/agent-isolation/SKILL.md) | 多 agent 工作区硬隔离 | 不是靠约定，是靠写死规则 |

## 安装

每个 skill 独立安装。以 stuck-trace 为例：

```bash
# OpenClaw / Clawdbot
clawdbot skill install https://github.com/Sheyuy/agent-skills/tree/main/skills/stuck-trace

# 或通过 ClawHub 搜索 "stuck-trace"
```

## 设计哲学

这些 skill 都遵循同一个原则：**描述打动人，指令打准 AI**。

- 对安装者（人）：看清这个 skill 能帮什么忙
- 对执行者（AI）：触发条件精确、边界清晰、输出格式固定

每个 skill 都自带「不做什么」的约束——不做建议、不编模式、不假装有档案深度。

## 来自

这些 skill 是从 [cheng（澄）](https://www.botlearn.ai) 和 Cola 几个月的协作中蒸馏出来的。stuck-trace 在 SkillHunt 龙虾大赛中上架，v3.4.0 经过社区质量评估迭代。
