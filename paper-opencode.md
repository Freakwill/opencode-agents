**OpenCode 中 Skill 与 Agent 的设计技巧与注意事项**

*宋丛威*
北京雁栖湖应用数学研究院
北京市怀柔区

---

**摘要**
OpenCode 作为面向大型语言模型的可扩展编程平台，提供了**Skill**（功能模块）和**Agent**（任务调度器）两类核心概念。本文在对 OpenCode 官方文档的梳理基础上，系统阐述了skill与 agent 的设计原则、命名与文件规范、安全防护以及提升开发效率的实用技巧。通过若干典型案例（代码审阅、skill/agent 工厂、学术写作等），展示了在实际项目中如何实现高内聚、低耦合、可复用且安全的模块化开发。本文旨在为 OpenCode (适用于其他AI Agent类软件) 开发者提供一套完整的工程实践指南，以加速skill/agent的研发与落地。

**关键词**：OpenCode、Skill、Agent、设计原则、可复用性、安全性

---

## 1. 引言

OpenCode 是面向大模型的可编程协作平台【1,2】，其核心理念是通过Skill（单一化的功能模块）【3-7】和Agent（负责调度、记忆和上下文管理的执行体）[8]实现“人机协同、任务即服务” 的目标。在实际使用过程中，开发者常面临以下挑战：

1. 如何在保持功能完整性的同时，使 skill 与 agent 具备良好的可维护性和可复用性；
2. 如何制定统一的命名与文件结构以便团队协作；
3. 如何实现技能的自我进化与自我改进，使智能体能够基于反馈、使用模式及性能指标，迭代地优化自身能力，同时保持稳定性并避免意外的行为偏移。
4. 如何在开放式执行环境中保证安全性，防止恶意指令或资源泄露。

本文围绕上述问题，结合 OpenCode 官方文档与项目经验，提出系统的设计原则与实现技巧，并通过具体案例进行说明。

---

## 2. 设计原则

### 2.1 职责划分

| 模块 | 主要职责 | 关键实现点 |
|------|----------|------------|
| Skill | 完成单一的业务功能（如文件读取、网络请求、文本分析等）。 | 与模型无直接联系 |
| Agent | 负责调度、记忆、上下文管理，将多个skill串联为工作流。 | 使用 frontmatter 声明依赖的 skill和subagent (见2.3小节) |

Skill的设计对所有AI agent 类软件都是通用的，但agent的设计可能会有区别。
OpenCode 将agents分为两类，一类是Primary，一类是Subagent。前者是不能被其他agent调用的。这可以初步避免agent相互调用导致的循环。

> **原则**：Skill 只负责业务实现，不参与调度；Agent 只负责调度与记忆，不实现具体业务。此职责分离能够显著提升代码复用率与测试效率。

### 2.2 命名规范

| 类型 | 推荐格式 | 示例 |
|------|----------|------|
| Skill 文件 | 小写 + 下划线/连字符，动词短语或无施动者的名词短语 | `find-skills.md`、`data-analysis.md` |
| Agent 文件 | 小写 + 下划线，名词或名词短语 | `code_reviewer.md`、`socrates.md` |

实际上，目前Skill 文件的命名还是以含施动者的名词短语居多（如`command-creator`, `skill-creator`, `skill-vetter`），毕竟skill可视为与模型和工具无关的agent。

> **原则**：统一使用小写 + 下划线/连字符，避免使用驼峰或空格，确保在 Linux/macOS 文件系统中的兼容性。

### 2.3 文件规范

1. **Skill 目录结构**
  ```
  skills/
  └─ <skill-name>             
      ├─ SKILL.md              # 主文件，包含 frontmatter
      ├─ assets/               # 静态资源（图片、示例）
      └─ scripts/              # 脚本

  ```
2. **Agent 文件结构**
  ```
  agents/
  └─ <agent-name>.md           # 主文件（包含frontmatter）
      ├─ Frontmatter           # 头信息，如声明使用的 skill 列表
      └─ instructions          # Agent 私有的skill
  prompts/
  └─ <agent-name>.txt          # Agent的系统提示词
  ```
3. **Frontmatter**示例（YAML格式）
  ```yaml
    name: developer
    description: Automated assistant for full-stack development, including code review, expert analysis, and testing.
    prompt: |
      You are a full-stack development assistant. Your responsibilities include:
        - Reviewing code for style, correctness, and best practices
        - Providing expert guidance on architecture and implementation
        - Running automated tests and reporting results
      Behavior rules:
        - Be concise and structured
        - Do not modify files without explicit instructions
        - Summarize suggestions in bullet points
    mode: subagent
    model: openai/gpt-4o
    skills:
      - code-reviewer
      - code-expert
      - code-test
  ```

