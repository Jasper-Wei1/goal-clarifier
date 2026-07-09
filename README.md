# goal-clarifier

把模糊想法澄清成可交给 Agent 执行的目标指令。

适用于 Codex、Claude Code，以及其他支持 `SKILL.md` / system prompt 的 Agent 工作流。

很多时候，模型已经足够强，真正的瓶颈是人还没有把目标说清楚：

- 想做什么说不清
- 什么算完成说不清
- 什么不做说不清
- 哪些地方让 Agent 自主判断说不清
- 哪些情况必须先问用户说不清

`goal-clarifier` 的作用不是帮你把一句话润色得更像 prompt，而是通过交互式澄清，把一团模糊想法压成一条可执行、可验收、有边界的 Agent goal。

作者主页：[Jasper Wei](https://x.com/Jasper_Wei1)
---

## 这个 Skill 解决什么问题

它专门处理这类输入：

```text
我想做一个内容系统，但不知道从哪里开始。
我想让 Agent 帮我优化这个项目，但我说不清楚优化什么。
我有个想法，想先让 Agent 帮我拆清楚。
我知道自己不满意，但不知道最终要什么结果。
我想把这个需求整理成适合 Codex / Claude Code 执行的 goal。
```

它最终会帮你得到：

- 明确目标：到底要 Agent 完成什么
- 验收标准：什么结果算通过，什么结果算失败
- 执行边界：做什么，不做什么，做到哪里停
- 非目标：本轮明确不处理的事项
- 未知处理：遇到小未知怎么保守判断，遇到大未知什么时候回问
- 可复制 goal：可以直接交给 Codex / Claude Code / 其他 Agent 执行

---

## 核心方法

`goal-clarifier` 不是次数驱动，而是标准驱动。

它不会因为“问了几轮”就草草结束。只有当目标、验收标准和执行边界都足够清楚时，才输出最终 goal。

如果用户的请求已经足够清楚，它也不会为了“显得在澄清”而强行追问。它会先明确判断：

```text
这条 goal 已经具备目标、交付物和边界，可以直接执行。
```

然后跳过完整澄清流程，直接进入执行或结构化输出。少问问题不代表 skill 没有发挥作用，而是说明这条请求已经通过了 ready-goal 检查。

如果用户能说出一个模糊目标，它会进入目标审计：

```text
可指物：完成后能指着什么说“就是这个”？
可否证：什么情况算没做到？
有完成态：Agent 做到哪里应该停？
有使用场景：结果拿来做什么？
```

如果用户连目标都说不出来，它不会继续逼问“你到底要什么”，而是先做未知发现：

```text
盲点发现：你可能缺的是目标、标准、路径、边界、偏好，还是领域地图？
候选项反应：给出几个可能方向，让你判断哪个更接近、哪个明确不要。
参考物校准：给出几种交付物形态，让你看见“什么算好”。
一问一答访谈：只问会改变目标结构的关键问题。
```

---

## 工作流程

```text
Phase -1：起点定位
Phase 0：未知发现
Phase 1：形成目标原话
Phase 2：目标澄清
Phase 3：验收标准澄清
Phase 4：执行边界澄清
Phase 4.5：goal 草案压力测试
Phase 5：结构化 goal 输出
Phase 6：用户确认回测
```

最终输出格式大致是：

```markdown
# Agent Goal

## Goal
{一句话目标}

## Context
{背景、材料、使用场景}

## Deliverable
{具体交付物}

## Acceptance Criteria
- [ ] {可检查标准 1}
- [ ] {可检查标准 2}
- [ ] Failure condition: {失败条件}

## Execution Boundaries
- Scope: {范围边界}
- Depth: {深度边界}
- Permissions: {权限边界}
- Judgment: {判断边界}

## Non-goals
- {本轮明确不做什么}

## Unknown Handling
- Small unknowns: {小未知如何处理}
- Large unknowns: {大未知何时回问}

## Ask Before Doing
- {必须先问用户的情况}

## Completion Definition
The goal is complete when {完成定义}.
```

---

## 如何安装

### 通用安装方式（适用于 Codex / Claude Code）

使用 `skills` CLI 一键安装：

```bash
npx -y skills add Jasper-Wei1/goal-clarifier -g --all
```

安装后，在 Codex 中可以使用：

```text
$goal-clarifier
```

在 Claude Code 中可以使用：

```text
/goal-clarifier
```

如果你的 Agent 支持 `SKILL.md`，也可以把本仓库作为普通 skill 源使用。核心文件是：

```text
SKILL.md
agents/openai.yaml
```

其中 `SKILL.md` 是主要能力文件，`agents/openai.yaml` 是 Codex UI 元数据。

---

## 如何更新

重新运行安装命令即可更新到仓库最新版本：

```bash
npx -y skills add Jasper-Wei1/goal-clarifier -g --all
```

如果你是在项目中手动复制 `SKILL.md`，重新复制最新版本即可。

---

## 使用示例

### 示例 1：只有模糊想法

```text
$goal-clarifier

我想让 Agent 帮我整理我的自媒体工作流，但我不知道最终应该让它产出什么。
```

它不会直接开始设计系统，而会先帮你判断：

- 你要的是诊断？
- 你要的是目录结构？
- 你要的是迁移方案？
- 你要的是可执行 SOP？
- 哪些东西本轮不处理？

### 示例 2：已有目标但不清楚

```text
$goal-clarifier

帮我把这个项目做得更专业。
```

它会检查“更专业”这个词是否在空转，并追问：

- 做完后能指着什么说完成？
- 什么情况算没做到？
- Agent 应该改代码、写文档、做审查，还是只给建议？
- 不允许它改哪些东西？

### 示例 3：准备交给 Agent 执行

```text
$goal-clarifier

我想把这段需求整理成可以交给 Codex 执行的 goal。
```

它会输出一份结构化 goal，包含目标、交付物、验收标准、执行边界、非目标和回问条件。

---

## 不适合什么场景

这个 skill 不负责直接执行复杂任务。

它更像目标成型前的“澄清层”。当目标已经清楚时，你应该把生成的 goal 交给 Codex、Claude Code 或其他执行型 Agent。

它也不会替用户做重大决策。遇到会改变目标、验收标准或执行边界的问题，它会要求先确认，而不是擅自决定。

---

## 设计原则

### 1. 不把头脑风暴当终点

头脑风暴只负责把用户的未知变成可反应材料。最终仍然要压成目标、验收和边界。

### 2. 不要求用户一开始就说清楚

很多人不是不愿意说清楚，而是还没有足够的表达材料。这个 skill 会先提供候选项、参考形态和低成本草案。

### 3. 不用高级词掩盖模糊

“专业、完整、系统、可落地、有价值”都必须被翻译成可检查条件。

### 4. 不让 Agent 无限扩张任务

边界和非目标是 goal 的一部分，不是补充说明。

---

## 参考与方法来源

`goal-clarifier` 的设计主要参考了两类方法。

### 1. dbs-goal：目标语言空转检测

来源：[dontbesilent2025/dbskill](https://github.com/dontbesilent2025/dbskill)

本项目吸收了 `dbs-goal` 的核心判断：

> 很多目标看起来像目标，但既不能确定下一步行动，也不能识别完成。

`dbs-goal` 中的三个测试被保留下来，并作为目标澄清阶段的基础标准：

- 可指物：完成后能指着什么说“就是这个”
- 可否证：什么情况算没做到
- 有完成态：做到哪里算结束

`goal-clarifier` 在此基础上进一步面向 Agent goal 模式，补充了：

- 验收标准
- 执行边界
- 非目标
- Unknown Handling
- Ask Before Doing
- Completion Definition

也就是说，`dbs-goal` 解决的是“这句话配不配叫目标”，`goal-clarifier` 进一步解决“这句话能不能交给 Agent 执行”。

### 2. Claude Fable：Finding your unknowns

本项目还参考了 Anthropic 的文章：

[A field guide to Claude Fable: Finding your unknowns](https://claude.com/blog/a-field-guide-to-claude-fable-finding-your-unknowns)

这篇文章对本项目最大的启发是：

> 用户经常不是不愿意说清楚，而是不知道自己还不知道什么。

因此，`goal-clarifier` 在正式目标审计前加入了“未知发现”阶段：

- 盲点发现：帮助用户识别自己缺的是目标、标准、路径、边界、偏好还是领域地图
- 候选项反应：给出几个可能方向，让用户通过“接近 / 不要 / 改一半”来表达
- 参考物校准：当用户不知道什么算好时，先给他看几种交付物形态
- 草案压力测试：在最终输出前检查另一个 Agent 执行时会在哪里脑补、越界或做不够

这让 `goal-clarifier` 不只适用于“用户已经能说出一个模糊目标”的情况，也适用于“用户只有一团感觉，还没有可输入目标”的情况。

---

## 文件结构

```text
goal-clarifier/
├── SKILL.md
└── agents/
    └── openai.yaml
```

---

## License

暂未指定 license。使用前请以仓库最新版本为准。
