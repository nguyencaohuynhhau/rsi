# Diataxis Writer Skill

**Language / Ngôn ngữ / 语言:** [English](./README.md) | [Tiếng Việt](./README.vi.md) | [中文](./README.zh.md)

这个技能帮助智能体使用 Diataxis 框架来编写、重构和审阅文档。

## 什么是 Diataxis？

Diataxis 是一种围绕读者真实需求来组织文档的框架。它不是把所有相关信息都塞进同一页，而是把文档分成四种类型：

| 文档类型 | 当读者想要 | 目标 |
| --- | --- | --- |
| Tutorial 教程 | 通过实践学习 | 沿着安全路径引导读者获得初步能力 |
| How-to guide 操作指南 | 完成一个具体任务 | 提供清晰步骤以达成结果 |
| Reference 参考文档 | 查找准确信息 | 提供完整、一致、易扫描的事实 |
| Explanation 解释说明 | 理解背景或原因 | 解释概念、决策、取舍和思维模型 |

Diataxis 的核心思想是：每份文档都应该向读者做出一个清晰承诺。教程不应该试图变成 API 参考文档。操作指南不应该被架构解释拉长。参考文档不应该让读者先读完一段长故事，才能找到需要的字段、选项或命令。

## 优点

- **减少混乱和臃肿的文档**：Diataxis 把学习、执行、查询和理解拆成不同的读者目标。
- **改善读者体验**：读者能找到与当前目标匹配的文档，而不是自己过滤无关内容。
- **更容易审阅质量**：审阅者可以先问“这个页面服务的是哪一种 reader job？”再修改细节。
- **更容易维护**：每个页面都有更清晰的范围和承诺，更新时不容易无限扩散。
- **适用范围不只技术文档**：它也适合 onboarding、知识库、流程文档、手册、runbook、产品文档、API 文档和运营指南。

## 为什么使用 Diataxis？

很多文档让人沮丧，并不是因为信息不够，而是因为把多种目的混在一起。新手需要被一步步引导。正在处理任务的人需要简洁准确的步骤。查资料的人需要表格、字段、默认值和约束。想理解的人需要背景和原因。

Diataxis 要求作者在写作前先选择主要读者目标。这样文档结构会更自然，更容易阅读，也更不容易自相矛盾。

## 什么时候使用这个技能？

在以下场景使用这个技能：

- 按 Diataxis 编写新文档；
- 审阅一份混乱或过长的文档；
- 把一个 “getting started” 页面拆成教程、操作指南、参考文档和解释说明；
- 重新设计知识库或文档站点；
- 编写 onboarding 文档、流程文档、runbook、手册、产品文档或 API 文档。

不要把 Diataxis 机械地套用到营销文案、销售提案、法律合同、新闻稿、小说，或主要用于情感说服的写作中。这些格式可以借用 Diataxis 的部分分类思路，但不应该被强行塞进这个框架。

## 快速用法

在支持 skills 的智能体中，可以这样提出任务：

```text
Review this document with Diataxis and suggest how to split it.
```

```text
Write a getting-started tutorial for this internal tool.
```

```text
Create reference docs for this CLI command and keep them easy to scan.
```

这个技能还包含一个启发式脚本，用于快速判断文档中的 Diataxis 信号：

从技能目录运行：

```bash
bash ./scripts/classify-doc.sh path/to/doc.md
```

这个脚本只适合快速诊断。最终判断仍应基于 reader job 和文档上下文。

## 安装

### 1. 使用 CLI（推荐）

```bash
npx skills add tronghieu/agent-skills --skill diataxis-writer
```

### 2. 手动安装（适合非技术用户）

1. **下载：** 转到此仓库的 `skills/` 文件夹下载 `diataxis-writer.zip` 文件。
2. **解压和复制：** 解压 `diataxis-writer.zip` 并将 `diataxis-writer` 文件夹复制到以下目录之一：

**针对特定项目：**
将 `diataxis-writer` 文件夹复制到项目根目录下的 `.agents/skills/` 或 `.claude/skills/`。

**全局安装（所有项目可用）：**
* **Mac / Linux：** `~/.agents/skills/` 或 `~/.claude/skills/`
* **Windows：** `%USERPROFILE%\.agents\skills\` 或 `%USERPROFILE%\.claude\skills\`（通常为 `C:\Users\<YourUsername>`）
