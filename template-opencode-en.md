# Design Tips and Considerations for Skills and Agents in OpenCode

*Song Congwei*
*Beijing Institute of Mathematical Sciences and Applications (BIMSA)*
*Huairou District, Beijing*

**Abstract**

**Keywords**

---

## 1. Introduction

Briefly introduce OpenCode, then introduce Skill and Agent. State the focus of this paper. Refer to https://opencode.ai/docs for details.

---

## 2. Design Principles

### 2.1 Responsibility
- **Skill Responsibilities**
- **Agent Scheduling**

### 2.2 Naming Conventions

Generally use **lowercase + underscore/hyphen**
- Agent: e.g., `code_reviewer.md`, `socrates.md`.
- Skill: e.g., `find-skills`, `data-analysis`, or `skill-creator`, `skill-vetter`.

### 2.3 File Conventions

- Skill directory structure
- Agent file conventions, especially frontmatter
- Effective use of frontmatter

Examples. Refer to https://agentskills.io.

### 2.4 Security

---

## 3. Design Techniques

To improve the development efficiency and performance of agents/skills, this paper contributes the following design techniques.

### 3.1 Variable Pre-setting

Set variables for frequently reused information.

### 3.2 Custom Commands

Referring to the OpenCode Shell syntax (`!` + command name + arguments), design quick commands.

Examples (more to come):
- `!save <path>`: Automatically save the latest conversation output to the default directory, or a specified path.

### 3.3 Simple Implementation of Agent Memory

### 3.4 Self-Management of OpenCode

The agent memory mechanism can be considered a form of OpenCode self-management. Self-management of OpenCode also includes management of its own configuration information.

### 3.5 Dialogue Examples

Example conversations can be set for the agent. This is essentially in-context learning and can be used for personality distillation. Provide examples.

---

## 4. Overview of Reusable Tools

A skill is not the tool itself. The actual tools are implemented by specific commands. Explain with examples.

---

## 5. Skill Case Studies

Skill case studies (in the paper, keep them concise — highlight key features and use `...` for the rest).

### 5.1 Code Review

### 5.2 Skill/Agent Factory

Automatically generate skills/agents based on prompts.

### 5.3 Academic Paper Writing

### 6. Agent Example: Personal Assistant

Paper writing
Web information gathering
Note taking
Travel planning
Meeting scheduling

---

## 7. Conclusion

---

## References
1. *OpenCode User Manual*. OpenCode Official Documentation. https://opencode.ai/docs.
2. Computer Programming Skills & Maintenance. Journal link: http://www.yipinqikan.com/gongyejishu/jisuanjizidonghualeiqikan/6765.html
3. Flake8 Official Documentation. https://flake8.pycqa.org/
4. Skills.sh Documentation. https://skills.sh/
5. ClawHub AI. https://clawhub.ai/
6. Skill.fish Platform. https://www.skill.fish/
7. Anthropic Skills Open Source Repository. https://github.com/anthropics/skills
8. Agents created by the author. https://github.com/Freakwill/opencode-agents
9. https://agentskills.io
