**OpenCode 中 Skill 与 Agent 的设计技巧与注意事项**

*宋丛威*
北京雁栖湖应用数学研究院
北京市怀柔区

---

### 摘要
OpenCode 作为面向大型语言模型的可扩展编程平台，提供了 **Skill**（功能模块）和 **Agent**（任务调度器）两类核心概念。本文在对 OpenCode 官方文档的梳理基础上，系统阐述了 Skill 与 Agent 的设计原则、命名与文件规范、安全防护以及提升开发效率的实用技巧。通过若干典型案例（代码审阅、Skill/Agent 工厂、学术写作等），展示了在实际项目中如何实现高内聚、低耦合、可复用且安全的模块化开发。本文旨在为 OpenCode 开发者提供一套完整的工程实践指南，以加速 Skill/Agent 的研发与落地。

**关键词**：OpenCode、Skill、Agent、设计原则、可复用性、安全性、代码审阅

---

## 1. 引言

OpenCode 是面向大模型的可编程协作平台，其核心理念是通过 **Skill**（单一、原子化的功能模块）和 **Agent**（负责调度、记忆和上下文管理的执行体）实现 **“人机协同、任务即服务”** 的目标【1】。在实际使用过程中，开发者常面临以下挑战：

1. 如何在保持功能完整性的同时，使 Skill 与 Agent 具备良好的可维护性和可复用性；
2. 如何制定统一的命名与文件结构以便团队协作；
3. 如何在开放式执行环境中保证安全性，防止恶意指令或资源泄露。

本文围绕上述问题，结合 OpenCode 官方文档（2025 版）与项目经验，提出系统的 **设计原则与实现技巧**，并通过具体案例进行说明。

---

## 2. 设计原则

### 2.1 职责划分

| 模块 | 主要职责 | 关键实现点 |
|------|----------|------------|
| **Skill** | 完成 **单一、原子化** 的业务功能（如文件读取、网络请求、文本分析等）。 | - 只暴露统一的 `!command` 接口<br>- 不直接维护全局状态 |
| **Agent** | 负责 **调度、记忆、上下文管理**，将多个 Skill 串联为工作流。 | - 使用 Frontmatter 声明依赖的 Skill<br>- 通过 `!set`、`!get` 实现持久记忆<br>- 处理错误回滚与容错 |

OpenCode 将Agents分为两类，一类是Primary，一类是Subagent。前者是不能别其他agent调用的。这可以初步避免agent相互调用导致的循环。

> **原则**：Skill 只负责业务实现，不参与调度；Agent 只负责调度与记忆，不实现具体业务。此职责分离能够显著提升代码复用率与测试效率。

### 2.2 命名规范

| 类型 | 推荐格式 | 示例 |
|------|----------|------|
| **Skill 文件** | 小写 + **下划线** 或 **连字符** | `find-skills.md`、`data-analysis.md` |
| **Agent 文件** | 小写 + **下划线**，含业务关键词 | `code_reviewer.md`、`socrates.md` |
| **命令** | `!` + 动词 + 可选参数 | `!save <path>`、`!search <keyword>` |

> **原则**：统一使用 *小写 + 下划线/连字符*，避免使用驼峰或空格，确保在 Linux/macOS 文件系统中的兼容性。

### 2.3 文件规范

1. **Skill 目录结构**
   ```
   skills/
   ├─ <skill-name>.md          # 主文件，包含 frontmatter
   ├─ README.md                # 使用说明（可选）
   └─ assets/                  # 静态资源（图片、示例脚本）
   ```
2. **Agent 文件结构**
   ```
   agents/
   ├─ <agent-name>.md           # 主文件，声明使用的 Skill 列表
   └─ prompt.md                 # 示例对话或上下文（可选）
   ```
3. **Frontmatter**示例（YAML格式）
   ```yaml
   name: code_reviewer
   description: 自动化代码审阅与质量检查
   skills:
     - flake8
     - pylint
     - tests_runner
   memory: true               # 是否启用持久记忆
   ```

> **原则**：所有元信息均放在 Frontmatter，保持文件的声明性与可机器读取性。

### 2.4 安全性

| 风险 | 防护措施 |
|------|----------|
| **任意系统调用** | 在 Skill 中显式声明 `allowed_commands`，仅允许白名单命令；执行前使用沙箱（`bash -c`）并限制 `PATH`。 |
| **数据泄露** | 对所有外部输入进行 **JSON schema** 验证；对敏感信息（如 API Key）使用 OpenCode 加密存储 `x` 文件。 |
| **持久化攻击** | 禁止 `!write` 操作写入除 `<DEFAULT_DIR>` 之外的路径；对 `!edit` 进行文件存在性检查。 |
| **交叉脚本注入** | 对所有用户提供的参数进行严格的 **转义** 与 **白名单校验**，避免 Shell 注入。 |

