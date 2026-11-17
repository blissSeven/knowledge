---
title: The Complete Guide to Building Skills for Claude
source: The-Complete-Guide-to-Building-Skill-for-Claude.pdf
publisher: Anthropic
type: 学习笔记 / 整理
tags: [Agent Skills, Claude, Skill 编写, prompt engineering]
---

# The Complete Guide to Building Skills for Claude

> 本文是 Anthropic 官方《The Complete Guide to Building Skills for Claude》（2026-01）的中文整理笔记。
> Skill 是一组指令，打包成一个简单文件夹，教 Claude 如何处理特定任务或工作流。与其在每次对话中重复解释偏好、流程和领域知识，不如教 Claude 一次，之后每次都受益。

## 目录

- [简介](#简介)
- [第 1 章 基础（Fundamentals）](#第-1-章-基础fundamentals)
- [第 2 章 规划与设计（Planning and design）](#第-2-章-规划与设计planning-and-design)
- [第 3 章 测试与迭代（Testing and iteration）](#第-3-章-测试与迭代testing-and-iteration)
- [第 4 章 分发与共享（Distribution and sharing）](#第-4-章-分发与共享distribution-and-sharing)
- [第 5 章 模式与故障排查（Patterns and troubleshooting）](#第-5-章-模式与故障排查patterns-and-troubleshooting)
- [第 6 章 资源与参考（Resources and references）](#第-6-章-资源与参考resources-and-references)
- [附录 A 快速清单](#附录-a-快速清单)
- [附录 B YAML frontmatter](#附录-b-yaml-frontmatter)
- [附录 C 完整技能示例](#附录-c-完整技能示例)

---

## 简介

**Skill 的本质**：一组指令，打包成一个简单文件夹，教 Claude 如何处理特定任务或工作流。这是为特定需求定制 Claude 的最强大方式之一。

**适用场景**（存在可重复的工作流时）：
- 从规格说明生成前端设计
- 用一致的方法论做研究
- 创建符合团队风格指南的文档
- 编排多步骤流程

Skill 与 Claude 的内置能力（代码执行、文档创建）配合良好。对 MCP 集成开发者来说，Skill 增加了一层能力，把原始的工具访问转化为可靠、优化的工作流。

**你将学到**：
- Skill 结构的技术要求与最佳实践
- 独立 Skill 与 MCP 增强工作流的模式
- 跨场景验证有效的模式
- 如何测试、迭代和分发 Skill

**适合谁**：
- 希望 Claude 一致遵循特定工作流的开发者
- 高级用户（power users）
- 想在整个组织内标准化 Claude 工作方式的团队

**两条阅读路径**：
- 构建独立 Skill：重点看「基础」「规划与设计」以及 category 1–2
- 增强 MCP 集成：看「Skills + MCP」章节和 category 3

> 目标：读完本指南后，能在一次工作时段内构建出一个可用的 Skill。使用 skill-creator 构建并测试第一个可用 Skill 大约需要 15–30 分钟。

---

## 第 1 章 基础（Fundamentals）

### 什么是 Skill

一个 Skill 就是一个文件夹，包含：

```
skill-name/
├── SKILL.md (必需)：含 YAML frontmatter 的 Markdown 指令
├── scripts/ (可选)：可执行代码（Python、Bash 等）
├── references/ (可选)：按需加载的文档
└── assets/ (可选)：输出中使用的模板、字体、图标
```

### 核心设计原则

#### 1. 渐进式披露（Progressive Disclosure）

Skill 使用三级系统：

| 层级 | 内容 | 加载时机 |
|------|------|----------|
| 第一级：YAML frontmatter | 始终加载在 Claude 的系统提示中 | 始终；只提供足够信息让 Claude 判断何时使用该 Skill，不加载全部内容 |
| 第二级：SKILL.md body | 完整的指令与指导 | Claude 认为该 Skill 与当前任务相关时 |
| 第三级：链接文件 | Skill 目录内打包的额外文件 | Claude 按需导航和发现 |

渐进式披露在保持专业能力的同时，最小化 token 消耗。

#### 2. 可组合性（Composability）

Claude 可以同时加载多个 Skill。你的 Skill 应该能与其他 Skill 良好协作，不要假设自己是唯一可用的能力。

#### 3. 可移植性（Portability）

Skill 在 Claude.ai、Claude Code 和 API 中工作方式一致。只要环境支持 Skill 所需的依赖，创建一次即可在所有平台使用，无需修改。

### 面向 MCP 开发者：Skills + Connectors

> 如果你不构建 MCP 的独立 Skill，可跳到「规划与设计」，之后随时回来。

如果你已有可运行的 MCP server，最难的部分已经完成了。Skill 是其上的知识层——捕获你已知的工作流和最佳实践，让 Claude 能一致地应用它们。

**厨房类比**：
- **MCP 提供专业厨房**：对工具、原料和设备的访问
- **Skill 提供菜谱**：如何创造有价值东西的分步指令

两者结合，让用户无需自己弄清每一步就能完成复杂任务。

| MCP（连接性） | Skills（知识） |
|---------------|----------------|
| 把 Claude 连接到你的服务（Notion、Asana、Linear 等） | 教 Claude 如何有效使用你的服务 |
| 提供实时数据访问和工具调用 | 捕获工作流和最佳实践 |
| Claude 能做什么（What Claude can do） | Claude 应该怎么做（How Claude should do it） |

**为什么对你的 MCP 用户很重要**：

*没有 Skill*：
- 用户连接了 MCP，却不知道下一步做什么
- 支持工单不断问「我怎么用你们的集成做 X」
- 每次对话都从零开始
- 用户提示方式不同，结果不一致
- 真正的问题是工作流指导缺失，用户却怪罪连接器

*有了 Skill*：
- 预构建的工作流在需要时自动激活
- 一致、可靠的工具使用
- 每次交互都嵌入最佳实践
- 降低集成的学习曲线

---

## 第 2 章 规划与设计（Planning and design）

### 从用例开始

在写任何代码之前，先确定 2–3 个你的 Skill 要支持的具体用例。

**好的用例定义**（示例）：

> **用例**：项目 Sprint 规划
> **触发条件**：用户说 "help me plan this sprint" 或 "create sprint tasks"
> **步骤**：
> 1. 从 Linear 获取当前项目状态（通过 MCP）
> 2. 分析团队速度和承载能力
> 3. 建议任务优先级
> 4. 在 Linear 中创建带正确标签和预估的任务
> **结果**：完整的 Sprint 规划，任务已创建

**自问清单**：
- 用户想完成什么？
- 这需要什么多步骤工作流？
- 需要哪些工具（内置或 MCP）？
- 应该嵌入哪些领域知识或最佳实践？

### 常见的 Skill 用例类别

Anthropic 观察到三类常见用例：

#### 类别 1：文档与资产创建（Document & Asset Creation）

**用于**：创建一致、高质量的输出，包括文档、演示文稿、应用、设计、代码等。

**真实例子**：frontend-design skill（另见 docx、pptx、xlsx、ppt 相关 skill）

> "Create distinctive, production-grade frontend interfaces with high design quality. Use when building web components, pages, artifacts, posters, or applications."

**关键技术**：
- 嵌入风格指南和品牌标准
- 模板结构，保证输出一致
- 定稿前的质量检查清单
- 无需外部工具——使用 Claude 内置能力

#### 类别 2：工作流自动化（Workflow Automation）

**用于**：受益于一致方法论的多步骤流程，包括跨多个 MCP server 的协调。

**真实例子**：skill-creator skill

> "Interactive guide for creating new skills. Walks the user through use case definition, frontmatter generation, instruction writing, and validation."

**关键技术**：
- 带验证关卡（validation gates）的分步工作流
- 常见结构的模板
- 内置的审查和改进建议
- 迭代优化循环

#### 类别 3：MCP 增强（MCP Enhancement）

**用于**：增强 MCP server 工具访问的工作流指导。

**真实例子**：sentry-code-review skill（来自 Sentry）

> "Automatically analyzes and fixes detected bugs in GitHub Pull Requests using Sentry's error monitoring data via their MCP server."

**关键技术**：
- 按顺序协调多个 MCP 调用
- 嵌入领域专业知识
- 提供用户原本需要自己指定的上下文
- 针对常见 MCP 问题的错误处理

### 定义成功标准

你怎么知道 Skill 起作用了？这些是期望目标——粗略的基准而非精确阈值。追求严谨，但也接受一定程度的直觉判断。

**定量指标**：
- **相关查询中 90% 触发 Skill** —— 测量方法：跑 10–20 个应该触发的测试查询，统计自动加载 vs. 需要显式调用的次数
- **在 X 次工具调用内完成工作流** —— 测量方法：对同一任务对比启用/不启用 Skill，统计工具调用数和总 token 消耗
- **每个工作流 0 次失败 API 调用** —— 测量方法：测试运行时监控 MCP server 日志，跟踪重试率和错误码

**定性指标**：
- **用户不需要提示 Claude 下一步** —— 评估：测试时记录你需要重定向或澄清的频率，询问 beta 用户反馈
- **工作流无需用户纠正即可完成** —— 评估：同一请求跑 3–5 次，比较输出的结构一致性和质量
- **跨会话结果一致** —— 评估：新用户能否在最少指导下首次完成任务

### 技术要求

#### 文件结构

```
your-skill-name/
├── SKILL.md                 # 必需 - 主技能文件
├── scripts/                 # 可选 - 可执行代码
│   ├── process_data.py      # 示例
│   └── validate.sh          # 示例
├── references/              # 可选 - 文档
│   ├── api-guide.md         # 示例
│   └── examples/            # 示例
└── assets/                  # 可选 - 模板等
    └── report-template.md   # 示例
```

#### 关键规则

**SKILL.md 命名**：
- 必须精确为 `SKILL.md`（区分大小写）
- 不接受任何变体（`SKILL.MD`、`skill.md` 等）

**Skill 文件夹命名**：
- 使用 kebab-case：`notion-project-setup` ✅
- 不要空格：`Notion Project Setup` ❌
- 不要下划线：`notion_project_setup` ❌
- 不要大写：`NotionProjectSetup` ❌

**不要 README.md**：
- Skill 文件夹内不要包含 README.md
- 所有文档放进 SKILL.md 或 references/
- 注意：通过 GitHub 分发时，你仍然需要一个仓库级 README 给人类用户看（见第 4 章）

### YAML frontmatter：最重要的部分

frontmatter 是 Claude 决定是否加载你的 Skill 的依据。务必写对。

**最小必需格式**：

```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

这就足以开始了。

**字段要求**：

- **name（必需）**
  - 只能用 kebab-case
  - 无空格、无大写
  - 应与文件夹名匹配

- **description（必需）**
  - **必须同时包含两部分**：Skill 做什么 + 何时使用（触发条件）
  - 少于 1024 字符
  - 不含 XML 标签（`<` 或 `>`）
  - 包含用户可能说出的具体任务
  - 如相关，提及文件类型

- **license（可选）**：开源时使用，常见 MIT、Apache-2.0

- **compatibility（可选）**：1–500 字符，说明环境要求（目标产品、所需系统包、网络访问需求等）

- **metadata（可选）**：任何自定义键值对，建议 author、version、mcp-server，例如：

```yaml
metadata:
  author: ProjectHub
  version: 1.0.0
  mcp-server: projecthub
```

**安全限制**：

frontmatter 中禁止：
- XML 尖括号（`< >`）
- 名称中包含 "claude" 或 "anthropic"（保留字）

原因：frontmatter 出现在 Claude 的系统提示中，恶意内容可能注入指令。

### 编写有效的 Skill

#### description 字段

> Anthropic 工程博客：「这些元数据提供刚好足够的信息，让 Claude 知道何时使用该 Skill，而无需把所有内容加载进上下文。」这是渐进式披露的第一级。

**结构**：`[做什么] + [何时使用] + [关键能力]`

**好的 description 示例**：

```yaml
# Good - 具体且可操作
description: Analyzes Figma design files and generates developer handoff documentation. Use when user uploads .fig files, asks for "design specs", "component documentation", or "design-to-code handoff".

# Good - 包含触发短语
description: Manages Linear project workflows including sprint planning, task creation, and status tracking. Use when user mentions "sprint", "Linear tasks", "project planning", or asks to "create tickets".

# Good - 清晰的价值主张
description: End-to-end customer onboarding workflow for PayFlow. Handles account creation, payment setup, and subscription management. Use when user says "onboard new customer", "set up subscription", or "create PayFlow account".
```

**不好的 description 示例**：

```yaml
# Too vague（太模糊）
description: Helps with projects.

# Missing triggers（缺少触发条件）
description: Creates sophisticated multi-page documentation systems.

# Too technical, no user triggers（太技术化，无用户触发短语）
description: Implements the Project entity model with hierarchical relationships.
```

#### 编写主体指令

frontmatter 之后，用 Markdown 写实际指令。

**推荐结构模板**：

```markdown
---
name: your-skill
description: [..]
---

# Your Skill Name

# Instructions

## Step 1: [First Major Step]
Clear explanation of what happens.

Example:
\`\`\`bash
python scripts/fetch_data.py --project-id PROJECT_ID
\`\`\`
Expected output: [describe what success looks like]

(Add more steps as needed)

## Examples

### Example 1: [common scenario]
User says: "Set up a new marketing campaign"
Actions:
1. Fetch existing campaigns via MCP
2. Create new campaign with provided parameters
Result: Campaign created with confirmation link

## Troubleshooting

### Error: [Common error message]
Cause: [Why it happens]
Solution: [How to fix]
```

#### 指令编写最佳实践

**1. 具体且可操作**

✅ 好：
```
Run `python scripts/validate.py --input {filename}` to check data format.
If validation fails, common issues include:
- Missing required fields (add them to the CSV)
- Invalid date formats (use YYYY-MM-DD)
```

❌ 差：
```
Validate the data before proceeding.
```

**2. 包含错误处理**

```markdown
# Common Issues

## MCP Connection Failed
If you see "Connection refused":
1. Verify MCP server is running: Check Settings > Extensions
2. Confirm API key is valid
3. Try reconnecting: Settings > Extensions > [Your Service] > Reconnect
```

**3. 清晰引用打包资源**

```
Before writing queries, consult `references/api-patterns.md` for:
- Rate limiting guidance
- Pagination patterns
- Error codes and handling
```

**4. 使用渐进式披露**

让 SKILL.md 专注于核心指令。把详细文档移到 `references/` 并链接过去。

---

## 第 3 章 测试与迭代（Testing and iteration）

Skill 可以按不同严谨程度测试：

| 测试方式 | 说明 |
|----------|------|
| 在 Claude.ai 中手动测试 | 直接运行查询并观察行为。快速迭代，无需设置 |
| 在 Claude Code 中脚本化测试 | 自动化测试用例，跨更改可重复验证 |
| 通过 skills API 编程测试 | 构建评测套件，对定义的测试集系统运行 |

根据质量要求和 Skill 的可见性选择方法。小团队内部使用的 Skill 与部署给数千企业用户的 Skill，测试需求不同。

> **Pro Tip：先在单个任务上迭代，再扩展**
> 最有效的 Skill 创建者会先在一个有挑战性的任务上反复迭代直到 Claude 成功，然后把成功的做法提取成 Skill。这利用了 Claude 的上下文学习能力，比广泛测试提供更快的信号。有了可用基础后，再扩展到多个测试用例覆盖。

### 推荐的测试方法

有效的 Skill 测试通常覆盖三个方面：

#### 1. 触发测试（Triggering tests）

**目标**：确保 Skill 在正确时机加载。

**测试用例**：
- ✅ 在明显任务上触发
- ✅ 在改述的请求上触发
- ❌ 在不相关主题上不触发

**示例测试集**：

*应该触发*：
- "Help me set up a new ProjectHub workspace"
- "I need to create a project in ProjectHub"
- "Initialize a ProjectHub project for Q4 planning"

*不应该触发*：
- "What's the weather in San Francisco?"
- "Help me write Python code"
- "Create a spreadsheet"（除非 ProjectHub skill 处理表格）

#### 2. 功能测试（Functional tests）

**目标**：验证 Skill 产生正确输出。

**测试用例**：有效的输出生成、API 调用成功、错误处理生效、边界情况覆盖。

**示例**：
```
Test: Create project with 5 tasks
Given: Project name "Q4 Planning", 5 task descriptions
When: Skill executes workflow
Then:
- Project created in ProjectHub
- 5 tasks created with correct properties
- All tasks linked to project
- No API errors
```

#### 3. 性能对比（Performance comparison）

**目标**：证明 Skill 相比基线改善了结果。使用「定义成功标准」中的指标。

**基线对比示例**：

| 无 Skill | 有 Skill |
|----------|----------|
| 用户每次提供指令 | 自动执行工作流 |
| 15 轮来回消息 | 仅 2 个澄清问题 |
| 3 次失败的 API 调用需要重试 | 0 次失败的 API 调用 |
| 消耗 12,000 tokens | 消耗 6,000 tokens |

### 使用 skill-creator skill

skill-creator 可通过 Claude.ai 的插件目录获取，或为 Claude Code 下载，帮助你构建和迭代 Skill。如果你有 MCP server 并且知道最重要的 2–3 个工作流，通常能在一次工作时段内（15–30 分钟）构建并测试一个可用 Skill。

**创建 Skill**：
- 从自然语言描述生成 Skill
- 产出格式正确的 SKILL.md（含 frontmatter）
- 建议触发短语和结构

**审查 Skill**：
- 标记常见问题（模糊的 description、缺少触发条件、结构问题）
- 识别潜在的过度/不足触发风险
- 基于 Skill 声明的目的建议测试用例

**迭代改进**：
- 使用 Skill 遇到边界情况或失败后，把这些例子带回 skill-creator
- 示例："Use the issues & solution identified in this chat to improve how the skill handles [specific edge case]"

**用法**：
```
"Use the skill-creator skill to help me build a skill for [your use case]"
```

> 注意：skill-creator 帮助你设计和优化 Skill，但不执行自动化测试套件，也不产出定量评估结果。

### 基于反馈的迭代

Skill 是活文档。根据以下信号迭代：

**触发不足（Undertriggering）信号**：
- Skill 在该加载时不加载
- 用户手动启用它
- 关于何时使用的支持问题

→ **解决**：给 description 添加更多细节和细微差别，可能包括关键词（尤其是技术术语）

**过度触发（Overtriggering）信号**：
- Skill 在不相关的查询上加载
- 用户禁用它
- 对其用途感到困惑

→ **解决**：添加反向触发条件（negative triggers），更具体

**执行问题**：
- 结果不一致
- API 调用失败
- 需要用户纠正

→ **解决**：改进指令，添加错误处理

---

## 第 4 章 分发与共享（Distribution and sharing）

Skill 让你的 MCP 集成更完整。当用户比较连接器时，带 Skill 的能更快提供价值，相比纯 MCP 方案形成优势。

### 当前分发模式（2026 年 1 月）

**个人用户如何获取 Skill**：
1. 下载 Skill 文件夹
2. 按需压缩文件夹
3. 通过 Settings > Capabilities > Skills 上传到 Claude.ai
4. 或放到 Claude Code 的 skills 目录

**组织级 Skill**：
- 管理员可在工作区范围部署 Skill（2025-12-18 上线）
- 自动更新
- 集中管理

### 开放标准

Anthropic 已将 Agent Skills 发布为开放标准。与 MCP 一样，Skill 应可跨工具和平台移植——同一个 Skill 无论用 Claude 还是其他 AI 平台都应可用。也有部分 Skill 专为特定平台能力设计，作者可在 compatibility 字段中注明。Anthropic 正与生态成员合作推进该标准。

### 通过 API 使用 Skill

对编程场景（构建利用 Skill 的应用、agent 或自动化工作流），API 提供对 Skill 管理和执行的直接控制。

**关键能力**：
- `/v1/skills` 端点用于列出和管理 Skill
- 通过 `container.skills` 参数把 Skill 加到 Messages API 请求
- 通过 Claude Console 进行版本控制和管理
- 与 Claude Agent SDK 配合构建自定义 agent

**何时用 API vs. Claude.ai**：

| 用例 | 最佳平台 |
|------|----------|
| 最终用户直接与 Skill 交互 | Claude.ai / Claude Code |
| 开发中的手动测试和迭代 | Claude.ai / Claude Code |
| 单独的、临时的工作流 | Claude.ai / Claude Code |
| 以编程方式使用 Skill 的应用 | API |
| 大规模生产部署 | API |
| 自动化流水线和 agent 系统 | API |

> 注意：API 中的 Skill 需要 Code Execution Tool beta，它提供 Skill 运行所需的安全环境。

实现细节参考：Skills API Quickstart、Create Custom skills、Skills in the Agent SDK。

### 当前推荐做法

先把 Skill 托管在 GitHub 公共仓库，配清晰的 README（给人类访客——这与 Skill 文件夹是分开的，Skill 文件夹内不应含 README.md）和带截图的示例用法。然后在 MCP 文档中添加一节链接到 Skill，解释两者结合使用的价值，并提供快速入门指南。

**1. 托管在 GitHub** —— 开源 Skill 用公共仓库、清晰的含安装说明的 README、示例用法和截图
**2. 在你的 MCP 仓库中文档化** —— 从 MCP 文档链接到 Skill、解释结合使用的价值、提供快速入门
**3. 创建安装指南**（示例）：

```markdown
# Installing the [Your Service] skill

1. Download the skill:
   - Clone repo: `git clone https://github.com/yourcompany/skills`
   - Or download ZIP from Releases

2. Install in Claude:
   - Open Claude.ai > Settings > skills
   - Click "Upload skill"
   - Select the skill folder (zipped)

3. Enable the skill:
   - Toggle on the [Your Service] skill
   - Ensure your MCP server is connected

4. Test:
   - Ask Claude: "Set up a new project in [Your Service]"
```

### 定位你的 Skill

你如何描述 Skill 决定了用户是否理解其价值并愿意尝试。在 README、文档或营销材料中，遵循这些原则：

**聚焦结果，而非特性**：

✅ 好：
> "The ProjectHub skill enables teams to set up complete project workspaces in seconds — including pages, databases, and templates — instead of spending 30 minutes on manual setup."

❌ 差：
> "The ProjectHub skill is a folder containing YAML frontmatter and Markdown instructions that calls our MCP server tools."

**突出 MCP + Skills 的故事**：

> "Our MCP server gives Claude access to your Linear projects. Our skills teach Claude your team's sprint planning workflow. Together, they enable AI-powered project management."

---

## 第 5 章 模式与故障排查（Patterns and troubleshooting）

以下模式来自早期采用者和内部团队创建的 Skill，代表常见且有效的做法，而非强制模板。

### 选择你的方法：问题优先 vs. 工具优先

类比 Home Depot（家得宝）：你可能带着问题进门（「我需要修厨房橱柜」），员工指引你到正确工具；或者你挑了一个新电钻，问如何用它完成你的工作。Skill 同理：

- **问题优先（Problem-first）**：「我需要建立项目工作区」→ 你的 Skill 按正确顺序编排 MCP 调用。用户描述结果，Skill 处理工具。
- **工具优先（Tool-first）**：「我连接了 Notion MCP」→ 你的 Skill 教 Claude 最优工作流和最佳实践。用户有访问权，Skill 提供专业知识。

多数 Skill 偏向某一方向。知道哪种框架适合你的用例，有助于选择下面的模式。

### 模式 1：顺序工作流编排（Sequential workflow orchestration）

**使用时机**：用户需要按特定顺序执行多步骤流程。

**示例结构**：

```markdown
# Workflow: Onboard New Customer

## Step 1: Create Account
Call MCP tool: `create_customer`
Parameters: name, email, company

## Step 2: Setup Payment
Call MCP tool: `setup_payment_method`
Wait for: payment method verification

## Step 3: Create Subscription
Call MCP tool: `create_subscription`
Parameters: plan_id, customer_id (from Step 1)

## Step 4: Send Welcome Email
Call MCP tool: `send_email`
Template: welcome_email_template
```

**关键技术**：明确的步骤顺序、步骤间的依赖、每个阶段的验证、失败时的回滚指令。

### 模式 2：多 MCP 协调（Multi-MCP coordination）

**使用时机**：工作流跨越多个服务。

**示例**：设计到开发的交接

```markdown
## Phase 1: Design Export (Figma MCP)
1. Export design assets from Figma
2. Generate design specifications
3. Create asset manifest

## Phase 2: Asset Storage (Drive MCP)
1. Create project folder in Drive
2. Upload all assets
3. Generate shareable links

## Phase 3: Task Creation (Linear MCP)
1. Create development tasks
2. Attach asset links to tasks
3. Assign to engineering team

## Phase 4: Notification (Slack MCP)
1. Post handoff summary to #engineering
2. Include asset links and task references
```

**关键技术**：清晰的阶段分离、MCP 间的数据传递、进入下一阶段前的验证、集中式错误处理。

### 模式 3：迭代优化（Iterative refinement）

**使用时机**：输出质量随迭代提升。

**示例**：报告生成

```markdown
# Iterative Report Creation

## Initial Draft
1. Fetch data via MCP
2. Generate first draft report
3. Save to temporary file

## Quality Check
1. Run validation script: `scripts/check_report.py`
2. Identify issues:
   - Missing sections
   - Inconsistent formatting
   - Data validation errors

## Refinement Loop
1. Address each identified issue
2. Regenerate affected sections
3. Re-validate
4. Repeat until quality threshold met

## Finalization
1. Apply final formatting
2. Generate summary
3. Save final version
```

**关键技术**：明确的质量标准、迭代改进、验证脚本、知道何时停止迭代。

### 模式 4：上下文感知的工具选择（Context-aware tool selection）

**使用时机**：相同结果，但根据上下文选择不同工具。

**示例**：文件存储

```markdown
# Smart File Storage

## Decision Tree
1. Check file type and size
2. Determine best storage location:
   - Large files (>10MB): Use cloud storage MCP
   - Collaborative docs: Use Notion/Docs MCP
   - Code files: Use GitHub MCP
   - Temporary files: Use local storage

## Execute Storage
Based on decision:
- Call appropriate MCP tool
- Apply service-specific metadata
- Generate access link

## Provide Context to User
Explain why that storage was chosen
```

**关键技术**：清晰的决策标准、回退选项、对选择的透明度。

### 模式 5：领域专有智能（Domain-specific intelligence）

**使用时机**：你的 Skill 提供超越工具访问的专业知识。

**示例**：金融合规

```markdown
# Payment Processing with Compliance

## Before Processing (Compliance Check)
1. Fetch transaction details via MCP
2. Apply compliance rules:
   - Check sanctions lists
   - Verify jurisdiction allowances
   - Assess risk level
3. Document compliance decision

## Processing
IF compliance passed:
- Call payment processing MCP tool
- Apply appropriate fraud checks
- Process transaction
ELSE:
- Flag for review
- Create compliance case

## Audit Trail
- Log all compliance checks
- Record processing decisions
- Generate audit report
```

**关键技术**：嵌入逻辑中的领域专业知识、行动前先合规、全面的文档记录、清晰的治理。

### 故障排查（Troubleshooting）

#### Skill 无法上传

**错误**："Could not find SKILL.md in uploaded folder"
**原因**：文件命名不精确为 SKILL.md
**解决**：重命名为 SKILL.md（区分大小写）；用 `ls -la` 验证显示 SKILL.md

**错误**："Invalid frontmatter"
**原因**：YAML 格式问题
**常见错误**：
```yaml
# Wrong - missing delimiters
name: my-skill
description: Does things

# Wrong - unclosed quotes
name: my-skill
description: "Does things

# Correct
---
name: my-skill
description: Does things
---
```

**错误**："Invalid skill name"
**原因**：名称含空格或大写
```yaml
# Wrong
name: My Cool Skill
# Correct
name: my-cool-skill
```

#### Skill 不触发

**症状**：Skill 从不自动加载
**修复**：修改 description 字段（参见「description 字段」的好/坏示例）。

**快速检查清单**：
- 是否太泛？（"Helps with projects" 不起作用）
- 是否包含用户真正会说的触发短语？
- 如适用，是否提及相关文件类型？

**调试方法**：问 Claude："When would you use the [skill name] skill?" Claude 会引用 description 回应。根据缺失部分调整。

#### Skill 过度触发

**症状**：Skill 在不相关查询上加载

**解决方案**：

1. **添加反向触发条件**：
```yaml
description: Advanced data analysis for CSV files. Use for statistical modeling, regression, clustering. Do NOT use for simple data exploration (use data-viz skill instead).
```

2. **更具体**：
```yaml
# Too broad
description: Processes documents
# More specific
description: Processes PDF legal documents for contract review
```

3. **澄清范围**：
```yaml
description: PayFlow payment processing for e-commerce. Use specifically for online payment workflows, not for general financial queries.
```

#### MCP 连接问题

**症状**：Skill 加载了，但 MCP 调用失败

**检查清单**：
1. 验证 MCP server 已连接 —— Claude.ai：Settings > Extensions > [Your Service]，应显示 "Connected"
2. 检查认证 —— API key 有效且未过期、授予了正确的权限/范围、OAuth token 已刷新
3. 独立测试 MCP —— 让 Claude 直接调用 MCP（不用 Skill）："Use [Service] MCP to fetch my projects"；如果这也失败，问题在 MCP 而非 Skill
4. 验证工具名称 —— Skill 引用了正确的 MCP 工具名、检查 MCP server 文档、工具名区分大小写

#### 指令未被遵循

**症状**：Skill 加载了，但 Claude 不遵循指令

**常见原因**：

1. **指令太冗长** —— 保持简洁、用项目符号和编号列表、把详细参考移到单独文件
2. **指令被埋没** —— 把关键指令放顶部、用 `## Important` 或 `## Critical` 标题、必要时重复关键点
3. **语言含糊**：
```markdown
# Bad
Make sure to validate things properly

# Good
CRITICAL: Before calling create_project, verify:
- Project name is non-empty
- At least one team member assigned
- Start date is not in the past
```

> **高级技巧**：对关键验证，考虑打包一个以编程方式执行检查的脚本，而非依赖语言指令。代码是确定性的，语言解释不是。参见 Office skills 中的示例。

4. **模型「偷懒」** —— 添加明确鼓励：
```markdown
# Performance Notes
- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps
```
> 注意：把这段加到用户提示词中比加到 SKILL.md 更有效。

#### 大上下文问题

**症状**：Skill 似乎很慢或响应质量下降

**原因**：Skill 内容过大、同时启用的 Skill 太多、所有内容都被加载而非渐进式披露

**解决方案**：
1. **优化 SKILL.md 大小** —— 把详细文档移到 references/、用链接代替内联、SKILL.md 保持在 5000 词以内
2. **减少启用的 Skill** —— 评估是否同时启用了超过 20–50 个 Skill、建议选择性启用、考虑把相关能力做成 Skill「包」

---

## 第 6 章 资源与参考（Resources and references）

如果你是第一次构建 Skill，先从最佳实践指南开始，然后按需参考 API 文档。

### 官方文档

**Anthropic 资源**：
- Best Practices Guide
- Skills Documentation
- API Reference
- MCP Documentation

**博客文章**：
- Introducing Agent Skills
- Engineering Blog: Equipping Agents for the Real World
- Skills Explained
- How to Create Skills for Claude
- Building Skills for Claude Code
- Improving Frontend Design through Skills

### 示例 Skill

**公共 Skill 仓库**：GitHub `anthropics/skills`，包含 Anthropic 创建的可定制 Skill。

### 工具与实用程序

**skill-creator skill**：
- 内置于 Claude.ai，Claude Code 也可用
- 可从描述生成 Skill
- 审查并给出改进建议
- 用法："Help me build a skill using skill-creator"

**验证**：
- skill-creator 可以评估你的 Skill
- 问："Review this skill and suggest improvements"

### 获取支持

- **技术问题**：Claude Developers Discord 社区论坛
- **Bug 报告**：GitHub Issues `anthropics/skills/issues`，需包含 Skill 名称、错误消息、复现步骤

---

## 附录 A 快速清单

在上传前后用此清单验证你的 Skill。如果想更快开始，先用 skill-creator 生成初稿，再逐项核对。

### 开始之前
- [ ] 确定了 2–3 个具体用例
- [ ] 确定了所需工具（内置或 MCP）
- [ ] 阅读了本指南和示例 Skill
- [ ] 规划了文件夹结构

### 开发过程中
- [ ] 文件夹用 kebab-case 命名
- [ ] SKILL.md 文件存在（拼写精确）
- [ ] YAML frontmatter 有 `---` 分隔符
- [ ] name 字段：kebab-case，无空格，无大写
- [ ] description 包含 WHAT（做什么）和 WHEN（何时用）
- [ ] 任何地方都没有 XML 标签（`< >`）
- [ ] 指令清晰且可操作
- [ ] 包含错误处理
- [ ] 提供了示例
- [ ] references 清晰链接

### 上传之前
- [ ] 测试了在明显任务上触发
- [ ] 测试了在改述请求上触发
- [ ] 验证在不相关主题上不触发
- [ ] 功能测试通过
- [ ] 工具集成正常（如适用）
- [ ] 压缩为 .zip 文件

### 上传之后
- [ ] 在真实对话中测试
- [ ] 监控触发不足/过度触发
- [ ] 收集用户反馈
- [ ] 迭代 description 和指令
- [ ] 更新 metadata 中的版本号

---

## 附录 B YAML frontmatter

### 必需字段

```yaml
---
name: skill-name-in-kebab-case
description: What it does and when to use it. Include specific trigger phrases.
---
```

### 所有可选字段

```yaml
name: skill-name
description: [required description]
license: MIT  # Optional: License for open-source
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch"  # Optional: Restrict tool access
metadata:  # Optional: Custom fields
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
```

### 安全说明

**允许**：
- 任何标准 YAML 类型（字符串、数字、布尔值、列表、对象）
- 自定义 metadata 字段
- 长 description（最多 1024 字符）

**禁止**：
- XML 尖括号（`< >`）—— 安全限制
- YAML 中的代码执行（使用安全的 YAML 解析）
- 名称以 "claude" 或 "anthropic" 为前缀的 Skill（保留字）

---

## 附录 C 完整技能示例

关于展示本指南模式的、可用于生产环境的完整 Skill：

- **Document Skills** —— PDF、DOCX、PPTX、XLSX 创建
- **Example Skills** —— 各种工作流模式
- **Partner Skills Directory** —— 来自 Asana、Atlassian、Canva、Figma、Sentry、Zapier 等合作伙伴的 Skill

这些仓库保持更新，包含本指南之外的更多示例。克隆它们，按你的用例修改，作为模板使用。

---

> 整理说明：本文档由 PDF 原文整理为结构化中文笔记，保留了全部核心概念、代码示例、清单与表格。关键术语保留英文以便对照原文。原始出处：Anthropic《The Complete Guide to Building Skills for Claude》。
