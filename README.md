<p align="center">
  <img src="./assets/profile-hero.svg" alt="微凉 — AI Application Developer & Builder" width="100%" />
</p>

<h1 align="center">你好，我是微凉 👋</h1>

<h3 align="center">我做的不是“跑一次的 Demo”，而是有人工确认、有规则兜底、可验证的 AI 应用。</h3>

<p align="center">
  Agent 编排 · 多模态检索 · 数据工程 · AI 辅助研发
</p>

<p align="center">
  <a href="https://github.com/weicode703?tab=repositories">Repositories</a>
  ·
  <a href="https://weicode703.github.io/interview-notes/">知识库</a>
</p>

---

## 我关心的，是 AI 能不能在真实业务里跑起来

企业场景里 AI 的瓶颈从来不是模型，而是**数据可靠性与工具边界的可控性**。所以我习惯把 AI 应用当作一个真实软件系统来设计：模型只是其中的理解组件，真正决定它能否交付的，是数据链路、校验规则、人工卡点、失败恢复和上线后的回读。

我的几个项目恰好覆盖了这条完整链路：

```text
house-agent  →  多子图 Agent、人工卡点与反幻觉兜底
AIWear       →  多模态检索、RAG 降级与流式工程
sync engine  →  企业交付的幂等同步与全量校准（代码不公开）
skill        →  把重复流程封装成可复用的自动化
```

---

## 01 / 智居 Agent

### [一个会确认、会兜底的租房对话 Agent](https://github.com/weicode703/house-agent)

> 用户一句口语进来，先结构化成意图和参数，再到推荐、预约、偏好、闲聊四个子图——有真实后果的动作前，一定停下来等人工确认。

