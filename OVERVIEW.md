# 系统提示词设计模式库 — 整库理解

> 本文档是对 system_prompts_leaks/ 语料库（165 个系统提示词）的分析总览。
> 来源涵盖 Anthropic、Google、OpenAI、xAI、Perplexity 及 15+ 其他厂商。
> 后续所有 Skill 均以此为全局上下文。

## 基本信息

- **语料来源**: system_prompts_leaks/（GitHub 开源项目）
- **覆盖厂商**: Anthropic (Claude 全系列), Google (Gemini 全系列), OpenAI (GPT-4o ~ GPT-5.5, Codex, o3/o4), xAI (Grok 3~4.3), Perplexity, Meta AI, Mistral (Le Chat), Notion AI, Warp, Brave 等
- **覆盖产品形态**: Chat 应用、编程 Agent、浏览器 Agent、文档编辑、语音助手、AI 搜索、教育平台、企业 Workspace、设计工具、移动端
- **蒸馏时间**: 2026-05-04

---

## 1. 语料结构

### 类型分布

| 产品形态 | 代表产品 | 数量(约) |
|---|---|---|
| 通用 Chat | Claude.ai, GPT-4o/5.x, Gemini Pro, Grok, Le Chat, Meta AI | ~40 |
| 编程 Agent | Claude Code, Codex, Gemini CLI, Jules, Warp | ~25 |
| 浏览器/计算机 Agent | Claude Chrome, ChatGPT Agent Mode, Fellou | ~5 |
| 文档编辑 | Claude for Word/Excel/PPT, Copilot in Word | ~5 |
| AI 搜索 | Perplexity, Google Search AI Mode, Brave, Kagi | ~5 |
| 语音助手 | Perplexity Voice, GPT-4o Voice, Sesame AI | ~3 |
| 设计工具 | Claude Design, Canvas/Canmore | ~3 |
| 教育平台 | Flint K12, NotebookLM | ~3 |
| 企业 Workspace | Gemini Workspace, Notion AI | ~3 |
| 人格/角色 | GPT-5.1 personalities, Grok Personas | ~15 |
| API 裸模型 | Claude API, GPT API, Gemini API, o3/o4 API | ~15 |
| 工具定义 | OpenAI tools (canvas, image, search, memory, code) | ~10 |

### 一句话主旨

**顶级 AI 产品的系统提示词是一套工程化的行为控制协议，由身份定义、工具集成、安全边界、输出格式、对话流程、上下文管理和个性化七大模块组成。**

---

## 2. 核心设计模式（提取的骨架）

### 模式 1: 分层架构（所有厂商共用）

```
┌─────────────────────────────────────┐
│ Layer 0: 身份与人格 (Identity)        │  谁是 AI + 性格基调
├─────────────────────────────────────┤
│ Layer 1: 行为准则 (Behavioral Rules)  │  做什么/不做什么
├─────────────────────────────────────┤
│ Layer 2: 工具集成 (Tool System)       │  能用什么工具 + 何时用
├─────────────────────────────────────┤
│ Layer 3: 安全防线 (Safety Guardrails) │  底线 + 注入防御
├─────────────────────────────────────┤
│ Layer 4: 输出控制 (Output Control)    │  格式 + 长度 + 风格
├─────────────────────────────────────┤
│ Layer 5: 上下文管理 (Context Mgmt)    │  记忆 + 压缩 + 窗口优化
└─────────────────────────────────────┘
```

### 模式 2: 人格叠加架构

```
base-persona (身份声明)
  + personality-overlay (性格修饰: default/friendly/professional)
  + domain-overlay (领域适配: coding/writing/search)
  + platform-overlay (平台适配: mobile/voice/cli)
```

OpenAI GPT-5.1 系列最清晰地展示了这个模式：基础身份 + 可插拔人格层。

### 模式 3: 工具发现机制

- Claude Opus 4.7: `tool_search` — 主动搜索可用工具
- Claude Code: `ToolSearch` — 按需加载延迟工具
- Gemini 3.1 Pro: 每次 response 前生成 `tool_code` block

### 模式 4: 安全纵深防御

```
Layer 1: 输入验证 (注入检测)
Layer 2: 权限分级 (三级: 禁止/需确认/自动)
Layer 3: 输出过滤 (敏感信息)
Layer 4: 上下文隔离 (记忆/文档内容不信任)
Layer 5: 级联锁定 (一次违规后提升警戒级别)
```