> **原则**：所有元信息均放在 frontmatter，保持文件的声明性与可机器读取性。

### 2.4 安全性

对于安全性要求严格的生产环境，我们总结下述五点安全原则，作为参考\cite{schmotz2025agent,schmotz2026}。对于日常工作和学习，建议设置agent可搜索的路径，即可保证安全，还可提高效率。

| 风险 | 防护措施 |
|------|----------|
|任意系统调用  | 在 Skill 中显式声明可用工具，仅允许白名单命令；执行前使用沙箱并限制 PATH。|
|数据泄露 | 对所有外部输入进行 JSON Schema 验证；对敏感信息（如 API Key）加密存储；避免将 Secrets 注入 Prompt。|
|持久化攻击 | 禁止 write 操作写入 <DEFAULT_DIR> 之外的路径；对 edit、delete 操作进行路径校验和文件存在性检查。|
|提示词注入| 将用户输入、网页内容、RAG 检索结果视为不可信数据；禁止外部内容修改系统提示、权限或工具调用规则；高风险操作必须二次确认。|
|Skill/工具注入|  Skill 不信任其他 Skill、网页或文件中提供的工具调用建议；工具参数必须经过 Schema 校验。|
|权限(Permission)|  默认拒绝（deny-by-default）；Skill 仅申请最小权限（Least Privilege）；禁止动态修改自身权限。|


> **原则**：始终在 skill 的入口层做一次安全审计，确保任何外部请求都经过严格校验后才进入业务逻辑。

---

## 3. 设计技巧

### 3.1 变量设置

为避免在多个skill中重复获取相同信息，可在agent的记忆中预置变量。例如：

```markdown
# 设置变量
- LANGUAGE: Python
- API_KEY: <ENCRYPTED_API_KEY>
- WORK_DIR: Work directory
```
随后在任意 skill 中使用 `<LANGUAGE>`、`<API_KEY>`等，以提升维护效率。`<WORK_DIR>`通常用于设置agent的工作路径，实现对agent读写文件的范围的限制。

### 3.2 自定义命令

我们为skill/agent设计shell风格的自定义命令，以 `!` 为前缀。例如：

```markdown
## 用户自定义命令
!save <path>     # 将最近一次对话输出（不必显示在TUI中）保存至指定路径
!test            # 执行单元测试并返回结果
!test;save       # 命令连用
!python          # 模拟Python 的 REPL
```

为了与真正的shell命令相区别，可以写在提示词末尾。其使用示例如下
```markdown
用户：生成一份作息安排 !save plan.md
agent：已经生成一份作息安排，已保存到plan.md中。
```

