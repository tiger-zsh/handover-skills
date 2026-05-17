# handover-skills

一组**显式触发**的项目交接文档生成 skill。配合事件驱动的 [`sop`](https://github.com/tiger-zsh/sop) 仓库一起，覆盖"新人接手 / 离职交接"的完整链路。

## 设计原则

这个仓库里的 skill 都是 **B 类（显式触发型）**：

- **只在用户明确请求时触发**（例如 `/onboarding`、"生成 runbook"、"写一份架构文档"）。
- **不**承担"回答相关问题"的职责——`项目怎么启动？` `线上挂了怎么办？` 这类普通问答归正常对话或 `sop` 处理，本仓库的 skill 不会被触发。
- **产出固定**：每个 skill 对应一份目标项目下的 `docs/*.md` 文档。

这和 [`sop`](https://github.com/tiger-zsh/sop) 仓库形成互补：

| 仓库 | 形态 | 触发 | 产出 |
|---|---|---|---|
| `sop` | A 类：事件驱动 | 异常名、错误信息、症状关键词、`/sop` | `docs/sop/*.md` 故障剧本 |
| `handover-skills`（本仓库） | B 类：显式触发 | 用户明确要求"生成/写/更新某类文档"、`/onboarding` 等 | `docs/onboarding.md` 等交接文档 |

两个仓库职责清晰，可以独立演进，不混在一起。

## Skill 路线

| Skill | 状态 | 用途 | 产出 |
|---|---|---|---|
| `onboarding` | ✅ 已实现 | 新人第一天到第一周快速进入项目 | `docs/onboarding.md` |
| `runbook` | 🚧 阶段 2 | 生产运维、发布、回滚、告警 | `docs/runbook.md` |
| `architecture-map` | 🚧 阶段 2 | 系统结构、模块边界、数据流 | `docs/architecture.md` |
| `handover-audit` | ⏸ 阶段 3 评估 | 交接前缺口审计（可能只做 checklist 模板） | `docs/handover-checklist.md` |

落地节奏故意分阶段：

- **阶段 1（当前）**：只做 `onboarding`，验证"显式触发型 skill"形态是否好用，跑 2-3 个真实项目。
- **阶段 2**：加 `runbook` 和 `architecture-map`。后两个比 onboarding 风险高（涉及生产 / 架构意图），需要强制"来源标记"机制（代码确认 / 配置确认 / 用户确认 / 待确认）。
- **阶段 3**：根据前 3 个 skill 在真实项目里的使用情况，决定 `handover-audit` 是独立 skill 还是只保留为共享 checklist 模板。

## 文档放在哪里

所有 skill 生成的文档都写到**目标项目**的 `docs/` 下，不写在本仓库内。Skill 通过当前工作目录或 `git rev-parse --show-toplevel` 识别项目根。

## 和 CLAUDE.md 的关系

生成文档后，skill 会**提示**用户在项目级 `CLAUDE.md` 加一段引用，但**不**自动改 `CLAUDE.md`——是否引用、放在哪里，由用户决定。

## 新人接手推荐流程

```text
1. /onboarding         先知道项目怎么跑、怎么测、看什么
2. /architecture       理解模块边界、数据流、核心依赖
3. /runbook            掌握发布、回滚、备份、告警
4. 遇到具体错误时       让 sop skill 自动触发查剧本
```

## 安装

参考根目录或各 skill 目录下的 `SKILL.md`。

## 相关

- [`sop`](https://github.com/tiger-zsh/sop) —— 事件驱动的故障剧本 skill