### 模式 5: 自适应输出

- Claude Mobile: 按问题复杂度分级（1-2 句 / 短列表 / 1 屏 / 2 屏）
- Perplexity Voice: "Please talk quickly"
- Claude Code: 极简默认，详细仅在被要求时
- Gemini CLI: 目标少于 3 行文字

---

## 3. 关键术语词典

| 术语 | 行业用法 | 与常识用法的差异 |
|---|---|---|
| System Prompt | 注入到对话开头的系统级指令，定义 AI 的全局行为规则 | 不是"系统提示框"，而是一套完整的**行为控制协议** |
| Tool Gating | 工具的条件激活/禁用机制 | 不是简单开关，而是基于用户意图的**上下文感知路由** |
| Persona Overlay | 可叠加的人格修饰层 | 不是角色扮演 prompt，而是**工程化的性格参数化** |
| Injection Defense | 防御用户/外部内容中的恶意指令 | 不是关键词过滤，而是**多层纵深防御体系** |
| Memory Silencing | 记忆的静默应用规则 | 不是"记住不告诉用户"，而是**禁止任何归因表达的隐形个性化** |
| Citation Format | 引用来源的结构化格式 | 不是学术论文引用，而是**机器可解析的 inline 标记** |
| Context Compression | 上下文窗口的压缩策略 | 不是简单截断，而是**保留关键信息的智能摘要** |
| Action Classification | 操作的分级授权机制 | 不是权限列表，而是**基于风险等级的自动/确认/禁止三层** |
| Channel Architecture | 输出的多通道分离 | 不是多模态输出，而是**隐藏推理/可见过程/最终结果的三层隔离** |
| Agent Delegation | 子代理委派模式 | 不是简单分工，而是**上下文隔离 + 摘要压缩的任务分发** |

---

## 4. 批判（局限与盲点）

### 语料的时代局限
- 这些提示词反映的是 2024-2026 年的设计实践，未来可能演化
- 泄露的提示词可能不是最新版本

### 厂商偏差
- Anthropic 偏重安全与对话质量
- OpenAI 偏重工具集成与多模态
- Google 偏重搜索集成与 Workspace 生态
- xAI 偏重社交媒体数据与言论自由
- 样本中缺少中国厂商（百度、阿里、字节）的系统提示词

### 设计盲区
- 大多数提示词没有显式的 A/B 测试或优化方法论
- 安全策略多为"黑名单"思维，缺少"白名单"或"能力边界"声明
- 个性化系统普遍缺少用户可控性和透明度设计

---

## 5. 可 Skill 化的内容

### 高优先级（跨厂商验证 + 高实用性）

- [x] persona-design — 身份与人格定义模式
- [x] personality-system — 可插拔人格系统设计
- [x] tool-specification — 工具定义与集成模式
- [x] safety-guardrails — 安全防线与伦理边界设计
- [x] memory-system — 记忆与个性化架构
- [x] output-formatting — 输出格式与风格控制
- [x] search-integration — 搜索与知识检索集成
- [x] conversation-flow — 对话流程与路由设计
- [x] context-management — 上下文与窗口管理
- [x] agent-delegation — 多代理与委派模式
- [x] injection-defense — 注入防御与安全架构
- [x] citation-system — 引用与归属系统设计
- [x] voice-optimization — 语音场景优化
- [x] mobile-adaptation — 移动端适配
- [x] code-engineering — 编程代理模式

### 中优先级（特定场景有价值）

- [ ] educational-design — 教育/教学助手模式
- [ ] document-editing — 文档编辑代理模式
- [ ] enterprise-workspace — 企业 Workspace 集成
- [ ] creative-persona — 创意/娱乐人格设计
- [ ] browser-agent — 浏览器自动化模式

### 不适合 Skill 化的内容

- 具体工具的 JSON Schema 定义（过时太快）
- 具体的模型版本号和知识截止日期
- 特定产品的 UI 交互协议
- 具体的文件路径和 URL 格式

### 预估 Skill 数量

**约 15 个**（本次蒸馏目标）

---

## ✅ 质量门检查

- [x] 主旨能用一句话说清
- [x] 骨架列出 5 个一级模式
- [x] 关键术语词典 10 条
- [x] 批判阶段列出 3 类局限
- [x] 覆盖全部 6 个厂商的产品