| 我的角色 | 项目定位 | 当前状态 | Links |
|---|---|---|---|
| 独立设计并实现 | 多子图对话 Agent | 已上线 | [Repository](https://github.com/weicode703/house-agent) |

### 这个项目里的关键设计

- **单入口多子图**：主图用结构化输出做意图路由，分发到推荐、预约、偏好、闲聊四个子图；长会话状态持久化，结束后清理，断点可恢复。
- **人工卡点（HITL）**：预约这类有真实后果的动作前，子图 `interrupt` 中断、经 gateway 上抛给用户确认，确认后从断点恢复执行——**技术上能全自动和该不该全自动是两回事**。
- **反幻觉双保险**：模型抽取 8 个结构化字段后，正则强校验兜底（城市必须带市/区/县后缀、预算必须数字+单位+方向词），任何一条不过就打回用户重新说——宁可多问一句，不可错写一条。
- **SQL 审核节点**：AI 生成的查询先过独立的 `check_query` 审核节点，拦截 NOT IN 遇 NULL、JOIN 类型不匹配这类坑，通过才执行。

**Core Stack**

`Python` `LangGraph` `LangChain` `MySQL` `Redis` `Docker`

---

## 02 / AIWear

### [以图搜衣 + 穿搭问答的多模态应用](https://github.com/weicode703/AIWear)

> CLIP 把衣服编码成向量做检索，Qwen-VL 理解图像语义，规则兜底——三路互补保证永远有结果。

| 我的角色 | 项目定位 | 当前状态 | Links |
|---|---|---|---|
| 独立开发并部署 | 多模态检索 + RAG 应用 | 已上线 | [Repository](https://github.com/weicode703/AIWear) |

### 这个项目里的关键设计

- **向量检索**：CLIP-ViT-Base-Patch16 把服装图编码成 512 维向量存入 Redis，以图搜衣 Top-5 相似推荐。
- **三路互补**：Qwen-VL 负责图像语义理解，零样本分类与关键词规则兜底，任何一路失效都有替补。
- **RAG 三级降级**：主模型异常自动切换 OpenAI 兼容接口，再降级本地能力——**问答服务永不 502**；答案带来源引用抑制幻觉。
- **流式与鉴权**：SSE 自定义五事件协议（start/progress/delta/done/error）；JWT + Redis 黑名单实现登出即时生效。
- **交付形态**：Docker Compose 一键编排应用、数据库与缓存。

**Core Stack**

`Python` `FastAPI` `CLIP` `Qwen-VL` `Redis` `SSE` `Docker Compose`

---

## 03 / 企业交付：数据同步引擎与 AI 数据底座

### 把 98 项周期任务搬进系统，一条变更都不能丢

> 实习期间参与集团型企业数字化项目：对接 16 位负责人，把 98 项周期汇报任务梳理成结构化配置并自动流转——任务变更 1 分钟内自动写入钉钉。

| 项目形态 | 我的角色 | 当前状态 | Links |
|---|---|---|---|
| 企业级数据链路 | 开发成员，负责数据接入与同步服务 | 集团在用 | 企业项目 · 代码不公开 |

### 这个项目里的关键设计

- **规则与执行分离**：98 项任务全部落成“周期、负责人、数据来源、异常处理”结构化配置，变更只改配置不改代码——快在架构，不在性能。
- **同步引擎**：纯 Python 标准库实现（http.server + sqlite3 + hmac），自实现 HS256 JWT 与 Webhook 验签；事务去重保证幂等，消费失败指数退避重试（60 秒→1 小时，最多 8 次），超限进死信队列，每日 03:00 全量校准对账兜底。
- **平台的限制即需求**：钉钉同步表禁止第三方建行，设计“预置等待行 + 填充复用”方案，完全在平台规则内运行——先弄清边界在哪，再设计流程。
- **AI 数据底座**：为 Agent 工具调用提供准确实时的数据层——Agent 输出的上限，取决于它调到的数据质量。

**Core Stack**

`Python 标准库` `SQLite` `Webhook` `HMAC / HS256` `钉钉开放平台`

---

## 04 / 工具与知识库

| 项目 | 一句话 | Links |
|---|---|---|
| **面试八股知识库** | 按技术栈自建 40+ 篇题库站，frontmatter 驱动刷题池，审计脚本自动查格式与断链，CI 自动发布 | [在线 ↗](https://weicode703.github.io/interview-notes/) |
| **校招邮件投递助手** | 把重复的邮件投递流程封装成 Skill：岗位截图 + 邮箱截图 → AI 生成正文 → 粘贴发送 | [Repository ↗](https://github.com/weicode703/campus-apply-email-skill) |

---

## 这些项目背后的统一方法

| 原则 | 我的工程取向 |
|---|---|
| **模型负责理解，规则负责兜底** | 口语理解交给模型，每个 AI 环节后面跟一个确定性校验层。 |
| **宁可多问一句，不可错写一条** | AI 输出不确定时打回确认，而不是猜。 |
| **AI 出活，验证体系保质量** | 给上下文、拆任务、约束输出、机器检查、错误回写——五步缺一不可。 |
| **恢复优先于根因** | 先止血再破案，结论落进监控和配置防止复发。 |
| **上线不是结束** | 配置存在 ≠ 校验通过 ≠ 浏览器走通 ≠ 真实用户在用——流程真的在跑才算交付。 |

## 技术版图

| Layer | Technologies | What I build |
|---|---|---|
| **Agent 编排** | LangGraph, LangChain, Structured Output, Interrupt | 意图路由、多子图、人工卡点与断点恢复 |
| **检索与多模态** | CLIP, Qwen-VL, Redis Vector, RAG | 以图搜衣、三级降级、来源引用 |
| **后端** | Python, FastAPI, JWT, SSE | API、鉴权、流式输出与错误协议 |
| **数据** | MySQL, SQLite, Redis, 全量校准 | 幂等同步、退避重试、死信与对账 |
| **交付** | Docker Compose, Nginx, Linux, GitHub Actions | 编排部署、自动构建与发布 |
| **AI 工具流** | Claude Code, Codex, Skill | 上下文工程、任务拆解、机器验收 |

## 现在仍在推进

- 持续打磨 Agent 的人工卡点与反幻觉校验模式
- 深入 RAG 可靠性与评测方法
- 把更多重复流程沉淀成 Skill 和自动化脚本
- 在真实交付里验证“数据可靠性优先”这条路

---

<h3 align="center">让 AI 真正跑在业务里。</h3>

<p align="center">
  会确认 · 会兜底 · 可验证
</p>

<p align="center">
  <a href="https://github.com/weicode703?tab=repositories">Explore my repositories</a>
  ·
  <a href="https://weicode703.github.io/interview-notes/">Read my notes</a>
</p>