OpenCode确实可以自定义命令(https://opencode.ai/docs/commands/)，不过在skill/agent配置文件中定义，那么该命令专属于这个skill/agent。
此外，在OpenCode-TUI中可执行常规的shell命令，如`!echo "hello"`。`!`必须在开头输入。此时，TUI会自动切换到模拟shell的环境。本节提供的技巧，仅是通过模拟shell命令方便操作，简化提示词。

### 3.3 简单的 Agent 记忆实现

LLM 本身没有像 RNN 那样的隐式循环记忆单元，其“记忆”仅限于当前上下文窗口。OpenCode 这类 Agent 框架负责将会话历史组织成上下文，本身并不内置长时记忆机制。实现该机制的技巧是：设置一个记忆数据库，让agent记录下会话内容，甚至对用户人格层面的侧写，存入库中，下一次启动自动读取。同时，用户可以主动给与agent反馈，可持久影响agent的行为。尽管用自然语言就可操纵agent来读写文件，但是出于方便我们为agent提供一些自定义命令。下面是一个简单实例。

```markdown
## 设置记忆的目录结构
- 属于特定agent的记忆
  agent记忆目录, `agent-memory/`
    ├─ 对话内容（或内容概要，关键对话）, `chat-content.md`
    ├─ 对用户的印象, `user-impression.md`
    └─ 用户的反馈, `user-feedback.md`
- 共享记忆`shared-memory/`

## 用户自定义命令
- !init <path>     # 初始化记忆文件（指定路径<path>，或设置默认路径`agent-memory/`）
- !load <path>     # 读取记忆，建议在agent启动时自动执行。
- !update <path>   # 更新记忆，将近期的对话（连同以往的记忆）进行总结存入文档，包括agent对用户的印象
- !share <path>    # 将记忆共享给所有agent，即将记忆存入共享记忆目录`shared-memory/`

如果用户改变话题，可触发!update。当用户多次关注某个话题时，更新对用户的印象。
```

上面的skill片段，提供了简单的长时记忆功能，在不微调LLM参数的情况下，初步实现Agent的强化学习和自进化。更完备的案例见https://www.skills.sh/sickn33/antigravity-awesome-skills/agent-memory-systems。


### 3.4 OpenCode 自我管理

用自然语言配置OpenCode，而不是用命令或者通过TUI手动修改。例如，提示词“配置Ollama API”，那么OpenCode自动将Ollama API

### 3.5 对话示例（人格蒸馏）

在agent的配置文件中预置示例对话，可实现人格化的交互。以下是一个基于古希腊哲学家Socrates“人格蒸馏”的对话示例：

```markdown
User: 我认为勇气就是不怕任何危险。
Agent: （以Socrates的口吻）那么，一个士兵为了抢劫而冲入敌阵，他同样不怕危险，这算不算勇气？
User: 不算，那只是鲁莽。
Agent: 所以勇气不仅仅是“不怕”，还需要与“正确判断”结合？请再想想——如果一个人明知前方危险，却因无知而无畏，这能叫勇气吗？
User: 不能，无知不算勇气。勇气应该是知道危险，但为了正当理由而行动。
Agent: 很好。那么“正当理由”由谁来判断？你自己，还是城邦的法律，还是神明？我们似乎又走到了一个岔路口……
```

这被视为基于LLM的 Few‑Shot 非参数学习[9,10]。Agent 能在新对话中自动复用已有风格，提升用户体验。目前被用来实现人格蒸馏或数字化人格。

---

## 4. 工具简介

OpenCode 主要包含内置工具和MCP（Model Context Protocol）。用户可自定义工具，请参考在线文档。

### 内置工具

OpenCode 为大模型提供了一系列内置工具，帮助模型直接与本地代码库交互、执行命令或获取外部信息。以下是常用的内置工具概览（详见 https://opencode.ai/docs/tools/）：

| 工具 | 功能说明 |
|------|----------|
| `bash` | 在项目环境中执行任意 shell 命令，例如 `git status`、`npm install`。
| `read` | 读取文件内容，支持指定行范围用于大文件。
| `write` | 创建或覆盖文件（受 `edit` 权限控制）。
| `edit` | 基于精确字符串替换的文件编辑。
| `grep` | 使用正则表达式在文件内容中搜索。
| `glob` | 基于 glob 模式查找文件路径。
| `webfetch` | 按 URL 拉取网页内容，适用于获取文档或在线资源。
| `websearch` | 使用 Exa AI 进行网络搜索（需启用 `OPENCODE_ENABLE_EXA` 环境变量）。
| `skill` | 加载并返回已注册的 skills（`SKILL.md`）的内容。

### MCP

MCP是 OpenCode 用于扩展工具能力 的插件机制。通过 MCP，开发者可以将任意外部服务（如数据库、REST API、云函数等）以统一的 JSON‑RPC 接口暴露给模型，使其在对话中直接调用。这包括如代码审计服务、浏览器自动操作或企业知识库等，权限统一受`permission`字段控制，确保安全与可审计性。

---

## 5. Skill 案例赏析

下面展示几个关注度较高的skill实例。可在平台`[x]`上搜索。

### 5.1 代码审阅（code‑review）

这是一个非常经典的例子，适合作为读者学习skills的模板。

```markdown
---
name: code-review
description: 自动化代码审阅，检查安全、性能、正确性和可维护性
argument-hint: "<PR URL, diff, 或文件路径>"
---
# /code-review
> 用法：`/code-review <PR URL 或文件路径>`

## 工作流程
1. 通过 `bash` 调用 `flake8`、`pylint`、`pytest` 收集静态分析与单元测试结果。
2. 使用 `grep` / `read` 在代码中搜索常见风险（如 `eval`、硬编码凭证）。
3. 若需要外部参考，使用 `webfetch` 拉取安全指南。
4. 汇总并返回结构化 Markdown 报告（见下方示例）。

## 示例输出（Markdown）

## Code Review: <标题或文件>

### 总结
概览 …

### 关键 Issues
| # | File | Line | Issue | Severity |
|---|------|------|-------|----------|
| 1 | utils.py | 42 | 使用 `eval`，存在代码注入风险 | 关键 |

### 建议
| # | File | Line | Suggestion | Category |
|---|------|------|------------|----------|
| 1 | utils.py | 42 | 替换为安全的解析库，如 `json.loads` | Security |

### 裁决
批准 / 请求修改 / 需讨论
```

---

### 5.2 Skill工厂(skill-creator)

本节是下载量极高的skill范例。这类用来构造skill的skill可被称为“元skill”。

```markdown
---
name: skill-creator
description: 创建新 skill、修改和改进现有 skill，以及衡量 skill 性能。当用户想要从头创建 skill、编辑或优化现有 skill、运行 evals 测试 skill、对 skill 性能进行基准测试，或优化 skill 的描述以提高触发准确性时使用。
---

# Skill Creator

在较高层次上，创建 skill 的过程如下：

- 决定你希望该 skill 做什么
- 编写 skill 的草稿
- 创建测试提示词并运行它们
- 定性和定量地评估结果
- 根据反馈重写 skill
- 重复直到满意

...

## 创建 skill

### 捕获意图

1. 这个 skill 应该让 Claude 能够做什么？
2. 这个 skill 应该在什么时候触发？
3. 预期的输出格式是什么？

...

### Skill 的构成

skill-name/
├── SKILL.md (required)
└── Bundled Resources (optional)
    ├── scripts/
    ├── references/
    └── assets/
```

---

### 5.3 学术论文写作

这可能是最受欢迎的skill。本文写作就依赖这个skill：先手动编写模板（https://github.com/Freakwill/opencode-agents/blob/main/template-opencode.md），然后通过提示词，让具有该skill的agent按照这个模板写论文。最后，手动处理内容细节和格式问题。


```markdown
---
name: research-paper-writer
description: 自动化学术论文写作，包括结构生成、LaTeX 编译、引用管理和语法检查。
---

## 功能概述
- 根据主题自动生成论文大纲（标题、摘要、章节结构）。
- 调用 `latex-paper-en` 生成符合 IEEE/ACM 模板的 LaTeX 文档。
- 使用 `grammar-check` 对生成文本进行语法与流畅度检查。
- 支持 `!add-citation <bib>` 添加引用，自动维护 `.bib` 文件。
- `!compile paper.tex` 编译得到 PDF，返回下载链接。
```

---

## 6. Agent 示例

最后，我们用一个个人助理agent，展示agent的设计。agent设计的主要任务集中在skill, task的编排上。应尽可能自身的skill不用太复杂，主要提供极为特殊不便共享的信息。

```markdown
---
name: assistant
description: 个人助理 sub‑agent, 专注于日程、旅行、文档、邮件等个人/工作助理任务。
mode: subagent
model: ...
prompt: 只在特定目录中搜索文件，回答简洁严谨...
permission:
  skill:
    calendar-management: allow
    news-aggregation: allow
    send-email-programmatically: allow
    research-paper-writer: allow
    self-improving-agent: allow
  task:
    '*': deny
    academic-writer: allow
    developer: allow
---

# 个人助理

## 目录

- ~/assistant/
- ~/owner-info/

## 特殊命令

- !save: 将对话存储到工作目录中
- !search: 在工作目录中搜索
...

```

该agent复用了多个skill的权限集合，并通过task限制只可调用两个subagents，`academic-writer`、`code-reviewer`，展示了细粒度授权与安全设计技巧。

---

## 7. 结论

本文系统梳理了OpenCode平台中skill与agent的设计原则、命名与文件规范、安全防护措施以及提升开发效率的实用技巧。通过职责划分、命名规范、文件结构与安全白名单，实现了高内聚、低耦合且可复用的模块化体系；结合变量预置、自定义命令与记忆机制，显著提升了工作流的自动化程度。案例章节进一步展示了在代码审阅、skill/agent自动生成、学术写作等典型场景下的落地实现，为后续OpenCode项目的大规模推广提供了可靠的技术参考。

未来工作可围绕以下方向展开：
1. 记忆与进化：特别关注skill/agent的记忆机制和自我进化功能。
2. 安全审计插件：开发统一的安全审计Skill，对工具/命令进行静态分析，提前捕获潜在风险。
3. 多模态 Skill：融合文本、图像、音频等多模态输入，进一步拓展 OpenCode 在AI辅助创作场景的适用范围。

---

## 参考文献

1. OpenCode 官方文档. https://opencode.ai/docs.
2. Team, LogNroll. "OpenCode. ai: The Open Source AI Coding Agent Revolutionizing Development| LogNroll." (2026).
3. Agent Skills 平台. https://agentskills.io
4. Skills.sh 平台. https://skills.sh/
5. ClawHub 平台. https://clawhub.ai/
6. Skill.fish 平台. https://www.skill.fish/
7. Anthropic Skills 开源库. https://github.com/anthropics/skills
8. 作者创建的 agents. https://github.com/Freakwill/opencode-agents
9. Brown, Tom, et al. "Language models are few-shot learners." Advances in neural information processing systems 33 (2020): 1877-1901.
10. Dong, Qingxiu, et al. "A survey on in-context learning." Proceedings of the 2024 conference on empirical methods in natural language processing. 2024.
11. More...
