# ✨ Bugment 增强特性详解

本文档详细说明 Bugment 相比原生 Augment 的三大增强能力。

---

## 🖼️ 一、图片理解增强

### 原生 Augment 的不足

原生 Augment 对截图的处理较为基础，通常只能做简单的图片识别，无法深入理解 UI 结构和布局细节，在编程场景中帮助有限。

### Bugment 的增强

Bugment 对图片处理进行了**专门的编程场景优化**。当你在对话中发送截图时，会自动生成结构化的 **📷 Image Context** 分析报告，包含以下四个维度：

#### 1. 场景描述（Scene）
自动识别截图中的界面状态和组件结构。例如：
> "分组栏展开状态，显示系统分组（全部、未读、@我等）、标签分组（标签分组…、标签分组6等）、以及分组内的子标签"

#### 2. 可见文本提取（Visible Text）
精确提取界面中的所有文字内容，包括按钮文字、标签名称、数字统计等：
> "全部 99+、未读 99+、@我、加急、单聊 99+、群聊 99+、Pin 22、标签分组...99+、标签14、标签7..."

#### 3. UI 结构分析（UI/Structure Details）
深度分析界面的布局细节：
- 系统分组图标（圆形消息、未读等）在左侧对齐
- 标签分组有展开箭头 ▶，箭头在最左边，然后是文件夹图标，然后是名称
- 标签（标签14等）有标签图标，但图标位置和系统分组的图标没有左对齐
- 分组内的子标签图标和顶层标签图标在同一水平位置，没有额外缩进

#### 4. 问题证据推理（Evidence Relevant to User Question）
结合用户的提问，自动定位截图中与问题相关的关键线索：
1. 顶层节点的图标没有左对齐：系统分组图标、标签分组图标、独立标签图标的左边缘不在同一垂直线上
2. 分组内子标签的图标没有比分组图标向右偏移 6px
3. 标签选中时图标颜色未变蓝

### 典型使用场景

```
用户：发送 UI 截图 + "看一下拖拽时有几个问题"
  ↓
Bugment：自动分析生成 Image Context
  ↓
AI：精确定位到 TagItem.vue 的 CSS 问题 → 给出具体修复代码
```

**特别适合：** 前端 CSS 调试、布局修复、UI 还原、组件对齐问题排查。

---

## 🧠 二、提示词增强

Bugment 对对话中的系统提示词进行了深度优化，在原生 Augment 的基础上注入了一套**面向工程实践的行为规范**，使 AI 在编程场景下的理解、执行和协作质量显著提升。

### 原生 Augment 的不足

原生 Augment 的系统提示词偏通用，缺少对编程工作流的结构化约束：
- 不会主动做计划就直接改代码
- 修 bug 容易在下游打补丁而非定位根因
- 缺少验证环节，改完不测试
- 长任务容易丢失上下文，前后不一致

### Bugment 注入的增强指令

Bugment 在每次对话中自动注入以下六个维度的行为增强：

#### 1. Bug 修复纪律（Bug Fixing Discipline）
```
优先在上游修复根因，而非在下游打补丁。
修改前必须先定位根因。
避免过度工程——能一行改好的就别写十行。
对专业代码库，先仔细确认 bug 位置再动手。
添加回归测试，但实现保持最小化。
```
**效果**：避免 AI 盲目改代码，杜绝"头痛医脚"式的修复。

#### 2. 规划节奏（Planning Cadence）
```
非简单任务必须先拟定简洁计划。
同一时间只推进一个步骤。
发现新约束或新信息后必须刷新计划。
```
**效果**：AI 会先列出 TODO，逐步推进，而不是一口气胡乱改一堆文件。

#### 3. 测试纪律（Testing Discipline）
```
重大实现改动前必须先设计或更新测试。
未经明确指示，绝不删除或弱化已有测试。
无法直接运行时，提供可复制粘贴的验证命令。
```
**效果**：代码修改后自动跑测试验证，而非改完就宣布"完成"。

#### 4. 验证工具优先（Verification Tools）
```
优先使用已有的自动化验证手段（如 Playwright、单元测试）来确认工作成果。
工具不可用时，提供可直接执行的验证命令给用户。
```
**效果**：每一步改动都有验证闭环，不会出现"看起来改了但没测过"的情况。

#### 5. 长线程工作流（Long-horizon Workflow）
```
跨多轮对话的工作，使用轻量级工作区笔记（如 progress.txt）
记录待办测试清单，避免重复劳动。
仅在确实能避免返工时才创建笔记文件。
```
**效果**：跨会话长任务不丢上下文，每次恢复工作都知道做到哪了。

#### 6. 进度笔记（Progress Notes）
```
优先使用轻量工作区产物，而非长篇聊天回顾。
除非必要，不创建重复的 .md 文件或过度文档。
```
**效果**：保持工作区干净，信息集中，不会生成大量无用的 markdown 文件。

### 提示词增强工具（Enhance Prompt）

除系统指令外，Bugment 还内置了一个**提示词增强 MCP 工具**，用户可以在消息中添加 `-enhance` 标记来触发：

