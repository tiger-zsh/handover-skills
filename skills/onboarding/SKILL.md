---
name: onboarding
description: 仅当用户明确要求创建、更新或审查项目 onboarding（新人入门）文档，或输入 /onboarding 时使用。不要在普通的 "项目怎么启动 / 怎么理解" 问答中触发——那类一次性问答应直接作答，本 skill 的任务是生成可持久化的 docs/onboarding.md 文档。
---

# onboarding

为目标项目生成或更新 `docs/onboarding.md`，让新人能在第一天到第一周内完成本地启动、跑通测试、找到核心入口、知道第一个低风险练手任务。

## 触发与不触发

只在用户**要求一份持久化文档**时触发。判断方法：用户要的是"在 `docs/` 下落一份文件"，还是"现在就告诉我答案"？

触发的典型说法：
- `/onboarding`
- "生成 / 写 / 更新 onboarding 文档"
- "帮我整理新人入门文档"
- "我要交接项目，先准备 onboarding"

**不要**触发的场景（直接回答或交给其它 skill）：
- "这个项目怎么启动？" → 直接回答。
- "我该看哪个文件入手？" → 直接回答。
- 用户报告启动报错 → 这是 bug，归 `sop` skill。

为什么这条边界重要：onboarding 文档是给"未来的新人"看的，需要确认、需要落盘；而日常问答只需要当下的答案。如果在每次问答里都生成完整文档，文档会被反复重写、和用户当前的临时问题混在一起，反而失去价值。

## 工作流程

### 1. 确认项目根

```bash
git rev-parse --show-toplevel 2>/dev/null || pwd
```

所有产出写到 `<项目根>/docs/onboarding.md`。

如果该文件已存在，进入**更新模式**——只改差异，保留用户已确认的内容。详细约束见 [references/MAINTENANCE.md](references/MAINTENANCE.md)。

### 2. 自动扫描生成草稿

读以下文件来推断信息（按项目实际存在的挑选）：

- 包管理：`package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` / `Gemfile` / `pom.xml`
- 启动脚本：`Makefile` / `justfile` / `Taskfile.yml` / `scripts/`
- 容器化：`Dockerfile` / `docker-compose*.yml`
- 项目说明：`README*.md` / `CONTRIBUTING.md` / `CLAUDE.md`
- 环境变量：`.env.example` / `.env.sample`

**优先复用项目的 `CLAUDE.md`**：如果项目里有 `CLAUDE.md`，先完整读一遍。CLAUDE.md 通常是项目熟悉者写给 AI 的协作指南，往往已经覆盖了 onboarding 该写的大部分内容——架构概览、启动命令、已知坑、关键文件。这种情况下：

- **复用**而不是重写——把 CLAUDE.md 已有的事实直接搬过来（标 `[代码确认]`，加上 "来自 CLAUDE.md" 说明），不另起炉灶。
- 把 §9 "接下来该看什么" 明确指回 `CLAUDE.md`，避免 onboarding.md 和 CLAUDE.md 长期 drift。
- onboarding 的差异化定位：按**新人最先想知道什么**的顺序重新组织（启动 → 测试 → 入口 → 练手任务）；CLAUDE.md 通常按 "AI 协作上下文" 的顺序组织，两者排版不同但事实应一致。

为什么这条重要：在 CLAUDE.md 写得好的项目里，沿用这条会让 `[待确认]` 项压到极少。反之每次都自己重新推断，既容易出错又和 CLAUDE.md drift。

然后基于这些信息填写 [assets/ONBOARDING.md](assets/ONBOARDING.md) 模板的各小节。

### 3. 每条信息都要有来源标记

这是本 skill 的核心约束。模板里每条信息后面要带标记：

- `[代码确认]` —— 从代码 / 配置文件直接读到
- `[用户确认]` —— 在交互中用户已确认
- `[待确认]` —— 你推断出来的，但代码里没直接体现

为什么这个标记很重要：onboarding 文档会被新人当作"权威指引"去执行。一旦你把推断写成事实（"启动命令是 npm run dev"，但其实代码里写的是 `bun dev`），新人会按错的命令操作并浪费时间。带标记可以让新人和复审者一眼看出"哪些是验证过的事实、哪些需要先核对"。

凡是没在文件里直接看到的，必须标 `[待确认]`。这条没有例外——宁可让文档显得"不完整"，也不能让它"看起来很完整但其实在瞎编"。

### 4. 逐节确认（除非用户说跳过）

把草稿展示给用户，逐节确认关键信息：

- 本地启动命令是否正确？
- 测试命令是否正确？
- "3-5 个核心入口"是否真的是新人最先该看的？
- "第一个低风险练手任务"是否合适？

用户确认的小节，把标记从 `[待确认]` 升级为 `[用户确认]`。

如果用户明确说"草稿够用，不要逐节问"，跳过这一步即可，但**所有 `[待确认]` 标记必须保留**——告诉用户"我没有逐节确认，所有 `[待确认]` 项你或下一个接手人需要自己核实"。

### 5. 写入 docs/onboarding.md

如果 `docs/` 不存在，先创建。然后写入。

### 6. 提示用户考虑加 CLAUDE.md 引用（不要自动改）

完成后输出一段建议，比如：

```markdown
建议在项目 CLAUDE.md 中加入：

- 新人入口文档：`docs/onboarding.md`
```

不要自动改 CLAUDE.md。CLAUDE.md 是项目的协作规则文件，自动改有侵入性；不同项目对入口引用的位置和格式偏好不同，应由用户决定。只有用户明确说"也帮我加到 CLAUDE.md"时再改。

## 验收标准

一次成功的 onboarding 文档应该满足：

1. 在该项目中按文档命令能完成本地启动。
2. 文档能让新人跑通主要测试。
3. 文档列出 3-5 个核心入口文件或模块（不是 10+ 个——多了等于没指引）。
4. 文档列出**一个**具体的低风险练手任务（不是 "你可以做 X 或 Y 或 Z"，要给一个明确建议）。
5. 未确认的内容明确标 `[待确认]`，没有把推断写成事实。
6. 文档总长建议控制在 200 行以内。超过说明在抢 architecture-map / runbook 的职责，需要拆分。

## 边界：本 skill 不做什么

- **不**写故障 / 错误排查 → 交给 `sop` skill
- **不**写发布 / 回滚 / 备份 / 告警 → 交给 `runbook` skill（计划中）
- **不**写完整架构图 / 模块边界 / 数据流 → 交给 `architecture-map` skill（计划中）
- **不**自动修改项目 `CLAUDE.md`

如果用户的需求超出 onboarding 范围，明确告诉他"这部分归 X skill / 等对应 skill 上线后处理"，不要把所有内容都塞进 onboarding。

## 参考

- [references/MAINTENANCE.md](references/MAINTENANCE.md) —— 更新模式的行为约束、何时建议更新
- [assets/ONBOARDING.md](assets/ONBOARDING.md) —— 生成 docs/onboarding.md 用的模板
