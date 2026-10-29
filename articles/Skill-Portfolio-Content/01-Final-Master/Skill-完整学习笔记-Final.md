---
title: "Skill：概念、技术原理与多场景应用"
description: "Skill 如何用渐进加载、指令和资源复用工作方法？在不同场景下，又该怎样判断它与脚本、MCP 是否适合配合？"
category: "Agent Systems"
subcategory: "Skill"
tags:
  - "Agent Skills"
  - "Progressive Disclosure"
  - "SKILL.md"
  - "MCP"
type: "learning-note"
sources: []
status: "reviewed"
---

# Skill：概念、技术原理与多场景应用

> 本文整理自[@技术爬爬虾【Agent Skills (Claude Skills) 详细攻略，一期视频精通】](https://www.bilibili.com/video/BV1HuiyBQE9G/?share_source=copy_web&vd_source=0e7a33d9084dfc869a888b89447e1a46)，保留原有示例、操作过程与选型思考。文中的 Claude Code、Codex 与开源仓库操作分别对应特定工具或示例，不代表所有 Agent 的统一实现。文末“个人选型思考”是学习阶段的判断，并非项目实测结论。

## 1. Skill 是什么？

我把 Skill 理解为一份“**带目录的说明书**”：它将某类重复任务所需的提示词、操作步骤与辅助文件放在同一个文件夹中，让 Agent 在需要做这类任务时找到并使用它。这样做的价值不仅是复用提示词，也让较长的规则和例子有了固定的存放位置，不必每次在对话中重新解释。

在支持 Agent Skills 的工具中，常见做法是先让 Agent 知道 Skill 的名称与用途；当任务匹配时，再读取完整指令；需要脚本、范文或模板时，再访问对应文件。这就是“渐进式披露”的直观含义。它有机会减少一开始放入上下文的内容，但实际节省多少 Token，取决于工具如何展示技能列表、是否调用该 Skill，以及后续读入了多少材料，不能理解为必然大幅节省。

因此，Skill 的几个特点可以概括为：**可复用、可分项管理，在支持的工具中可按任务匹配自动触发，也可由用户显式调用**。不同工具的发现路径和调用方式仍需分别确认。

## 2. 三层渐进加载：先知道“有什么”，再读取“怎么做”

| 层级 | 内容 | 典型读取时机 | 类比 |
| --- | --- | --- | --- |
| 元数据 | `name`、`description` | Agent 发现可用 Skill 时 | 目录 |
| 指令 | 任务流程、规则与示例，写在 `SKILL.md` 正文 | Skill 被选中或显式调用时 | 正文 |
| 资源 | 脚本、参考文档、模板、图片等 | 执行步骤确实需要时 | 附录 |

这张表帮助我理解了为什么不用把所有材料放在同一段提示词里：先用简短描述判断“这份说明书是否相关”，相关时才展开说明书，再按具体步骤取用附录。

## 3. 一个 Skill 文件夹怎样组织？

一个 Skill 通常是一个文件夹，核心文件是 `SKILL.md`。在 Agent Skills 开放规范中，它使用 YAML frontmatter 写元数据，后面接 Markdown 指令；`scripts/`、`references/`、`assets/` 是可选的辅助目录。

```text
skill-name/
├── SKILL.md          # 元数据 + 指令（核心文件）
├── scripts/          # 可执行脚本（可选）
│   └── main.py
├── references/       # 补充文档（可选）
│   └── doc.md
└── assets/           # 图片、模板等（可选）
    └── pic.png
```

这里的 `pic.png` 是目录结构示例。把文件分开放，也意味着后续可以单独修改脚本、范文或模板，不必将它们全部塞进指令正文。

### `SKILL.md` 写什么？

```markdown
---
name: skill-name
description: 简要说明该 Skill 的用途，以及 AI 在什么场景下应调用它
---

# 指令部分
具体的任务拆解、步骤、规则、示例
```

`name` 和 `description` 让 Agent 知道这是什么、适合什么任务；正文则告诉它选中后怎样做。下方两张原图分别展示了“字幕转 Markdown 笔记”Skill 的元数据，以及正文中关于截图位置、输出格式等操作规则。看图时可以留意：**第一张帮助匹配任务，第二张才进入具体执行要求**。

![字幕转 Markdown Skill 的名称与用途说明](图片附件/Pasted%20image%2020260726120934.png)

![同一 Skill 中关于截图标记与输出格式的指令](图片附件/Pasted%20image%2020260726121109.png)

### 元数据还可以写哪些字段？

以上字段很有助于理解“谁能调用、何时调用”，但它们不能一概视为 Agent Skills 开放规范的必填字段。下表按 **Claude Code 的配置语境**保留原来的比较；迁移到其他工具前，要核对目标工具是否识别。

| 字段 | 在 Claude Code 中的作用 | 使用时的考虑 |
| --- | --- | --- |
| `when_to_use` | 补充适用场景或触发短语 | 当 `description` 不足以说明任务边界时再加 |
| `disable-model-invocation` | 设为 `true` 时，不让 Claude 自动调用 | 希望仅由用户手动触发时使用；默认是 `false` |
| `user-invocable` | 设为 `false` 时，从用户的 `/` 菜单隐藏，也不接受用户直接输入 `/name` 调用 | 只希望 Claude 在后台按需使用时考虑；默认是 `true` |
| `effort` | 设置调用该 Skill 时的推理投入级别 | 属于 Claude Code 的平台配置，支持值还取决于所用模型；不宜机械地给所有 Skill 设为 `low` |

## 4. 怎样安装、创建和调用？

可以手动编写 `SKILL.md` 和辅助文件；
也可以从 GitHub 下载或克隆已有 Skill 文件夹，放进工具识别的位置；
还可以使用分享网站、安装工具或插件。如skills分享网站skills.sh,通过命令自动下载并放在目录里；
对于已有明确流程的任务，可以让 AI 辅助起草，再由人检查指令和附带文件。某些工具还提供专用创建向导，但命令名称与安装条件依平台而变；例如 Claude Code 的 `skill-creator` 是插件提供的能力，不能假设每个安装环境都自带同名命令。

### 以 Claude Code 中处理 SRT 字幕为例

在 Claude Code 的项目级配置中，Skill 可以放在 `项目目录/.claude/skills/skill-name/`；如果希望跨项目使用，可以放在 `~/.claude/skills/`。下面沿用教学视频中的字幕任务，把“文件是怎样被发现和使用的”连成一个过程。

1. 在项目目录启动 Claude Code，例如在终端输入 `claude`。需要查看可用技能时，可通过当前版本的技能列表或 `/` 菜单确认；原笔记使用 `/skills`，具体命令以所用版本的界面为准。
2. 用户提出任务：“帮我处理这个 SRT 字幕文件。”
3. Claude Code 发现可用 Skill，并把用于判断适用场景的名称、描述等信息提供给模型。
4. 模型根据任务和描述判断是否使用“字幕转 Markdown 笔记”Skill。用户也可以显式调用它。
5. 选中后，工具读取 `SKILL.md` 的任务规则；若后续步骤需要脚本或参考文件，再访问相应资源。
6. Agent 依指令处理字幕，必要时调用工具或脚本，最后输出 Markdown 笔记，或者按 Skill 要求写入文件。

笔记还记录了“Agent 询问用户是否使用该 Skill、用户输入 Yes”的交互。这可以出现在需要授权的操作或某些配置中，但**并非每次调用 Skill 的固定步骤**；自动调用、显式调用与工具权限确认是不同的事。最终产物也不一定是 JSON 调用请求，取决于该任务要完成什么。

下图展示的是个人目录下的 `.claude/skills` 文件夹，可用来对照“项目级”和“个人级”的存放位置。

![个人 Claude Code 目录中的 skills 文件夹](图片附件/Pasted%20image%2020260726121920.png)

### 从 Claude Code 迁移到 Codex：文件可复用，接入方式要核对

由于两者都支持以 `SKILL.md` 为核心的 Skill，已有指令、脚本和参考资料可以作为迁移起点；但“复制过去就一定能直接运行”并不成立。平台可能使用不同的发现目录、调用方式、权限机制与扩展字段。尤其要检查 Skill 是否依赖 Claude Code 专属命令、工具名或路径。

原笔记的操作记录是：打开 `C:\Users\用户名\.codex\config.toml`，启用 `[features] skills = true`；把 Skill 从 `项目目录/.claude/skills/skill-name/` 复制到 `项目目录/.codex/skills/skill-name/`；启动 Codex，输入 `/skill` 验证。下图保留了当时的配置截图，**作为学习过程中的旧版操作记录**。

![原笔记记录的 Codex skills 配置截图](图片附件/Pasted%20image%2020260721165747.png)

按目前的 Codex 文档，本地项目 Skill 的发现位置是 `.agents/skills/`，用户级位置是 `$HOME/.agents/skills/`；新建后可检查是否被发现，需要时重启。注意：实际迁移时应先核对正在使用的 Codex 版本与官方说明，以上旧截图理解为当时的配置经验，而不是当前所有版本的安装指南。

### 使用别人分享的 Skill

以提到的 [Awesome Claude Skills](https://github.com/ComposioHQ/awesome-claude-skills) 为例：先在 GitHub 查看仓库和具体 Skill，再下载或克隆；挑选需要的文件夹，放到目标工具识别的目录；重新打开或刷新工具后，提出符合该 Skill 用途的请求，观察它是否被发现、是否实际按照指令工作。**能被识别，不等于输出一定符合预期**；尤其是下载自他人的脚本与指令，使用前应检查内容。

## 5. 资源层：脚本、参考资料怎样参与任务？

`SKILL.md` 可以只包含文字指令，也可以配套脚本、文档、图片和模板。原图展示了 `SKILL.md` 与 `scripts/`、`references/`、`assets/` 并列的组织方式。它们的角色不同：指令说明“该怎样做”，资源则在某一步提供执行能力或具体材料。

![Skill 文件夹中指令与脚本、参考文档、素材的关系](图片附件/Pasted%20image%2020260721184406.png)

### 案例一：用脚本从视频截取画面

以“字幕转 Markdown 笔记”为开源示例。[tech-shrimp/agent-skills-examples](https://github.com/tech-shrimp/agent-skills-examples) 中的“字幕转markdown”目录确实包含 `SKILL.md` 和 `scripts/screenshot.py`，其指令也写了在生成笔记后调用该脚本对视频截图。

这个例子的调用过程值得完整保留，因为它说明了模型、指令和脚本怎样接力：

1. 用户提出“对这段视频进行截图并生成笔记”。
2. Agent 根据任务选中相应 Skill，读取 `SKILL.md` 的规则。
3. 指令告知 Agent 在合适步骤运行 `scripts/screenshot.py`。
4. Agent 通过可用执行环境调用脚本，取得截图或截图文件路径等结果。
5. Agent 把脚本结果与字幕内容结合，生成最后的 Markdown 笔记。

这样，重复的截图动作可以交给脚本处理，模型重点负责判断何时运行、怎样组织笔记。脚本源码**不一定需要在初始上下文中完整展开**；但若 Agent 为理解或修改脚本而读取它，源码仍可能进入上下文。脚本在本机执行也不自动意味着“代码安全”，仍需检查来源、运行权限与环境。

### 案例二：用 `references/` 提供写作范文

如果任务不是执行程序，而是让 AI 参考一篇范文的行文方式，可以把范文放在 `references/`：

```text
writing-assistant/
├── SKILL.md
└── references/
    └── sample-article.md   # 范文，用于学习行文风格
```

详细过程是：用户给 Claude Code 写作要求和材料；Agent 选中写作 Skill；`SKILL.md` 要求先读 `references/sample-article.md`；Agent 使用 `Read` 读取范文，再结合用户材料写作。这里的 `Read` 是该工具环境中的读取动作，不是 Skill 开放规范要求所有 Agent 都必须拥有的同名工具。图中可以重点看**指令如何引导 Agent 再去取用范文**，而非在任务一开始就把范文全文放进技能描述。

![写作请求中 Agent 选用 Skill 并读取 references 范文的流程](图片附件/Pasted%20image%2020260721222001.png)

## 6. Skill 与 MCP：相近任务里各自解决什么问题？


| 比较角度 | Skill 在本文示例中的侧重点 | MCP 在本文示例中的侧重点 |
| --- | --- | --- |
| 直观类比 | 带目录的任务说明书 | 连接外部能力的标准化接口或“工具箱” |
| 主要用途 | 固化流程、规则、模板，指导 Agent 怎样完成任务 | 让客户端发现并使用服务提供的工具、资源或提示词 |
| 常见载体 | `SKILL.md`，可附脚本和资料 | MCP 服务及其暴露的能力 |
| 上下文成本 | 可先只展示简短描述；使用后仍会加载指令或资料 | 与展示的能力描述、调用结果和客户端实现有关 |
| 制作与维护 | 纯指令型 Skill 往往可以先从 Markdown 写起；带脚本时也需处理运行环境 | 自建 MCP 服务通常涉及接口、服务运行与维护；使用现成服务则不一定需要自己开发 |

注：Skill执行脚本是他的辅助功能，因为每台电脑的Python环境不同，因此Skill脚本执行失败的概率比较高，而MCP是一个由Nodejs或Python编写的软件包，执行成功率>Skill，不过MCP 服务也有安装、鉴权、网络和运行环境等故障点。


### 两者可以怎样配合？

二者可以互补，**各司其职**：

- **Skills** 负责**管理提示词**，将复杂流程、规则、模板固化。
    
- **MCP** 负责**执行具体操作**（如创建 GitHub 仓库、调用外部 API）。

>原则：**Skill 负责“怎么说”，MCP 负责“做什么”**，发挥各自的长处。实际使用时，Skill 也可能包含脚本，MCP 也能提供资源和提示词，边界并非完全互斥。

具体做法是：先在 Claude Code 等客户端配置 GitHub MCP 服务，再在某个 Skill 的指令中写明，创建仓库时调用它提供的 `create_repository` 工具。如果用户请求符合这份 Skill 的用途，Agent 可以遵循指令调用相应 MCP 工具，并根据返回结果继续处理。要确认具体工具名、参数和权限，应查看**所使用的 MCP 服务文档或客户端实际列出的工具**，不能只凭一个示例推定所有 GitHub MCP 服务完全相同。

下图是一段 Skill 指令，写了先检查 `ai-docs` 仓库、必要时调用 `create_repository`，再用 `create_or_update_file` 上传文件。它展示的是**期望的操作流程**，并不是仓库已经创建或文件已经上传的执行记录。

![Skill 指令中安排 GitHub MCP 建仓与上传文件的步骤](图片附件/Pasted%20image%2020260726170653.png)

## 7. 官方与社区资源

- [Agent Skills 开放规范](https://agentskills.io/specification)：查看 `SKILL.md`、元数据和辅助目录的通用格式。
- [Anthropic 官方 Skills 仓库](https://github.com/anthropics/skills)：查看 Skill 文件夹示例。
- [Awesome Claude Skills 社区仓库](https://github.com/ComposioHQ/awesome-claude-skills)：寻找社区分享的技能；使用前仍需检查具体内容。
- [tech-shrimp 的字幕转 Markdown 示例](https://github.com/tech-shrimp/agent-skills-examples)：对应上文带截图脚本的案例。

## 8. 我的选型思考：什么时候值得做成 Skill？

以下是我基于这份学习材料形成的判断，**不是已经在项目中验证过的效果**。

**个人短期、小型项目。** 如果整个项目只需要一两条固定提示规则，我倾向于先把它们写在项目 README、`CLAUDE.md` 或工具支持的同类项目说明里。此时专门建立 Skill 文件夹，可能增加学习和维护成本，却未必带来明显收益。简单文本文件加检索也能覆盖“把一段提示词按需找出来”的部分需求；已有 MCP 的资源或工具也可能承担按需取用提示材料的某些环节。不过这些做法不等于完整复刻 Skill 的发现、调用、资源组织与权限机制。若目的是把互不干扰的子任务分开处理，使用独立会话或支持的子 Agent 可能更合适。Skill 在被加载后仍会占用上下文，因此并非“用了 Skill 就不会有上下文成本”。我的结论是：**个人少量、简单需求下，Skill 可以是可选项，不必为了形式而使用**。

**多人长期协作。** 如果团队需要反复完成相似任务，我更看重 Skill 把流程、模板、示例和检查标准集中维护的作用：大家可以复用同一份规则，减少每个人重新编写提示词的工作。对于只需说明步骤、不接外部系统的场景，从 Markdown Skill 起步可能比自建 MCP 服务轻；如果要访问外部系统或稳定执行操作，则仍需选择合适的工具、脚本或 MCP 服务。团队可以让 AI 辅助起草与修改 Skill，再由人审查并做版本管理；不能仅凭写了 Skill 就保证所有人输出完全一致、合规，也不能假设 AI 会自动完成可靠的更新。这里的主要价值在于**让可复用的工作方法有明确载体**，节省 Token 只是可能收益之一。

据此，我会优先考虑以下三类场景：

1. 高频、流程相对标准化的重复任务。
2. 多人协作、需要共用输出规范和模板的团队工作。
3. 单段简单提示词难以稳定交代清楚、需要配合范文、示例和步骤的任务；若没有外部工具调用需求，可以先尝试轻量的指令型 Skill。

这仍是选型起点。最终是否采用 Skill、脚本、MCP 或其他方式，要看任务复杂度、团队维护成本、宿主工具能力和实际效果。