> **原则**：始终在 Skill 的入口层做一次安全审计，确保任何外部请求都经过严格校验后才进入业务逻辑。

---

## 3. 设计技巧

### 3.1 变量设置

为避免在多个 Skill 中重复获取相同信息，可在 **Agent** 的记忆中预置变量。例如：

```markdown
!set language="python"
!set openai_key="${ENCRYPTED_OPENAI_KEY}"
```
随后在任意 Skill 中使用 `${language}`、`${openai_key}`，实现 **全局配置式** 的信息共享，提升维护效率。

### 3.2 自定义命令

OpenCode 支持 **Shell‑style** 的自定义命令，以 `!` 为前缀。例如：

```markdown
!save <path>   # 将最近一次对话输出保存至指定路径
!run_test      # 执行单元测试套件并返回结果
```

Opencode确实可以自定义命令(https://opencode.ai/docs/commands/)，不过在skill/agent配置文件中定义，那么该命令专属于这个skill/agent。

### 3.3 简单的 Agent 记忆实现

OpenCode 只是简单地管理会话内容，不提供长时记忆功能。这需要通过skill实现。这里介绍一种简单的技巧。

```markdown
# 设置记忆的目录结构
- 属于特定agent的记忆
  agent工作目录
    ├─ 对话内容
    └─ 对用户的印象
- 共享记忆

# 用户自定义命令
- !init <path>     # 如果没有记忆文件（指定路径<path>，或设置默认路径），则初始化
- !load <path>     # 读取记忆，建议在agent启动时自动执行。
- !update <path>   # 更新记忆，将近期的对话（连同以往的记忆）进行总结存入文档，包括agent对用户的形象
- !share <path>    # 将记忆共享给所有agent
```

上面的skill片段，提供了简单的长时记忆功能。尽管用自然语言就可操纵agent来存储文件，但是处于方便我们为agent提供一些自定义命令。

### 3.4 OpenCode 自我管理

1. **配置文件统一管理**：所有 **Agent/Skill** 的关键配置信息（如 API Key、路径前缀）统一存放在 `<DEFAULT_DIR>/config.yaml`，并在 Agent 中通过 `!load_config` 加载。
2. **版本控制**：建议使用 `git` 对 `skills/`、`agents/` 目录进行 **细粒度提交**，每次功能更新对应一条 **结构化提交信息**（如 `feat(agent): add memory for code reviewer`），便于审计。

### 3.5 对话示例（人格蒸馏）

在 Agent 的 `prompt.md` 中预置 **示例对话**，可实现人格化的交互。例如：

```markdown
User: 请帮我检查这段代码的 PEP8 合规性。
Agent: 好的，我将使用 `flake8` 检查，并把结果返回给您。
...（后续交互示例） ...
```
通过 **Few‑Shot** 学习，Agent 能在新对话中自动复用已有风格，提升用户体验。

---

## 4. 工具简介

OpenCode 为大模型提供了一系列 **内置工具**，帮助模型直接与本地代码库交互、执行命令或获取外部信息。以下是常用的内置工具概览（详见 https://opencode.ai/docs/tools/）：

| 工具 | 功能说明 |
|------|----------|
| `bash` | 在项目环境中执行任意 shell 命令，例如 `git status`、`npm install`。
| `read` | 读取文件内容，支持指定行范围用于大文件。
| `write` | 创建或覆盖文件（受 `edit` 权限控制）。
| `edit` | 基于精确字符串替换的文件编辑。
| `apply_patch` | 应用统一格式的补丁文件，常用于批量修改。
| `grep` | 使用正则表达式在文件内容中搜索。
| `glob` | 基于 glob 模式查找文件路径。
| `skill` | 加载并返回已注册的 Skill（`SKILL.md`）的内容。
| `todowrite` | 管理任务列表，帮助 LLM 组织多步骤工作流。
| `webfetch` | 按 URL 拉取网页内容，适用于获取文档或在线资源。
| `websearch` | 使用 Exa AI 进行网络搜索（需启用 `OPENCODE_ENABLE_EXA` 环境变量）。
| `question` | 在执行任务时向用户发起交互式提问。

### MCP 服务器概述

MCP（Model Context Protocol）服务器是 OpenCode 用于 **扩展工具能力** 的插件机制。通过 MCP，开发者可以将任意外部服务（如数据库、REST API、云函数等）以统一的 JSON‑RPC 接口暴露给模型，使其在对话中直接调用。这包括如 **代码审计服务**、**CI/CD 系统** 或 **企业知识库** 等，权限统一受 `permission` 配置控制，确保安全与可审计性。

---

## 5. Skill 案例

下面展示几个 **Skill** 示例，使用 OpenCode 官方示例与 Anthropic 的 `code‑review` Skill 为参考，实现了更完整的声明和权限配置。

### 5.1 代码审阅（code‑review）

```markdown
---
name: code-review
description: 自动化代码审阅，检查安全、性能、正确性和可维护性
argument-hint: "<PR URL, diff, 或文件路径>"
permission:
  skill:
    bash: allow          # 用于执行 lint、测试等命令
    edit: allow          # 允许在报告中插入代码片段
    read: allow
    grep: allow
    webfetch: allow
metadata:
  version: 1.0
  author: claude
---
# /code-review
> 用法：`/code-review <PR URL 或文件路径>`

## 工作流程
1. 通过 `bash` 调用 `flake8`、`pylint`、`pytest` 收集静态分析与单元测试结果。
2. 使用 `grep` / `read` 在代码中搜索常见风险（如 `eval`、硬编码凭证）。
3. 若需要外部参考，使用 `webfetch` 拉取安全指南。
4. 汇总并返回结构化 Markdown 报告（见下方示例）。

## 示例输出（Markdown）
```markdown
## Code Review: <标题或文件>

### Summary
概览 …

### Critical Issues
| # | File | Line | Issue | Severity |
|---|------|------|-------|----------|
| 1 | utils.py | 42 | 使用 `eval`，存在代码注入风险 | 🔴 Critical |

### Suggestions
| # | File | Line | Suggestion | Category |
|---|------|------|------------|----------|
| 1 | utils.py | 42 | 替换为安全的解析库，如 `json.loads` | Security |

### Verdict
Approve / Request Changes / Needs Discussion
```
```

---

### 5.2 代码生成（code‑generator）

```markdown
---
name: code-generator
description: 根据自然语言描述生成代码片段，支持多语言（Python、Go、TS 等）
permission:
  skill:
    edit: allow
    write: allow
    read: allow
    bash: ask        # 需要用户确认后才执行编译或运行命令
metadata:
  version: 1.0
---
# /code-generator
> 用法：`/code-generator "生成一个读取 CSV 并计算均值的 Python 脚本"`

实现步骤：
1. 使用 LLM 生成代码字符串。
2. 用 `write` 将代码写入临时文件（`/tmp/generated.py`）。
3. 若用户同意，使用 `bash` 运行 `python /tmp/generated.py` 并返回结果。
```

---

## 6. Agent 示例

### 6.1 assistant（个人助理）

```markdown
---
name: assistant
description: 个人助理 sub‑agent, 兼具 Kim 的全部功能，专注于日程、旅行、文档、邮件等个人/工作助理任务。
mode: subagent
permission:
  skill:
    calendar-management: allow
    news-aggregation: allow
    send-email-programmatically: allow
    research-paper-writer: allow
    self-improving-agent: allow
    task-tracking: allow
  task:
    '*': deny
    academic-writer: allow
    code-reviewer: allow
---
```

该 Agent 复用了多个skill的权限集合，并通过 **task** 限制只可调用两个subagents，`academic-writer`、`code-reviewer`，展示了细粒度授权与安全设计技巧。

---

## 7. 结论

本文系统梳理了 **OpenCode** 平台中 **Skill** 与 **Agent** 的设计原则、命名与文件规范、安全防护措施以及提升开发效率的实用技巧。通过 **职责划分**、**统一命名**、**文件结构约定** 与 **安全白名单**，实现了高内聚、低耦合且可复用的模块化体系；结合 **变量预置**、**自定义命令** 与 **记忆机制**，显著提升了工作流的自动化程度。案例章节进一步展示了在 **代码审阅、Skill/Agent 自动生成、学术写作** 等典型场景下的落地实现，为后续 OpenCode 项目的大规模推广提供了可靠的技术参考。

**未来工作** 可围绕以下方向展开：
1. **自动化测试框架**：为每个 Skill 自动生成单元测试，实现 CI/CD 流水线的全流程验证。
2. **安全审计插件**：开发统一的安全审计工具，对 `allowed_commands` 进行静态分析，提前捕获潜在风险。
3. **多模态 Agent**：融合文本、图像、音频等多模态输入，进一步拓展 OpenCode 在 **AI 辅助创作** 与 **交互式教学** 场景的适用范围。

---

## 参考文献

1. OpenCode 使用手册. OpenCode 官方文档. https://opencode.ai/docs. (2025)
2. 《电脑编程技巧与维护》杂志社. 期刊简介与投稿指南. http://www.yipinqikan.com/gongyejishu/jisuanjizidonghualeiqikan/6765.html. (2025)
3. Flake8 官方文档. https://flake8.pycqa.org/. (2024)
4. Pylint 官方文档. https://pylint.org/. (2024)
5. Pandoc 手册. https://pandoc.org/. (2024)
6. SymPy 官方文档. https://www.sympy.org/. (2024)
7. NumPy 与 SciPy. https://numpy.org/、https://scipy.org/. (2024)
8. OpenAI API 文档. https://platform.openai.com/docs. (2025)