```
用户输入：新加一个登录页面 -enhance
         ↓
Bugment 自动分析：
  - 读取项目代码结构和技术栈
  - 回溯最近 5-10 轮对话历史
  - 理解当前工作上下文
         ↓
输出增强后的精准提示词：
  "在 src/pages/ 下新建 LoginPage.tsx，使用项目已有的
   React + TailwindCSS 技术栈，参考 RegisterPage 的布局结构，
   包含用户名/密码表单、记住我选项、忘记密码链接，
   调用 src/services/auth.ts 中的 login API..."
```

**触发方式**：在消息任意位置添加 `-enhance`、`-enhancer`、`-Enhance`、`-Enhancer`

### 对比总结

| 维度 | 原生 Augment | Bugment |
|------|-------------|---------|
| **修 bug** | 可能在下游打补丁 | 强制定位根因，最小化修改 |
| **做计划** | 直接开始写代码 | 先列 TODO，逐步推进 |
| **测试** | 改完不一定测 | 改动后必须验证 |
| **长任务** | 容易丢上下文 | 自动维护进度笔记 |
| **提示词** | 用户写什么就执行什么 | 自动增强为精准、可执行的指令 |
| **代码风格** | 可能加无关注释 | 不加不删注释，保持原有风格 |

---

## 🛠️ 三、Skills 技能调用

### 什么是 Skills

Skills 是一组专业化的指令集，AI 会根据对话内容**自动匹配**并加载对应技能。当技能被激活时，AI 获得该领域的专业知识和最佳实践，显著提升任务完成质量。

### 内置技能列表

| 技能 | 触发条件 | 说明 |
|------|----------|------|
| **claude-api** | 代码中导入 `anthropic`、`@anthropic-ai/sdk` 或用户要求使用 Claude API | 帮助构建基于 Claude API 或 Anthropic SDK 的应用程序 |
| **docx** | 用户要求创建、读取、编辑或操作 Word 文档（.docx 文件） | 支持内容提取、查找替换、批注等操作 |
| **frontend-design** | 用户要求构建网页组件、页面、海报、应用界面等前端内容 | 生成高质量、有设计感的前端代码和 UI，避免千篇一律的 AI 风格 |
| **pptx** | 用户需要创建、编辑或分析演示文稿（.pptx 文件） | 支持新建演示文稿、修改内容、处理布局、添加演讲者备注等 |
| **screenshot** | 用户明确要求进行桌面或系统截图（全屏、特定应用窗口或像素区域） | 进行系统级别的屏幕截图 |
| **template-skill** | 模板技能（预留扩展位） | 预留的技能模板插槽 |
| **xlsx** | 用户需要打开、读取、编辑、创建电子表格文件（.xlsx / .xlsm / .csv / .tsv） | 支持公式计算、格式化、图表、数据清洗、格式转换等表格操作 |

### 如何安装 Skills

Skills 文件存放在用户 home 目录下，所有 IDE 中的 Augment 插件共享：

```bash
# Skills 存放路径（二选一，推荐同时配置）
~/.claude/skills/      # Claude Code 兼容路径
~/.augment/skills/     # Augment 专用路径
```

#### 方式一：手动创建（推荐）

每个 skill 是一个目录，包含一个 `SKILL.md` 文件：

```
~/.claude/skills/
├── claude-api/
│   └── SKILL.md
├── docx/
│   └── SKILL.md
├── frontend-design/
│   └── SKILL.md
├── pptx/
│   └── SKILL.md
├── screenshot/
│   └── SKILL.md
├── template-skill/
│   └── SKILL.md
└── xlsx/
    └── SKILL.md
```

然后创建符号链接到 Augment 路径：

```bash
mkdir -p ~/.augment/skills
for skill in claude-api docx frontend-design pptx screenshot template-skill xlsx; do
  ln -sf ~/.claude/skills/$skill ~/.augment/skills/$skill
done
```

#### 方式二：通过 Claude Code 插件市场安装

如果你安装了 Claude Code CLI，可以直接使用官方插件市场：

```bash
# 添加 Anthropic 官方技能仓库
claude /plugin marketplace add anthropics/skills

# 安装文档类技能（docx/pptx/xlsx）
claude /plugin install document-skills@anthropic-agent-skills

# 安装示例技能
claude /plugin install example-skills@anthropic-agent-skills
```

### 自定义技能

你可以基于 `template-skill` 创建自己的技能：

```bash
mkdir -p ~/.claude/skills/my-skill
```

编辑 `~/.claude/skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: 描述技能的用途和触发条件
---

# 技能名称

在这里写你的技能指令...
```

---

## 📌 总结

| 增强项 | 原生 Augment | Bugment |
|--------|-------------|---------|
| **图片理解** | 基础图片识别 | 结构化 Image Context 分析（场景 + 文本 + UI 结构 + 证据推理） |
| **提示词** | 通用提示词 | 编程场景深度优化 |
| **Skills** | 不支持 | 7 个内置专业技能，自动匹配加载 |
| **模型支持** | 仅官方模型 | 32+ 模型，支持自定义 API 服务 |
