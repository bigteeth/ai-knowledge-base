# AGENTS.md

## 项目概述

本项目是一个面向 AI/LLM/Agent 领域的 AI 知识库助手，负责自动从 GitHub Trending 与 Hacker News 采集技术动态，经过 AI 分析与结构化处理后沉淀为标准化 JSON 知识条目，并支持进一步向 Telegram、飞书等渠道分发，形成可持续更新、可检索、可复用的技术情报库。

## 技术栈

- Python 3.12
- OpenCode + 国产大模型
- LangGraph
- OpenClaw

## 编码规范

- 遵循 PEP 8。
- 变量、函数、模块文件名统一使用 `snake_case`。
- 公共函数、类、模块优先使用 Google 风格 docstring。
- 禁止使用裸 `print()` 进行调试或业务输出，统一使用日志组件。
- 优先实现最小正确改动，避免无必要的抽象和过度封装。
- 新增代码必须可读、可测、可维护，避免隐藏副作用。

## 项目结构

```text
.
├── AGENTS.md
├── .opencode/
│   ├── agents/
│   └── skills/
└── knowledge/
    ├── raw/
    └── artic/
```

### 目录说明

- `.opencode/agents/`：Agent 定义、提示词、编排配置。
- `.opencode/skills/`：可复用技能、工具说明、外部系统操作封装。
- `knowledge/raw/`：原始采集内容，保留来源上下文，原则上不覆盖原始信息。
- `knowledge/artic/`：AI 清洗、归纳、结构化后的知识条目、专题卡片或分发素材。

## 知识条目 JSON 格式

所有结构化知识条目应尽量遵循统一 JSON Schema，至少包含以下字段：

```json
{
  "id": "gh_20260422_openclaw_agent_loop",
  "title": "OpenClaw adds multi-agent execution support",
  "source": "github_trending",
  "source_url": "https://github.com/example/repo",
  "author": "example-author",
  "published_at": "2026-04-22T08:30:00Z",
  "collected_at": "2026-04-22T09:00:00Z",
  "summary": "A short summary of the update and why it matters.",
  "content": "Detailed normalized content extracted from the source.",
  "tags": ["ai", "llm", "agent", "tooling"],
  "status": "analyzed",
  "score": 0.91,
  "language": "en",
  "distribution_channels": ["telegram", "feishu"],
  "references": [
    "https://news.ycombinator.com/item?id=123456"
  ],
  "metadata": {
    "model": "domestic-llm",
    "workflow": "langgraph_pipeline",
    "version": "v1"
  }
}
```

### 字段约定

- `id`：全局唯一 ID，建议使用 `来源_日期_主题` 形式。
- `title`：知识条目标题。
- `source`：来源类型，例如 `github_trending`、`hacker_news`。
- `source_url`：原始链接。
- `summary`：AI 输出的简明摘要。
- `tags`：主题标签，面向检索和分发筛选。
- `status`：处理状态，建议使用 `raw`、`analyzed`、`reviewed`、`published`。
- `distribution_channels`：计划或已投放渠道，例如 `telegram`、`feishu`。
- `metadata`：模型、工作流版本、附加上下文。

## Agent 角色概览

| 角色 | 主要职责 | 输入 | 输出 |
| --- | --- | --- | --- |
| 采集 Agent | 从 GitHub Trending、Hacker News 抓取 AI/LLM/Agent 相关动态，做基础过滤与去重 | 外部站点页面、链接、时间窗口 | 原始内容、候选条目、采集元数据 |
| 分析 Agent | 调用模型进行摘要、标签提取、相关性判断、价值评分与标准化字段生成 | 原始内容、来源信息、上下文提示 | 结构化 JSON、摘要、标签、状态 |
| 整理 Agent | 将分析结果落盘、归档、补充索引，并准备 Telegram/飞书分发内容 | 结构化 JSON、分发规则、模板 | 知识库文件、可分发文案、发布记录 |

## 协作要求

- 优先保留原始数据，再生成加工结果，不得直接用分析结果覆盖原始采集内容。
- 新增字段前先确认是否已有等价字段，避免结构漂移。
- 所有外部采集逻辑必须考虑幂等、去重、失败重试和速率限制。
- 分发逻辑必须与采集、分析逻辑解耦，避免单点失败影响整条链路。
- 引入新模型、新渠道或新工作流时，必须补充最小可用文档说明。

## 红线

以下操作绝对禁止：

- 禁止提交密钥、Token、Cookie、Webhook、账号凭据等任何敏感信息。
- 禁止编造不存在的项目、链接、来源、指标、结论或知识数据。
- 禁止在未审核的情况下批量覆盖或删除 `knowledge/raw/` 中的原始数据。
- 禁止使用裸 `print()`、临时调试代码、硬编码密钥或硬编码生产地址进入正式代码。
- 禁止在日志中输出 API Key、Token、Cookie 或其他敏感信息。
- 禁止绕过失败处理、重试控制、去重逻辑，直接重复写入知识条目。
- 禁止执行 `rm -rf`、危险删除、强制覆盖或其他高风险破坏性命令。
- 禁止未经确认向真实 Telegram 群组、飞书群组发送测试消息。
- 禁止在没有来源链接或来源不可信的情况下生成“已确认”知识条目。
- 禁止修改 `AGENTS.md` 本身，除非用户明确提出要求。
- 禁止随意变更 JSON 主结构，导致下游分析、索引或分发流程失效。
