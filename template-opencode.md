# OpenCode 中 Skill 与 Agent 的设计技巧与注意事项

*宋丛威*
*北京雁栖湖应用数学研究院*
*北京市怀柔区*

**摘要**

**关键词**


---

## 1. 引言

简单介绍OpenCode，然后介绍Skill，Agent。称述本文重点。具体参考https://opencode.ai/docs。

---

## 2. 设计原则

### 2.1 职责
- **Skill职责**
- **Agent 调度**

### 2.2 命名规范

通常使用 **小写+下划线/连字符**
- Agent：如 `code_reviewer.md`, `socrates.md`。
- Skill 如`find-skills`, `data-analysis`, 或`skill-creator`, `skill-vetter`

### 2.3 文件规范

- Skill目录结构
- Agent文件规范，尤其是frontmatter
- 有效利用frontmatter

举例。参考https://agentskills.io。

### 2.4 安全性


---

## 3. 设计技巧

为了提高agent/skill开发效率和性能，本文贡献下述设计技巧。

### 3.1 变量设置

为反复使用的信息设置变量

### 3.2 自定义命令

参考 OpenCode Shell 语法（`!` + 命令名 + 参数），设计快捷命令。

如（更多例子）
- `!save <path>`，将最近一次对话输出自动写入默认目录，或指目录。

### 3.3 简单的 Agent 记忆实现

### 3.4 OpenCode自我管理

Agent记忆功能可视为一种OpenCode的自我管理。OpenCode的自我管理还包括OpenCode对自己配置信息的管理。

### 3.5 对话举例

可为agent设置对话示范。这部分相当于上下文中学习（In-context learning），可用于实现人格蒸馏。举例。


---

## 4. 重用工具简介

skill不是工具本身。真正的工具是由特定命令实现的。举例解释。


## 5. skill案例

skill案例（在论文中不要写得太长，突出特点，其余用`...`省略）。

### 5.1 代码review

### 5.2 skill/agent工厂

根据提示词，自动生成skill/agent

### 5.3 学术论文写作

### 6. agent实例: 个人助理

论文写作
网络信息收集
笔记整理
旅行安排
会议安排


---

## 7. 结论

---

## 参考文献
1. 《OpenCode 使用手册》. OpenCode 官方文档.https://opencode.ai/docs。
2. 电脑编程技巧与维护. 期刊链接: http://www.yipinqikan.com/gongyejishu/jisuanjizidonghualeiqikan/6765.html
3. Flake8 官方文档. https://flake8.pycqa.org/
4. Skills.sh 文档. https://skills.sh/
5. ClawHub AI. https://clawhub.ai/
6. Skill.fish 平台. https://www.skill.fish/
7. Anthropic Skills 开源库. https://github.com/anthropics/skills
8. 作者创建的 agents. https://github.com/Freakwill/opencode-agents
9. https://agentskills.io

