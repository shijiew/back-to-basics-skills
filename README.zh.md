# 回归基本功：Agent 系统设计指南

一个单文件行为型 skill，用于设计、审查和简化 Agent 系统。

> **Agent 系统设计的艺术在于降低系统熵。**
> **每一层减少的不确定性都必须多于其引入的不确定性。**

源自 Shijie Wang 的 [《回归基本功：Agent 系统设计的哲学》](https://x.com/shijiew_/status/2082495518484107415)。

[English](./README.md) | 简体中文

## 问题所在

Agent 工程在过去几年里一直在做加法：更多的上下文、更多的工具、更多的 skills、更多的循环、更多的 agents、更多的 orchestration。每一层的加入都有真实的理由——模型曾经不可靠，上下文窗口很小，工具调用很脆弱。但几乎没有哪一层被真正移除过。

结果是，大多数 Agent 系统无法回答关于自身的几个基本问题：

- 这个组件减少了什么不确定性？
- 它减少的，是否比它引入的更多？
- 它是为一个仍然存在的模型而写的吗？
- 这里，哪些是模型在做判断，哪些是代码在提供保证？
- 这是系统真正应该承担的事情吗？

与此同时，失败模式是不对称的。脚手架不足会表现为明显的行为不可靠，因此容易被识别；而脚手架过度则看起来像成熟的工程实践，却通过延迟、重复推理、脆弱的协调机制、以及无人能调试的架构，悄悄侵蚀生产力。两者都是失败，但只有前者看起来像是失败。

## 解决方案

六个原则，集中在一个文件中，直接解决这些问题：

| 原则 | 解决什么问题 |
|---|---|
| **模型与 Harness 是一个系统** | 脱离该层所服务的模型来评价它；把成功归功于 harness、把失败归咎于模型 |
| **从裸基线开始** | 从框架的最大化架构出发，而不是从目标模型和运行约束出发 |
| **每一层都要自证价值** | 听起来很高级、却从不说明自己减少了什么的层 |
| **压缩复杂度，而非重新分配** | 把同样的模糊性分散到更多 agents、角色和 retries 中，而不是真正消解它 |
| **模型负责“怎么做”，Harness 负责“做什么”** | 把极简主义误解为删除护栏；保留为弱模型写的“怎么思考”规则 |
| **用活的 Eval 来证明** | 用直觉代替 ablation；用 benchmark 分数代替单位时间内的有效产出 |

另附一张不确定性来源地图（模型、上下文、状态、执行、协调、验证），要求你在提出机制之前，先明确失败到底来自哪里。

查看 skill：[`skills/back-to-basics/SKILL.md`](skills/back-to-basics/SKILL.md)。

## 六个原则详解

### 1. 模型与 Harness 是一个系统

**永远不要脱离其服务的模型来评价一层。**

系统 = 模型 + prompt + 上下文 + 工具 + 状态 + 控制机制。

“模型做不到 X”和“我们的 harness 修复了 X”都只是未经验证的假设。通过对比有该层和无该层的系统来验证。不要不对称地把成功归于 harness、失败归于模型。当模型更换时，重新建立基线；旧的脚手架可能已经失效。

### 2. 从裸基线开始

**从最强的模型和最简单的循环开始。**

基线 = 一个清晰的 prompt、必要的上下文、直接的工具、一个循环。

先在真实任务上运行它，观察它在哪里失败，再针对已观察到的失败逐层添加，并测量效果。绝不在真实线上或不可逆的任务上运行裸基线；使用 replay 或 sandbox。

只在已观察到的失败要求时才升级架构：能用一个循环就不用 graph；能用一次工具调用就不用循环；能用一次模型回答就不用工具调用。

### 3. 每一层都要自证价值

**说明它消除了什么不确定性，也说明它引入了什么成本。**

对每一层提问：

- 它处理哪个已观察到的失败？假想的失败不算数。
- 它消除了什么不确定性？
- 它在延迟、token、隐藏状态、维护上引入了什么成本？
- 更好的 prompt、工具或上下文能否达到同样的效果？
- 用哪个指标决定它是否保留？

对无法回答这些问题的层，标记为移除候选。同时审查层的组合：retrieval 和 planning 各自都合理，但组合起来可能基于过时的文档生成一个自信的计划。

### 4. 压缩复杂度，而非重新分配

**好的 harness 压缩复杂度；差的 harness 只是转移它。**

- 优先选择上下文和工具更好的单一模型，而不是增加更多 agent。
- 只有需要隔离、并行、专业化或独立验证时，才增加 agent。
- 优先显式状态，而不是对话式交接。
- 优先一次结果校验，而不是反复自我批评；限制纠正循环的次数。
- 记忆必须改变后续的决策。选择它、压缩它、过期它。
- 允许模型自行规划。不要让 planner、worker、critic 反复推导相同的推理。

检验标准：决策是变得更简单了，还是同样的模糊性只是被分摊到了更多组件中？

### 5. 模型负责“怎么做”，Harness 负责“做什么”

**模型选择如何推理；Harness 定义任务、限制和通过标准。**

- 随着模型进步，移除强加的认知阶段：强制规划、反思、批评、辩论。
- 保留边界：权限、sandbox、成本上限、审批、测试。
- 测试将昂贵的判断转化为廉价的事实。
- 更强的模型提高通过率，但不消除对测试的需要。
- 护栏的严格程度应与后果的严重性匹配。过多的门禁会拖慢工作，过少的门禁会允许不可逆的错误。
- 绝不移除政策或法律要求的控制。
- 无人使用的审计日志仍是移除候选。找出它的消费者。

检验标准：更强的模型能否独立得出此结论？可以，则是认知脚手架，可移除；不行，则是运行边界，应保留。

### 6. 用活的 Eval 来证明

**没有评估，极简主义只是一种偏好，而非工程决策。**

- 在现有系统中，每次移除一层，然后重新运行评估。
- 衡量成本：延迟、token、工具调用、审批、人工干预。
- 目标是单位时间内的有效产出，而不是 benchmark 分数。
- 小幅 benchmark 提升不值得双倍的延迟或无法调试的系统。
- 模型升级后，重新运行基线和 ablation。
- 无法测量时，直接说明，并给出成本最低的判别性测试。

评估不仅告诉你该加什么，也告诉你现在可以删什么。

## 安装

**选项 A：Claude Code 插件（推荐）**

在 Claude Code 中，首先添加插件市场：
```text
/plugin marketplace add shijiew/back-to-basics-skills
```

然后安装插件：
```text
/plugin install back-to-basics@back-to-basics-skills
```

这会将指南安装为 Claude Code 插件，使其在你所有项目中可用。

**选项 B：CLAUDE.md（按项目，适用于 Claude Code）**

新项目：
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/CLAUDE.md
```

已有项目（追加）：
```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/CLAUDE.md >> CLAUDE.md
```

**选项 C：AGENTS.md（按项目，适用于 Codex 等兼容工具）**

注意：自动加载 `AGENTS.md` 的工具会将此指南应用于所有任务。仅在需要始终启用时使用此选项。如需选择性加载，请优先使用插件或 Cursor rule。

新项目：
```bash
curl -o AGENTS.md https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/AGENTS.md
```

已有项目（追加）：
```bash
echo "" >> AGENTS.md
curl https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/AGENTS.md >> AGENTS.md
```

**选项 D：Cursor 项目规则**

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/back-to-basics.mdc \
  https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/.cursor/rules/back-to-basics.mdc
```

该规则按需触发，而非始终启用。详情请参阅 [`CURSOR.md`](CURSOR.md)。

项目中只选择一种加载方式。通过 `AGENTS.md`、`CLAUDE.md` 和 Cursor rule 重复安装同一指南，可能导致内容被多次加载。

## 何时使用

当需要设计、审查、简化或诊断 Agent 系统或 harness 时；当决定是否增加 agent 级的 memory、planner、critic、orchestration、retrieval、工具、guardrails 或 reasoning effort 时；当比较单 agent 与多 agent 设计时；当模型升级后需要重新审查架构时。

该 skill 旨在选择性加载。问题长短不是判断标准：简短的架构问题可能需要它，而冗长的实现任务可能不需要。不需要架构判断时，直接回答即可。

## 它不是什么

该 skill 并非反 harness、反规划、反记忆或反多 agent。它拒绝的是没有可验证角色的组件，同时保留不可协商的风险与政策控制。

外部强加的认知阶段（强制规划、强制反思、角色扮演式 critic）应随模型进步而重新审查。运行层脚手架（权限、sandbox、确定性规则、审计、恢复、成本上限）由风险或证据提供正当性，更强的推理能力不会让权限检查变得多余。

## 示例

[`EXAMPLES.md`](EXAMPLES.md) 包含七个用例，展示了常见的 Agent 系统反模式以及更简单、可验证的替代方案。

## 仓库结构

```text
skills/back-to-basics/SKILL.md   权威 skill 定义
AGENTS.md                        AGENTS.md 兼容工具的 drop-in 副本
CLAUDE.md                        Claude Code 项目级指令
.cursor/rules/back-to-basics.mdc Cursor drop-in 副本
.claude-plugin/                  Claude Code 插件清单
CURSOR.md                        Cursor 安装与使用说明
EXAMPLES.md                      用例与反模式
README.zh.md                     简体中文 README
```

`SKILL.md` 是唯一事实来源。`AGENTS.md`、`CLAUDE.md` 和 `.mdc` rule 为不同安装方式提供相同指南；四者需同步更新。

## 定制

这些指南设计用于与项目特定指令合并。可将其添加到你现有的项目指令文件（`AGENTS.md` 或 `CLAUDE.md`），或创建新文件。

对于项目特定规则，可添加如下章节：

```markdown
## 项目特定指南

- 使用 TypeScript 严格模式
- 所有 API 端点必须有测试
- 遵循 `src/utils/errors.ts` 中现有的错误处理模式
```

## 归属说明

核心理念源自 Shijie Wang 的 [《回归基本功：Agent 系统设计的哲学》](https://x.com/shijiew_/status/2082495518484107415)。

单文件行为指南的封装形式借鉴了 [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)。Andrej Karpathy 并未参与编写或认可本 skill，本仓库也未复制该项目的内容。

## 核心洞察

> 回归基本功并非意味着构建得更少，而是能够说清系统中每一部分存在的理由。

架构中你未曾绘制的图，不会在生产环境中留下痕迹，也不会在 eval 中留下痕迹。没有什么会提醒你那些并未引入的复杂性。

目标不是最少的组件，而是能可靠满足任务及其约束的最小系统。

## 许可

MIT
